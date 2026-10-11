# 首次使用示例（first-use.md）

用户：`帮我建立个人麦门档案。`

## 第 1 步：验证 MCP

1. ToolSearch 加载 `mcp__mcd-mcp__now-time-info` 并调用 → 成功返回服务器时间即连接正常
2. 调用 `mcp__mcd-mcp__query-my-account` → 确认账户权限
3. 任一失败 → 告知缺少的配置（MCP 地址 `https://mcp.mcd.cn`、Streamable HTTP、Bearer Token），暂停需真实数据的功能，不生成模拟数据

## 第 2 步：收集档案（用户可跳过任何问题）

```text
为了给你更准确的点餐建议，请告诉我：

1. 你常去的城市、区域或门店
2. 常用就餐方式：外送、到店取餐、堂食
3. 工作日午餐预算
4. 晚餐或周末预算
5. 喜欢的餐品或口味
6. 忌口、不吃的食材或需要排除的餐品
7. 饮料偏好
8. 是否关注热量或营养记录
9. 是否允许读取和保存近期订单记录
```

## 第 3 步：验证门店

- 只给城市 → 追问区域或地址
- 给了区域/位置 → `query-nearby-stores`（searchType=2, city+keyword, beType 按就餐方式）
- 需要外送 → `delivery-query-addresses` 后 `delivery-query-stores`
- 给了门店名 → 用 MCP 查询匹配校验

展示查询结果让用户选择确认 → 才写入 store-preferences.json。

## 第 4 步：建数据空间

在当前工作区创建 `麦麦人生工作台数据/` 全部 8 项（见 references/data-schema.md），从 templates/profile.json 复制骨架，填入用户已确认的内容（未答项保持 null）。

## 第 5 步：同步订单（需用户允许）

用户允许后调用 `order-list`，只保存实际返回数据，标记 synced_at 与 data_source=mcp，缺失字段留空。

完成后向用户汇报：MCP 状态、档案保存内容、常用门店、订单同步条数，并提示可用指令（今天吃什么 / 麦麦点餐日历 / 麦门账本…）。
