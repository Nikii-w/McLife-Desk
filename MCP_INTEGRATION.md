# 麦当劳 MCP 集成说明

## MCP Server

- 服务：麦当劳中国 MCP Server
- 接入地址：`https://mcp.mcd.cn`
- 协议：Streamable HTTP
- 鉴权：`Authorization: Bearer <用户自己的 MCP Token>`

用户需要在 WorkBuddy 中自行配置 MCP Token。本仓库只提供脱敏配置示例，不保存或共享任何 Token。

## 使用的 MCP Tools

### 初始化与账户

- `now-time-info`：获取当前时间，用于确定当前月份和时间窗口
- `query-my-account`：确认用户账户和权限状态

### 门店与菜单

- `query-nearby-stores`：根据用户位置查询附近门店
- `delivery-query-addresses`：查询配送地址
- `delivery-query-stores`：查询指定地址可配送门店
- `query-meals`：查询当前门店可售菜单
- `query-meal-detail`：查询餐品和套餐组成、规格及替换选项

### 营养、优惠与价格

- `list-nutrition-foods`：查询餐品营养信息
- `query-store-coupons`：查询指定门店可用优惠
- `available-coupons`：查询可领取优惠
- `query-my-coupons`：查询用户已有优惠券
- `calculate-price`：计算餐品、优惠、配送费和应付金额

### 订单与等级

- `order-list`：读取用户可用的近期历史订单
- `query-order`：补充单笔订单详情和状态
- `create-order`：用户明确确认后创建订单
- `cancel-order`：用户明确要求取消订单时使用

## 主要调用流程

### 首次初始化

```text
now-time-info
    ↓
query-my-account
    ↓
收集用户确认的个人档案
    ↓
query-nearby-stores / delivery-query-stores
    ↓
用户确认常用门店
    ↓
order-list
    ↓
更新工作台订单记录和当月等级
```

### 本月麦门档案等级

```text
now-time-info
    ↓
order-list
    ↓
按当前月份筛选订单
    ↓
统计订单次数、消费金额、优惠和餐品探索
    ↓
计算 Lv.1—Lv.4 月度等级
    ↓
更新麦门身份卡
```

切换历史月份时，Skill 重新按目标月份筛选订单并动态计算等级，不使用永久固定等级。

### 今日点餐

```text
读取个人档案和近期记录
    ↓
query-nearby-stores
    ↓
query-meals
    ↓
query-meal-detail
    ↓
query-store-coupons / query-my-coupons
    ↓
list-nutrition-foods
    ↓
calculate-price
    ↓
输出最多三个可解释方案
```

### 下单

```text
生成推荐方案
    ↓
用户表达采用意图
    ↓
calculate-price
    ↓
展示最终门店、餐品、优惠和价格
    ↓
用户明确确认
    ↓
create-order
    ↓
更新订单记录
```

未获得用户明确确认时，不得调用 `create-order`。

## 数据边界

- MCP 返回的菜单、价格、优惠、门店和订单状态优先于本地缓存
- 本地工作区只保存用户确认的档案和已获取的订单记录
- 数据不完整时必须明确提示
- 不得猜测缺失的价格、餐品、营养数据或订单状态
- 不得将本地演示数据冒充为 MCP 返回的真实数据
