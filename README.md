# GARVEE 美国站 · Shopify Catalog / Agentic 整改清单

基于 **Admin API `2026-07`** 与 **Global Catalog MCP** 的全量实测，对照 Shopify 官方指南逐条核对。

- 店铺：`mygarvee.myshopify.com` / Shop `72466989369` / `www.garvee.com` / USD / Shopify Plus
- 扫描口径：已发布商品 **6,500** 个、变体 **7,000** 个（全量分页，非抽样）
- 实测时间：2026-10-21

**在线报告**：https://igocc.github.io/garvee-shopify-catalog-remediation/

---

## 结论速览

| 项 | 实测 | 判定 |
|---|---|---|
| Global Catalog 收录 | `lookup_catalog` 5 个本店变体 → `resolved: 0` | ❌ 仍未收录 |
| 禁售分类商品 | 14 个 | ❌ P0 |
| US 境内 $0.00 无条件运费 | 1 条 | ❌ P0 |
| US 重复出现在 2 个配送 zone | Domestic + International | ❌ P0 |
| 无重量变体 | 2,116 / 7,000（30.2%） | ❌ P0 |
| Google & YouTube 渠道覆盖 | 91 / 2,500（3.6%） | ❌ P1 |
| 缺货变体 | 5,313 / 7,000（75.9%） | ❌ P1 |
| Unlisted 商品 | 5 个 | ❌ P1 |
| 历史未履约订单 | 8,575 单 + partial 5,494 单 | ⚠️ P2 |
| 已发货无 tracking | 230 / 1,000（23%） | ⚠️ P2 |
| 商品基础字段 | 无图片 0 / 价格≤0 0 / 无描述 0 / 无 URL 0 | ✅ |
| 三类政策 | ToS / Privacy / Refund 全部存在 | ✅ |
| 域名 + SSL | 5 个全部 verified | ✅ |

---

## 适用哪份官方指南

本店 `passwordProtection.enabled = false`、`primaryDomain = https://www.garvee.com`，即**已发布 Shopify 在线商店**，
适用通用 Agentic Storefronts 指南。

Agentic Plan 专用指南中以下要求**不适用于本店**，不要照做：

| 项 | 为何不适用 |
|---|---|
| `external_url` 标准元字段 | 仅无 Shopify 商店的店强制；实测未定义，不构成问题 |
| 添加现有域名做 TXT 验证 | 域名原生托管于 Shopify |
| Shopify Knowledge Base app | 仅 Agentic Plan 预装，未装不是问题 |
| Combined listings / catalog mapping | 6,500 商品 100% 单变体 |
| 企业资料 / 身份验证 / 营业执照 | 仅 Agentic Plan 要求 |

---

## 整改清单（6 类 / 26 条）

### P0-01 下架 14 个禁售分类商品

| 代码 | 官方禁售类 | 数量 |
|---|---|---|
| A | Alcohol — homebrewing and winemaking supplies | 10 |
| B | Science & laboratory — laboratory equipment | 3 |
| C | Hunting & shooting | 1 |

明细见 [`data/P0-01-banned-category-products.csv`](data/P0-01-banned-category-products.csv)。

> **别误伤**：`Wine Fridges` / `Wine Racks` / `Wine & Liquor Cabinets` 属于家具家电，**不在禁售范围**。

- [ ] 将 14 个商品改为 `DRAFT` 或 `ARCHIVED`
- [ ] 不要仅改分类，Shopify 会依据商品信息重新判定
- [ ] 改完后确认 `onlineStoreUrl` 失效

### P0-02 修正 US 境内 $0.00 无条件运费

```
ZONE Domestic ['US']
   "Delivered in 3-5 business days"  DeliveryRateDefinition $0.00  methodConditions = []
```

`methodConditions` 为空数组，美国境内任何订单都可选 $0 运费。

- [ ] 确认是否为有意包邮
- [ ] 若非包邮，补费率或增加 `TOTAL_PRICE` / `WEIGHT` 条件

### P0-03 移除 International zone 中的 US

```
ZONE Domestic      ['US']
ZONE International [AE AT AU BE CH CZ DK ES FI FR HK IE IL IT JP KR MY NL NO NZ PL PT SE SG US]
   "Undeliverable" $30 / $40
```

- [ ] 从 International zone 移除 `US`
- [ ] 确认 Domestic zone 是 US 唯一归属

### P0-04 补齐 2,116 个变体的重量

```
变体 7,000 个，重量为 0 或空：2,116（30.2%）
单位混用：KILOGRAMS 4,194 / POUNDS 184 / OUNCES 6
零值单位：GRAMS 1,009 / OUNCES 849 / KILOGRAMS 252
店铺设置：weightUnit = OUNCES
```

明细见 [`data/P0-04-missing-weight-variants.csv`](data/P0-04-missing-weight-variants.csv)。

- [ ] 补齐 2,116 个变体重量
- [ ] 统一单位到 OUNCES / IMPERIAL

### P1-01 修复 Google & YouTube 渠道覆盖率

```
Online Store 100.0% | Shop 100.0% | TikTok 99.8% | Pinterest 96.3%
Facebook & Instagram 76.8% | Google & YouTube 3.6%
```

- [ ] 确认生效的 feed 路径是 `Google & YouTube` 还是 `Simprosys`
- [ ] 将主推商品发布到 `Google & YouTube` publication

### P1-02 处理 5,313 个缺货变体

`search_catalog` 默认按 `available` 过滤，缺货商品不会被返回。这是搜不到的**第二个原因**。

明细见 [`data/P1-02-out-of-stock-variants.csv`](data/P1-02-out-of-stock-variants.csv)。

- [ ] 优先给主推 SKU 补货
- [ ] 确认外部仓库存已同步（9 个地点中仅 `US West Warehouse` 的 `shipsInventory = true`）

### P1-03 清理 5 个 Unlisted 商品

明细见 [`data/P1-03-unlisted-products.csv`](data/P1-03-unlisted-products.csv)。

- [ ] 改为 `ACTIVE` 或 `DRAFT` / `ARCHIVED`

### P2-01 ~ P2-05

历史未履约 / 无 tracking / FB&IG 覆盖 / 重复标题 / 无费率地点，详见 [在线报告](https://igocc.github.io/garvee-shopify-catalog-remediation/)。

---

## 执行顺序

```
1. P0-01  下架 14 个禁售商品
2. P0-02  确认/修正 $0 运费
3. P0-03  移除 International zone 中的 US
4. P1-03  清理 5 个 Unlisted 商品
5. P0-04  补 2,116 个变体重量 + 统一单位
6. P1-02  补主推 SKU 库存
7. P1-01  修 Google & YouTube 覆盖率
8. 回后台确认 Catalog 商品数是否从 0 变为正数
9. P2-01 ~ P2-05  清理历史与体验问题
```

---

## 数据文件

| 文件 | 内容 | 行数 |
|---|---|---|
| `data/audit-data.json` | 全量结构化数据 | — |
| `data/P0-01-banned-category-products.csv` | 禁售分类商品 | 14 |
| `data/P0-04-missing-weight-variants.csv` | 无重量变体 | 2,116 |
| `data/P1-02-out-of-stock-variants.csv` | 缺货变体 | 5,313 |
| `data/P1-03-unlisted-products.csv` | Unlisted 商品 | 5 |

所有数据仅含商品 ID / handle / 标题 / 分类 / 状态，**不含价格、客户、订单或凭证**。

---

## 证据边界

**已实测确认**：上表全部数字，均来自真实 Admin GraphQL 与 Global Catalog MCP 请求。

**无法通过 API 确认，必须后台查看**：

- `Sales channels > Agentic > Sources > Shopify Catalog` 的资格状态与商品数
- Agentic 附加条款接受状态
- 各 AI 渠道 direct checkout 的 opt-in 状态
- Catalog access 开关状态
- B2B-only 是否关闭（`markets` 字段权限不足）

`Shop.merchantApprovalSignals` 实测返回 `ACCESS_DENIED`（缺 `read_merchant_approval_signals`）；
`Shop` 类型 57 个字段中不存在 Catalog 资格或评分字段。

---

## 复现

```bash
# Global Catalog 反查（必须带合法 agent profile）
curl -X POST https://catalog.shopify.com/api/ucp/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{
        "name":"lookup_catalog","arguments":{
          "meta":{"ucp-agent":{"profile":"https://shopify.dev/ucp/agent-profiles/examples/2026-08-25/valid-with-capabilities.json"}},
          "catalog":{"ids":["gid://shopify/ProductVariant/49446593855801"],
                     "context":{"address_country":"US"}}}}}'
```

```graphql
# 配送配置
{ deliveryProfiles(first:5){ nodes{ name default coversAllItems locationsWithoutRatesCount
  profileLocationGroups{ locationGroup{ locations(first:20){ nodes{ name } } }
    locationGroupZones(first:20){ nodes{ zone{ name countries{ code{countryCode} } }
      methodDefinitions(first:20){ nodes{ name active
        methodConditions{ field operator conditionCriteria{ __typename } }
        rateProvider{ __typename ... on DeliveryRateDefinition { price{amount currencyCode} } } } } } } } } } }
```

```graphql
# 变体重量与可售性
{ productVariants(first:100, after:$cursor, query:"published_status:published"){
    pageInfo{hasNextPage endCursor}
    nodes{ id availableForSale inventoryItem{ tracked measurement{ weight{ value unit } } } } } }
```

---

## 参考官方文档

- [Shopify Catalog 概览](https://help.shopify.com/en/manual/shopify-catalog/index)
- [Catalog 收录要求](https://help.shopify.com/en/manual/shopify-catalog/requirements)
- [Agentic Storefronts](https://help.shopify.com/en/manual/online-sales-channels/agentic-storefronts)
- [Agentic Plan 设置](https://help.shopify.com/en/manual/online-sales-channels/agentic-storefronts/agentic-plan-setup)
