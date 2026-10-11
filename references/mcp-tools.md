# 麦当劳 MCP 工具规范（mcp-tools.md）

MCP Server：`https://mcp.mcd.cn`，Streamable HTTP，请求头 `Authorization: Bearer <用户自备 Token>`。配置示例见 `mcp-config.example.json`。

在 WorkBuddy 中这些工具以 `mcp__mcd-mcp__*` 前缀出现，多数属于延迟加载工具：先通过 ToolSearch 按名称加载 schema，再用 DeferExecuteTool 调用。

## 连接验证

- `now-time-info`：无副作用，首次使用/排查问题时先调用它确认连接
- `query-my-account`：确认账户权限（积分账户）

任一失败（连接失败、Token 无效）→ 按 SKILL.md「MCP 不可用规则」处理，不生成模拟数据。

## 门店与配送

| 工具 | 用途 | 要点 |
|---|---|---|
| `query-nearby-stores` | 到店门店查询 | beType=1 到店自取（返回门店无 beCode）/ beType=5 得来速（有 beCode）；searchType=2 按位置搜索（必填 city + keyword）、=1 搜收藏。无收藏时改用 searchType=2 |
| `delivery-query-addresses` | 查询配送地址 | 外送场景先查地址 |
| `delivery-create-address` | 新增配送地址 | 高风险：用户明确要求才调用 |
| `delivery-query-stores` | 指定地址可配送门店 | 返回 storeCode + beCode，外送点餐必用 |

## 菜单和营养

| 工具 | 用途 | 要点 |
|---|---|---|
| `query-meals` | 门店菜单 | storeCode 必填；beType=1 → orderType=1 不传 beCode；beType=2/6 → orderType=2 必传 beCode；beType=5 → orderType=1 必传 beCode；预约场景传 reservationDate（yyyy-MM-dd HH:mm） |
| `query-meal-detail` | 餐品/套餐详情 | 用于确认单品信息、套餐内容 |
| `list-nutrition-foods` | 营养信息 | 热量数据的权威来源，写入 nutrition.csv 时 confidence=known |

## 优惠和价格

| 工具 | 用途 | 要点 |
|---|---|---|
| `query-store-coupons` | 指定门店可用优惠 | orderType 1=到店（含自取/得来速）、2=外送 |
| `available-coupons` | 可领取优惠 | 只查询不领取 |
| `auto-bind-coupons` | 自动领券 | 高风险：用户明确要求才调用 |
| `query-my-coupons` | 用户已有优惠券 | 优惠优化模块必查 |
| `calculate-price` | 计算应付金额 | 下单前获取最终价格的唯一途径；含餐品、优惠、配送费 |
| `query-promotions` | 团餐促销规则 | 仅 beType=6 企业团餐场景 |

## 订单

| 工具 | 用途 | 要点 |
|---|---|---|
| `order-list` | 近期历史订单 | 需用户允许读取后调用；结果写入 orders.csv（data_source=mcp） |
| `query-order` | 订单详情/状态 | 查询单笔订单 |
| `create-order` | 创建订单 | 高风险：仅在用户明确确认"确认下单/创建订单"后调用 |
| `cancel-order` | 取消订单 | 高风险：用户明确要求才调用 |

## 其他

- `query-my-account`：积分账户
- `campaign-calendar`：活动日历（需当前时间时先调 now-time-info）

另有商城/派对/抽奖工具（mall-*、party-*、draw-lottery、query-lottery-info 等），其中 mall-create-order、party-order-create、draw-lottery 均属高风险操作，需用户明确确认；商城订单查询用 mall-order-list 而非 order-list。

## 调用规则

1. 菜单、价格、优惠、门店、配送和订单状态必须优先使用实时 MCP 数据
2. 用户偏好、预算、忌口保存在本地工作台数据中
3. 不为了展示功能而调用无关工具
4. 工具调用失败时，明确向用户说明失败原因
5. 不使用本地旧数据冒充当前实时结果

## 典型调用链路

- **今日点餐（到店）**：now-time-info → query-nearby-stores(beType=1) → query-meals → query-store-coupons / query-my-coupons → calculate-price →（用户确认）create-order
- **今日点餐（外送）**：now-time-info → delivery-query-addresses → delivery-query-stores → query-meals(beCode) → 券查询 → calculate-price → create-order
- **同步订单**：order-list → 逐条解析写入 orders.csv → 有营养数据的餐品写入 nutrition.csv
