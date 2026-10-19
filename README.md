# 多平台 AI 购物与对话直连落地实施方案
### （Google Gemini、Microsoft Copilot、Meta AI、Perplexity、Claude）

> **实施前提**：完全脱离 Shopify 等建站 SaaS 限制，基于企业自有的 **Google Ads Feed (GMC)** 和 **Meta Ads Feed**，构建自主可控的中央 Feed 托管与多平台直连分发网络。

- **GitHub 仓库**：https://github.com/igocc/ai-shopping-feed-guide
- **在线文档**：https://igocc.github.io/ai-shopping-feed-guide/

---

## 零、 核心底层架构：自建中央 Feed 托管节点

通过独立的对象存储（如 AWS S3、Cloudflare R2 或自建 Nginx），为两大标准 Feed 提供公开且固定不变的 HTTPS 访问链接：

```
[企业商品主数据库 / 自建 ERP]
          │
          ▼ 每日定时导出（UTF-8）
   ┌───────────────────────────┐
   │ 1. google_feed.xml (GMC格式)│
   │ 2. meta_feed.csv  (Meta格式)│
   └─────────────┬─────────────┘
                 ▼ 上传到独立对象存储（AWS S3 / Cloudflare R2 / Nginx）
       https://feed.yourdomain.com/catalog/google_feed.xml
       https://feed.yourdomain.com/catalog/meta_feed.csv
                 │
                 ├──► 1. Google Merchant Center ──► Google Gemini
                 ├──► 2. Microsoft Merchant Center ─► Microsoft Copilot
                 ├──► 3. Meta Commerce Manager ────► Meta AI
                 ├──► 4. Perplexity Merchant ─────► Perplexity Shopping
                 └──► 5. 本地轻量 MCP Server ──────► Claude (Anthropic)
```

---

## 一、 五大 AI 平台直连全景档案

---

### 1. Google Gemini 直连（基于 Google Merchant Center）

*Gemini 的商品搜索、比价、推荐及 AI Overviews 的底层全部直接读取 Google Shopping Graph。*

| 维度 | 具体规范与实操要点 |
| :--- | :--- |
| **申请入口** | `https://merchants.google.com/` （Google Merchant Center Next） |
| **准入条件** | 1. 独立域名所有权（公网有效 HTTPS，支持标准结账流程）；<br>2. 完备的政策页面（配送政策 Shipping、退换货 Refund、隐私条款 Privacy）；<br>3. 真实有效的商家联络信息（电话、客服邮箱、物理地址）。 |
| **需准备资料** | 企业注册英文主体、独立域名 DNS 解析权限（TXT 记录验证所有权）、客服热线及邮箱、退换货与退款政策页面 URL。 |
| **Feed 格式** | **Google Ads Feed 原生格式**（首选 `XML / RSS 2.0`，亦支持 `TSV` 或 `GZ` 压缩文件）。 |
| **更新频次** | **每日至少 1 次**（全量抓取）。针对库存与价格高频波动，可配置 Supplemental Feed 或 Content API 实时更新。 |
| **推送与直连方式** | 1. 登录 GMC 后台 -> **数据源（Data sources）** -> 添加主数据源；<br>2. 接入方式选择 **“计划抓取（Scheduled Fetch）”**；<br>3. 填写中央托管的 Feed 地址：`https://feed.yourdomain.com/catalog/google_feed.xml`，抓取时间设定为每日凌晨；<br>4. **关键开关**：在“成长/管理计划”中必须启用 **“在 Google 上免费发布商品（Free Listings）”**。 |

---

### 2. Microsoft Copilot 直连（基于 Microsoft Merchant Center）

*Copilot 在 Windows/Edge 及 Web 端回答购物咨询时，底层调取的是必应商业索引与微软 Merchant Center。*

| 维度 | 具体规范与实操要点 |
| :--- | :--- |
| **申请入口** | `https://ads.microsoft.com/` -> 工具 -> **Merchant Center** |
| **准入条件** | 1. 拥有已激活的 Microsoft Advertising 广告账户；<br>2. 拥有经 Bing Webmaster Tools 或 UET 标签验证的独立域名；<br>3. 站点支持 SSL 且页面包含退换货说明。 |
| **需准备资料** | 微软广告账号、企业执照主体、域名所有权证据（Bing 网站管理员账号或 DNS 验证）。 |
| **Feed 格式** | **100% 兼容 Google Ads Feed**（直接使用现有 `google_feed.xml` 或 `google_feed.tsv`，无需转换）。 |
| **更新频次** | **每日 1 次**。 |
| **推送与直连方式** | **方案 A（一键跨平台直连，推荐）**：在 MMC 中点击“从 Google Merchant Center 导入”，授权读取已配置的 GMC，设置每日全自动同步；<br>**方案 B（独立拉取）**：创建新目录 -> 新增 Feed -> 传输方法选择 **“现有 URL 定期抓取”**，填入 `google_feed.xml` 的外链。 |

---

### 3. Meta AI 直连（基于 Meta Commerce Manager）

*Meta AI（覆盖 WhatsApp、Instagram、Messenger 等）在用户搜索或触发商品推荐时，直接检索 Meta 商业目录。*

| 维度 | 具体规范与实操要点 |
| :--- | :--- |
| **申请入口** | `https://business.facebook.com/commerce` （Meta 商业管理平台） |
| **准入条件** | 1. Meta Business Suite 企业管理平台管理员权限；<br>2. 完成域名安全验证（Domain Verification）；<br>3. 符合《Meta 商业合作商家协议》及《商业交易政策》（禁售仿冒、侵权品）。 |
| **需准备资料** | Meta 商务账户、企业英文名称、已认证的独立站域名、客服联络邮箱。 |
| **Feed 格式** | **Meta Ads Feed 原生格式**（首选 `CSV` 或 `TSV`，亦支持 RSS XML）。 |
| **更新频次** | **每小时 或 每日 1 次**（Meta 后台支持设定以小时为维度的全量拉取）。 |
| **推送与直连方式** | 1. 进入 Commerce Manager -> 目标商品目录（Catalog） -> **“数据源（Data Sources）”**；<br>2. 选择 **“数据 Feed（Data Feed）”** -> “通过计划 Feed 上传（Scheduled Feed）”；<br>3. 填入：`https://feed.yourdomain.com/catalog/meta_feed.csv`；<br>4. 设定更新周期并完成首次拉取；<br>5. 目录用途中确保勾选 **“用于商业展示与产品推荐”**。 |

---

### 4. Perplexity 直连（基于 Perplexity Merchant Program）

*Perplexity 已正式推出原生 Shopping 引擎（Buy with Pro 及商品答案卡），优先索引签约商户的结构化商品。*

| 维度 | 具体规范与实操要点 |
| :--- | :--- |
| **申请入口** | `https://www.perplexity.ai/hub/merchant-program` （官方商家入驻表单） |
| **准入条件** | 1. 拥有合法独立电商站点（面向美国或全球合规交付）；<br>2. 落地页具备完备且可被机器直接读取的结构化代码（`Schema.org/Product`）；<br>3. 接受 Perplexity 机器人（PerplexityBot）正常爬取（robots.txt 中不可封禁）。 |
| **需准备资料** | 商家英文名、官方主域名、官方联系人/业务邮箱、商品类目与总 SKU 量、**GMC 格式商品 Feed URL**。 |
| **Feed 格式** | **直接复用 Google Ads Feed 格式**（`google_feed.xml`），标准字段包括 `id`, `title`, `description`, `link`, `image_link`, `price`, `availability`。 |
| **更新频次** | **每日抓取** + 页面访问时实时对比。 |
| **推送与直连方式** | 1. 在官方入驻入口填写公司与目录信息，并在 Catalog Feed 项中填入您的中央 Feed 链接：`https://feed.yourdomain.com/catalog/google_feed.xml`；<br>2. 在全站商品落地页（PDP）的 HTML `<head>` 中嵌入标准 JSON-LD 结构化数据（供 PerplexityBot 实时核验最新价格与库存）：<br>```json
{
  "@context": "https://schema.org/",
  "@type": "Product",
  "name": "商品标题",
  "image": "https://example.com/image.jpg",
  "offers": {
    "@type": "Offer",
    "priceCurrency": "USD",
    "price": "79.99",
    "availability": "https://schema.org/InStock"
  }
}
``` |

---

### 5. Claude 直连（基于自建 MCP Server 直连模式）

*Anthropic 官方没有中心化广告录入中心，但其核心开源标准 **MCP（Model Context Protocol）** 是大模型直连私有数据的首选。通过将现有 Feed 封装为轻量 MCP 服务，任何 Claude 客户端或智能体均可原生调用。*

| 维度 | 具体规范与实操要点 |
| :--- | :--- |
| **申请入口** | 无需向任何中心化平台申请审核，自主搭建即刻生效（Anthropic 开源标准）。 |
| **准入条件** | 具备一台独立云服务器或轻量无服务器容器（如 Cloudflare Workers / Docker）。 |
| **需准备资料** | 现有的 `google_feed.xml` 或 `meta_feed.csv` 文件，Python / Node 运行环境。 |
| **Feed 格式** | 服务端本地定时读取现成的 Feed 文件，将其解析为轻量 SQLite 数据库或内存 JSON。 |
| **更新频次** | 后台配置 Linux Cron 定时任务，每小时或每日从中央 S3/R2 下载最新 Feed 覆盖本地数据库。 |
| **推送与直连方式** | 1. 部署一个极简 Python MCP Server（使用官方 `mcp` SDK）；<br>2. 向 Claude 暴露两个轻量只读工具（Tool）：<br>   - `search_products(keyword, category, min_price, max_price)`：全文检索商品；<br>   - `get_product_detail(item_id)`：按 ID 获取商品详细参数、实时库存与落地页；<br>3. 在 Claude Desktop（或团队内部 Claude Agent）配置文件 `claude_desktop_config.json` 中配置接入该 MCP 端点：<br>```json
{
  "mcpServers": {
    "my-shop-catalog": {
      "command": "python",
      "args": ["/path/to/product_feed_mcp.py"]
    }
  }
}
```<br>4. **生效效果**：Claude 对话中遇到用户咨询商品推荐时，会自动触发本地 MCP 工具调取您最真实的商品数据。 |

---

## 二、 全平台 Feed 核心字段标准对照表（一份源数据适配所有 AI）

| 通用字段含义 | Google Gemini / Copilot / Perplexity (GMC XML 标签) | Meta AI (Meta CSV 标题列) | 规则与值约束 |
| :--- | :--- | :--- | :--- |
| **商品唯一ID** | `<g:id>` | `id` | 变体级唯一且不可变（如 `SKU-1001-BLK`） |
| **商品标题** | `<g:title>` | `title` | 纯文本，含核心属性，≤ 150 字符 |
| **商品描述** | `<g:description>` | `description` | 真实客观，严禁出现 HTML 标签，≤ 5,000 字符 |
| **商品落地页** | `<g:link>` | `link` | 公开可达的绝对 HTTPS URL，默认选中对应变体 |
| **商品主图** | `<g:image_link>` | `image_link` | 对应变体的真实图片直链（JPEG/PNG） |
| **价格** | `<g:price>` | `price` | 数字 + 币种，如 `79.99 USD` |
| **促销价** | `<g:sale_price>` | `sale_price` | 真实有效促销价，如 `59.99 USD`（无促销则留空） |
| **库存状态** | `<g:availability>` | `availability` | 严格枚举：`in_stock`, `out_of_stock`, `preorder` |
| **品牌** | `<g:brand>` | `brand` | 页面展示的真实商标/品牌名 |
| **成色** | `<g:condition>` | `condition` | 固定枚举：`new`, `refurbished`, `used` |
| **标准条码** | `<g:gtin>` | `gtin` | 8/12/13/14 位 UPC/EAN 标准码（有则必填） |
| **母商品分组** | `<g:item_group_id>` | `item_group_id` | 多变体商品归属的同一父级 ID |

---

## 三、 从 0 到 1 落地执行四步走

```
第 1 步：生成并托管中央 Feed 文件（第 1 天）
  ├─ 编写脚本从自有数据库每日导出 google_feed.xml 和 meta_feed.csv
  └─ 推送至 S3/R2，生成公共可读链接：https://feed.yourdomain.com/catalog/...

第 2 步：配置三大主流 AI 商业后台（第 1-2 天）
  ├─ Google Merchant Center：配置 Scheduled Fetch（每日拉取），勾选“免费发布商品”（覆盖 Gemini）
  ├─ Microsoft Merchant Center：关联 GMC 一键同步或直接配置定时抓取（覆盖 Copilot）
  └─ Meta Commerce Manager：配置定时 Feed，设置每小时/每日更新（覆盖 Meta AI）

第 3 步：提交 Perplexity 官方商家入驻（第 2-3 天）
  ├─ 提交官网 Merchant Form，填写中央 google_feed.xml 链接
  └─ 确保独立站页面底层的 Schema.org/Product JSON-LD 标签真实可读

第 4 步：搭建 Claude MCP 服务（按需选配，第 3 天）
  └─ 部署基于本地 Feed 缓存的轻量 MCP 检索服务，供 Claude 客户端或内部自动化接入
```
