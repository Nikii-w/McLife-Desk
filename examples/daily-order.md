# 日常点餐示例（daily-order.md）

用户：`根据我的预算、忌口、历史订单和当前优惠，推荐今天的午餐。`

## 流程

1. 检查 DATA_DIR 存在、MCP 可用（不可用则按硬约束停止，只给本地数据能支撑的部分）
2. 确定时间上下文（now-time-info）→ 判断餐段（午餐/晚餐）
3. 确定门店：用档案常用门店，或按用户当前位置实时查询；区分常用门店/当前点餐门店/外送门店
4. 实时查询：query-meals（按就餐方式选 beType/orderType）→ query-store-coupons + query-my-coupons → 必要时 query-meal-detail / list-nutrition-foods
5. 按 references/recommendation-rules.md 过滤：临时规则（查 temporary-rules.json 是否过期）→ 忌口 → 预算 → 重复度
6. 输出最多 3 个方案（最省钱 / 最符合习惯 / 探索），每个含明细、价格、优惠、热量（已知/估算）、匹配点、取舍、数据来源与不确定性

## 用户采纳后

1. 用户表达"就要方案 2" → 调用 calculate-price 获取最终价格
2. 展示最终订单并询问：

```text
请确认最终订单：

门店：人民广场店
就餐方式：到店取餐
餐品：麦辣鸡腿堡套餐
优惠：会员券
预计实付：¥28

确认创建订单吗？
```

**外送场景额外硬约束——下单前必须完整展示并核对收货地址**（信息必须来自 MCP `delivery-query-addresses` 的真实返回，不得凭记忆转述）：

```text
📋 下单前请核对收货地址（外送订单必看）：

| 项 | 内容 |
|---|---|
| 联系人 | 张三（女士） |
| 手机号 | 138****0000 |
| 配送地址 | ××市××区××路××小区 ×号楼 |
| 门牌号 | 502 |
| 预计送达 | 今天 11:30 |

收货地址完整无误吗？确认无误回复"确认下单"；地址有问题我引导你先修正。
```

3. 仅在用户明确回复"确认下单 / 创建订单"后调用 create-order；用户对地址未回应或指出有误时一律不下单
4. 成功后：新订单写入 orders.csv（data_source=mcp），营养数据写入 nutrition.csv，更新 workspace-status.md

## 手动补录

用户：`补录一笔昨天的麦当劳订单，并标记为手动记录。`

→ 无需 MCP 也可执行；询问必要字段（日期、门店、餐品、金额），写入 orders.csv 时 data_source=manual、synced_at=当前时间；热量字段留空或按用户提供的数值记入 nutrition.csv（source=manual）。
