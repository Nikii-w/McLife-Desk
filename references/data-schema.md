# 工作台数据结构（data-schema.md）

## 首次初始化

在当前工作区创建以下目录结构：

```
麦麦人生工作台数据/
├── profile.json
├── preferences.md
├── orders.csv
├── nutrition.csv
├── store-preferences.json
├── temporary-rules.json
├── monthly-reports/
├── product-images/        # 报品图片本地缓存（首次报品时自动创建）
└── dashboard.html         # 像素风工作台首页
└── workspace-status.md
```

可从 skill 包的 `templates/profile.json` 复制初始档案骨架。`workspace-status.md` 记录：初始化时间、MCP 连接状态、档案完成度、订单同步范围、各文件行数/更新时间。每次重大数据变更后更新它。

**dashboard.html 生成与刷新（初始化必做）**：复制 skill 包 `templates/dashboard.html` 到 `DATA_DIR/dashboard.html`，把用户的真实档案、订单、统计注入页面内嵌的 `<script id="mc-data" type="application/json">` JSON（字段结构见模板内演示数据，`data_source` 标 `demo` 的订单在注入真实数据后删除）。此后每次订单入账、档案变更、支付确认后都要同步刷新该 JSON，保持页面与本地数据一致。模板内不含任何个人信息，全部数据来自用户自己的 DATA_DIR。

## profile.json

只保存用户主动确认的长期偏好。字段可为 null（用户跳过时）：

```json
{
  "budget": {
    "weekday_lunch": null,
    "dinner": null
  },
  "preferences": {
    "favorite_categories": [],
    "avoid_items": [],
    "drink_preference": null,
    "track_calories": false
  },
  "preferred_stores": []
}
```

推荐扩展字段（用户确认后才写入）：`city_or_area`、`dining_modes`（外送/到店取餐/堂食）、`dietary_restrictions`（忌口/排除食材）、`setmeal_substitution_ok`（是否接受套餐替换）、`weekly_order_target`、`per_order_target`、`allow_order_sync`（是否允许读取历史订单）、`updated_at`。

修改档案前必须向用户复述将要保存的内容；涉及敏感或长期信息时请求确认。

## orders.csv

字段（至少）：

```
date, order_id, store, store_id, channel, total_price, discount, items, total_calories, data_source, synced_at
```

- `channel`：外送 / 到店取餐 / 堂食 / 得来速 / 未知
- `data_source`：`mcp`（order-list / create-order 自动记录）或 `manual`（用户补录，补录时必须标记）
- `total_calories`：可为空；来自 MCP 营养数据时为"已知"，来自估算时在 nutrition.csv 标 `confidence=estimated`
- 缺失字段保留空值，不猜测

## nutrition.csv

```
order_id, item_name, calories, confidence, source
```

- `confidence`：`known`（MCP list-nutrition-foods 返回）或 `estimated`
- `source`：`mcp-nutrition` / `estimate` / `manual`

## store-preferences.json

只保存经过 MCP 查询且用户确认过的门店：

```json
{
  "stores": [
    {
      "name": "",
      "store_code": "",
      "be_code": null,
      "city": "",
      "address": "",
      "dining_modes": [],
      "last_verified_at": ""
    }
  ]
}
```

`be_code` 仅得来速（beType=5）和外送场景有值。用户提供的城市/门店名未经 MCP 验证前，绝不能写入此文件。

## temporary-rules.json

临时偏好规则，过期后自动停止影响推荐（每次使用前检查 `expires_at` 与当前时间）：

```json
{
  "rules": [
    {
      "id": "",
      "rule": "接下来两周不要推荐薯条",
      "type": "avoid_item | prefer_item | budget_override | other",
      "created_at": "",
      "effective_at": "",
      "expires_at": ""
    }
  ]
}
```

## preferences.md

人类可读的偏好说明，与 profile.json 保持同步，供用户直接阅读和手动修改。

## monthly-reports/

按月存放月报：`2026-10.md`。月报必须基于实际保存数据，列出数据范围（起止日期、数据来源构成）和缺失情况（如热量缺失订单数）。

## 数据维护规则

- 所有本地数据允许用户查看、修改、删除
- 订单同步只追加/更新，不静默删除已有记录
- 手动补录与 MCP 记录冲突时，提示用户由其决定保留哪条
