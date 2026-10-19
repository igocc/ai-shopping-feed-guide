# 多平台 AI 购物直连实施全景指南
### （Google Gemini、Microsoft Copilot、Meta AI、Perplexity、ChatGPT、Claude）

> **实施目标**：完全脱离 Shopify 等建站 SaaS 限制，基于企业自有的 **Google Ads Feed (GMC)** 和 **Meta Ads Feed**，直连各大主流 AI 平台的商品推荐与购物搜索体系。

- **GitHub 仓库**：https://github.com/igocc/ai-shopping-feed-guide
- **在线文档**：https://igocc.github.io/ai-shopping-feed-guide/

---

## 零、 核心结论：各大 AI 平台入口属性与直连难度审定

在启动落地前，必须先看清各大平台获取商品数据的**底层逻辑完全不同**。市面上所谓的“AI 直连”分为三类截然不同的机制：

| 平台 | 入口属性判定 | 是否支持自主提交 Feed？ | 审核与准入机制 | 难度系数 | 真实审批现状与商业定论 |
| :--- | :--- | :---: | :--- | :---: | :--- |
| **Google Gemini** | **现成广告/电商后台**<br>(Google Merchant Center) | **是** | **系统自动化审核**<br>(合规即自动生效) | **★☆☆☆☆**<br>(极低 / 全开放) | **无需任何单独申请**。在 GMC 中提交现有的 Google Feed，勾选“免费发布商品”，系统自动入库 Google Shopping Graph，Gemini 对话直接调取。 |
| **Microsoft Copilot**| **现成广告/电商后台**<br>(Microsoft Merchant Center) | **是** | **系统自动化审核**<br>(可一键绑定 GMC) | **★☆☆☆☆**<br>(极低 / 全开放) | **无需任何单独申请**。通过微软广告后台创建 MMC，直接填入 Google Feed 或一键同步 GMC，Copilot 自动在必应商业索引中读取。 |
| **Meta AI** | **现成商业管理后台**<br>(Meta Commerce Manager) | **是** | **资质审查 + 域名验证**<br>(机器审核为主) | **★★☆☆☆**<br>(低 / 全开放) | **无需找 Meta AI 申请**。通过 Meta BM 建立 Catalog 目录，配置 Meta Feed 定时拉取，只要开启“商业推荐”权限即可覆盖 WhatsApp/Instagram/Messenger 推荐。 |
| **Perplexity** | **独立专属 AI 入口**<br>(Perplexity Merchant Program) | **是** | **专属表单 + 人工筛选**<br>(评估独立站质量与体量) | **★★★☆☆**<br>(中等 / 需审核) | **必须走专属入口申请**。面向独立站商户开放填表，需提供 GMC 格式 Feed URL，官方人工审核站点质量、美国本土履约能力与落地页 Schema 代码。 |
| **ChatGPT** | **独立专属 AI 入口**<br>(chatgpt.com/merchants) | **是** | **表面候补（Waitlist）**<br>**实质为【封闭定向邀约制】** | **★★★★★**<br>(极高 / 强门槛) | **普通申请不受制于提交即获批**。官方明确定位仅面向 *“Approved Partners”*。普通商户填表后基本长期处于 Waitlist 状态；当前官方重点与头部平台/战略品牌直连。自建技术完成不等于获批。 |
| **Claude** | **无任何商户或 Feed 入口**<br>(纯靠公网搜索引擎爬取) | **否**<br>*(官方根本无 Feed 端点)* | **不支持提交任何 Feed**<br>*(纯靠 SEO 爬虫自然收录)* | **N/A**<br>(不适用) | **彻底澄清**：Anthropic 官方**没有广告中心、没有商户平台、不接收任何 Feed**。公网 `claude.ai` 纯靠搜索引擎爬取独立站网页及 `Schema.org` 微数据；开源的 MCP 协议是**私有端对端插头**，全世界普通消费者根本无法通过 MCP 搜到你的商品。**做 Feed 获客请直接把 Claude 从 Feed 清单排除**。 |

---

## 一、 为什么 Claude 无法通过 Feed 提交？（原理剖析）

针对 Claude，必须严格区分**“公网消费者获客”**与**“企业私有内网工具”**：

### 1. 公网网民在 `claude.ai` 提问时，数据怎么来的？
普通消费者在 Claude 网页端或 App 询问购物建议（例如“推荐一款车库取暖器”），Claude 的数据链路为：
```
消费者在 claude.ai 提问
        │
        ▼ 触发联网搜索（调用第三方商业搜索引擎接口，如 Brave Search）
   检索公网网页
        │
        ▼ 抓取独立站公开详情页 HTML，解析 Schema.org/Product 代码（而非 Feed！）
   Claude 总结生成推荐，并附带独立站链接
```
* **结论**：对公网 Claude 而言，**没有任何地方可以提交 Feed**。商家唯一要做的是做好独立站 SEO，确保页面嵌入标准的 `Schema.org/Product` JSON-LD，并在 `robots.txt` 放行 `ClaudeBot`。

### 2. 那所谓的“自建 MCP”到底是怎么回事？
MCP（Model Context Protocol）是 Anthropic 开源的模型上下文协议，它是**私有外挂**，而不是公共广告系统：
* **谁能访问？** 只有**安装了该 MCP 配置文件的那台具体电脑/服务器**。
* **全世界普通网民能否访问？** **绝对不能**。全世界网民在 `claude.ai` 聊天，根本不可能连到你自建的私有 MCP 服务器。
* **自建 MCP 的唯一用途**：企业自建内部智能客服、销售助手查库存，**绝不是公网获客渠道**。

---

## 二、 准备清单与详细资料清单（0 到 1 必备材料）

在开始对接前，必须提前备齐以下四组核心资料：

### 1. 独立站合规与政策页面清单（四大必备合规页面）
所有商业与 AI 平台（尤其是 Google、Meta、Perplexity）在收录商品前均会通过自动化爬虫或人工复核独立站的合规性，缺少以下任何一项都将导致封禁或驳回：
* [ ] **Contact Us（联系我们页面）**：必须展示**真实可拨通的电话号码**、**专属客服邮箱**、**企业法定注册实体地址**（严禁仅留一个简单的无信息留言框）。
* [ ] **Shipping Policy（配送政策页面）**：明确列出配送覆盖国家/地区、合作承运商（如 UPS、FedEx）、订单处理时效（Handling Time，如 1-2 个工作日）、运输时效（Transit Time，如 3-5 个工作日）、明确的运费计算与包邮门槛。
* [ ] **Refund & Return Policy（退换货政策页面）**：明确退货支持时限（如 30 天无理由退货）、退货运费承担主体（买家还是商家）、退款原路返还时效。
* [ ] **Terms of Service & Privacy Policy**：公开合规的在线服务条款与隐私政策。
* [ ] **公开结账通道**：全站必须支持公开 HTTPS 正常下单并完成信用卡/PayPal 支付，严禁出现仅询盘或非闭环结算。

### 2. 企业资质与对接人资料清单
* [ ] **企业法定注册信息**：营业执照英文名称、注册地址、企业税号（EIN / VAT 等）。
* [ ] **域名 DNS 管理权限**：具备添加 TXT 记录权限，用于完成 Google、Bing、Meta 的域名所有权秒级认证。
* [ ] **官方业务与技术对接人（专供 Perplexity / ChatGPT 申请表填报）**：
  * 对接人姓名（First Name / Last Name）
  * 职务（Work Title）
  * 企业域名工作邮箱（如 `service@yourdomain.com`，严禁使用 Gmail 等公共邮箱）
  * 领英职业主页链接（LinkedIn Profile URL）

### 3. 中央托管 Feed 静态资产清单
在独立对象存储（如 AWS S3、Cloudflare R2 或独立 Nginx 服务器）上配置两个**固定不变、公开可直接下载**的静态链接：
* [ ] `https://feed.yourdomain.com/catalog/google_feed.xml`（遵循 Google Merchant Center XML/RSS 规范）
* [ ] `https://feed.yourdomain.com/catalog/meta_feed.csv`（遵循 Meta Commerce CSV/TSV 规范）

---

## 三、 各平台直连实操指南（逐一落地）

---

### 1. Google Gemini 直连（基于 Google Merchant Center）
* **入口属性**：**现成后台，无需单独申请**。
* **后台地址**：`https://merchants.google.com/`
* **前置条件**：完成 GMC 独立站域名所有权验证（DNS TXT 或 HTML 标记）。
* **具体配置步骤**：
  1. 登录 GMC 后台，点击 **“商品 (Products)” -> “数据源 (Data sources)”**，新增主数据源。
  2. 目标国家选择您的销售目的地（如美国），语言选择英语。
  3. 传输方式选择 **“计划抓取 (Scheduled Fetch)”**，填入中央 Feed 链接：`https://feed.yourdomain.com/catalog/google_feed.xml`。
  4. 抓取频率设定为**每日 1 次**（建议设在当地时间凌晨）。
  5. **【关键核心开关】**：进入 **“设置/增长” -> “管理计划 (Manage programs)”**，确保必须激活 **“在 Google 上免费发布商品 (Free Product Listings)”**。
* **生效机制**：只要 Free Listings 通过机器审核，商品自动进入 Google Shopping Graph，Gemini 即可在对话中直接检索并推荐。

---

### 2. Microsoft Copilot 直连（基于 Microsoft Merchant Center）
* **入口属性**：**现成后台，无需单独申请**。
* **后台地址**：`https://ads.microsoft.com/` -> 工具 -> **Merchant Center (MMC)**
* **具体配置步骤（两种方案）**：
  * **方案 A（一键跨平台同步，最推荐）**：在 MMC 中直接选择“从 Google Merchant Center 导入”，授权读取已配置好的 GMC，设置每日自动镜像同步，零开发量。
  * **方案 B（独立定时抓取）**：在 MMC 中创建店铺目录，传输方法选择“现有 URL 定期抓取”，填入 `google_feed.xml` 的外链。
* **生效机制**：需确保独立域名已在 Bing Webmaster Tools 或通过 UET 标签完成验证；商品目录审核通过后，Copilot 自动调取。

---

### 3. Meta AI 直连（基于 Meta Commerce Manager）
* **入口属性**：**现成后台，无需单独申请**。
* **后台地址**：`https://business.facebook.com/commerce`
* **具体配置步骤**：
  1. 在 Meta Business Suite 中完成独立域名安全验证（Domain Verification）。
  2. 进入 Commerce Manager -> 选择商品目录（Catalog） -> 点击 **“数据源 (Data Sources)”**。
  3. 选择 **“数据 Feed (Data Feed)”** -> **“通过计划 Feed 上传 (Scheduled Feed)”**。
  4. 填入中央静态链接：`https://feed.yourdomain.com/catalog/meta_feed.csv`。
  5. 更新频率建议设为**每日 1 次 或 每小时 1 次**。
  6. **【核心设置】**：在目录设置中，确保该 Catalog 的业务用途包含“商业展示与产品推荐”，而非仅作为封闭受限广告定向。
* **生效机制**：Meta AI（覆盖 WhatsApp、Instagram、Messenger 等）在用户触发购物对话时直接检索该目录。

---

### 4. Perplexity 直连（基于 Perplexity Merchant Program）
* **入口属性**：**独立专属入口，需人工审核**。
* **申请入口**：`https://www.perplexity.ai/hub/merchant-program`
* **申请表填报细节**：
  * 品牌/商户名称（Brand Name）、独立站主域名（Website URL）；
  * 对接人姓名、职务、企业邮箱、LinkedIn 个人主页；
  * 主营商品品类、总 SKU 数量、主要履约国家（优先支持美国市场）；
  * **Catalog Feed 链接**：直接填写您的 `google_feed.xml` 静态链接。
* **落地硬性技术配合（必做）**：
  * 独立站商品详情页（PDP）的 HTML `<head>` 中必须包含完整的 **`Schema.org/Product`** JSON-LD 代码（包含 `name`, `image`, `offers.price`, `offers.priceCurrency`, `offers.availability`）。
  * 检查站点 `robots.txt`，严禁拦截 `PerplexityBot`。

---

### 5. ChatGPT 直连（基于 ChatGPT Merchants 入口）
* **入口属性**：**独立专属入口，表面候补，实质为封闭定向邀约制**。
* **申请入口**：`https://chatgpt.com/merchants/`
* **现状与预期审定**：
  * 官方文档明确说明直连仅面向 *“Approved Partners”*。普通商户提交申请后自动进入 Waitlist，官方无承诺审核时效（SLA）。
  * 目前公开接入案例主要集中于平台级合作或超大体量零售商。
  * **实操策略**：**资料照填，技术备妥，作为战略储备，但不作为短期主要获客渠道**。
* **申请表填报材料**：
  * 对接人姓名、职务、工作邮箱、LinkedIn URL；
  * 公司英文名、总部国家、独立站官网、主营类目、唯一 SKU 数量；
  * Feed 就绪状态：勾选“Feed 已就绪”。
* **获批后的官方标准技术通道**：
  * **协议**：SFTP 推送至 OpenAI 分配的根目录。
  * **格式**：首选 `parquet` (zstd 压缩) 或 `jsonl.gz`（UTF-8 编码）。
  * **规则**：固定文件名覆盖更新，全量快照每日 1 次；快照遗漏商品保留至多 14 天；主动下架需标记 `"is_eligible_search": false`。

---

## 四、 全平台通用 Feed 核心字段标准对照表

只需自建一套导出服务，产出以下标准字段，即可无损满足所有支持 Feed 的平台：

| 通用业务字段 | Google (GMC XML 标签) | Meta (CSV 列名) | 规范约束与说明 |
| :--- | :--- | :--- | :--- |
| **商品唯一 ID** | `<g:id>` | `id` | 变体级不可变 ID（如 `SKU-1001-BLK-10`），修改价格或标题不得更换 ID |
| **商品标题** | `<g:title>` | `title` | 纯文本，包含核心变体属性，≤ 150 字符，不堆砌促销词 |
| **商品描述** | `<g:description>` | `description` | 客观真实纯文本，≤ 5,000 字符，严禁包含 HTML 标签 |
| **商品落地页** | `<g:link>` | `link` | 公开可达的绝对 HTTPS URL，默认选中对应变体 |
| **主图链接** | `<g:image_link>` | `image_link` | 对应变体的高清直链（JPEG/PNG），公网直接可访问 |
| **售价** | `<g:price>` | `price` | 数字 + 币种，如 `79.99 USD`，必须与落地页应付金额一致 |
| **促销价** | `<g:sale_price>` | `sale_price` | 真实生效促销价，如 `59.99 USD`（无促销则留空） |
| **库存状态** | `<g:availability>` | `availability` | 严格固定枚举：`in_stock`, `out_of_stock`, `preorder` |
| **品牌** | `<g:brand>` | `brand` | 页面展示的真实品牌/商标名 |
| **成色** | `<g:condition>` | `condition` | 固定枚举：`new`, `refurbished`, `used` |
| **标准条码** | `<g:gtin>` | `gtin` | 8/12/13/14 位 UPC/EAN 标准码（有则必填，保留前导零） |
| **母商品分组** | `<g:item_group_id>` | `item_group_id` | 多变体商品归属的同一父级 ID |

---

## 五、 企业推进的优先级落地路线

```
【第一优先级：确定性 100%，当天生效】
  1. 生成并托管中央静态 Feed（google_feed.xml 和 meta_feed.csv）
  2. 登录 Google Merchant Center，配置 Scheduled Fetch 并开启“免费发布商品”（覆盖 Gemini）
  3. 登录 Microsoft Merchant Center，一键关联 GMC 自动同步（覆盖 Copilot）
  4. 登录 Meta Commerce Manager，配置 Scheduled Feed 定时排程（覆盖 Meta AI）

【第二优先级：确定性 70%，1-2 周内跟进】
  5. 检查独立站商品落地页（PDP）底层的 Schema.org/Product JSON-LD 标签
  6. 提交 Perplexity Merchant Program 官方入驻表单，提供中央 google_feed.xml 链接

【第三优先级：战略留档 / SEO 常规维护】
  7. 提交 ChatGPT Merchants 申请表留档进 Waitlist，备妥技术规格但不赌审批时效
  8. 针对 Claude 及其他无商户系统的 AI：确保 robots.txt 允许爬取，做好独立站常规 SEO 即可
```
