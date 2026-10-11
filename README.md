# 麦麦人生工作台（McLife Desk）

一个运行在 WorkBuddy 中的个人麦当劳消费管理工作台 Skill。

它连接用户自己的麦当劳 MCP，帮助用户：

- 建立个人点餐档案，保存预算、忌口和口味偏好
- 查询和同步近期订单
- 查看麦麦点餐日历
- 计算本月麦门档案等级
- 测试麦门消费人格
- 生成麦门身份卡
- 分析消费次数、金额、优惠和已知营养数据
- 根据实时菜单、门店、优惠和个人偏好辅助下一次点餐

本项目是麦当劳程序员创意开发大赛参赛作品，不是麦当劳官方产品。

## 使用前提

用户需要：

1. 安装 WorkBuddy
2. 申请属于自己的麦当劳 MCP Token
3. 在 WorkBuddy 中配置麦当劳 MCP Connector
4. 安装本 Skill

每个用户必须使用自己的 Token。本项目不包含任何真实 Token、账号凭证、订单数据或个人信息。

## 安装与配置

详细步骤见 `INSTALL.md`。

MCP 配置示例见 `mcp-config.example.json`。请将 `YOUR_MCP_TOKEN` 替换为用户自己申请的 Token，不要把真实 Token 提交到 GitHub。

## 常用指令

安装后，可以在 WorkBuddy 中使用：

```text
初始化麦麦人生工作台
同步近期麦当劳订单
打开我的麦麦点餐日历
同步当前月份订单并计算我的本月麦门档案等级
切换到上个月看我上个月的麦门等级
开始麦门消费人格测试
生成我的麦门身份卡
看看今天吃什么
查看本月麦门账本
```

## 核心设计

本 Skill 的主体是 WorkBuddy 工作台和可执行的 Skill 规则，不是静态 HTML 页面。

HTML 模板只用于展示已经从 MCP 获取并保存的数据。实时菜单、价格、优惠、订单状态和下单操作必须由 Skill 调用麦当劳 MCP 完成。

## 目录结构

```text
mc-life-desk/
├── SKILL.md
├── INSTALL.md
├── CONTEST_DECLARATION.md
├── MCP_INTEGRATION.md
├── mcp-config.example.json
├── references/
├── examples/
└── templates/
```

## 隐私与安全

- 不提交真实 MCP Token、密码、手机号、地址或账号凭证
- 不将用户订单数据提交到公开仓库
- 用户可以查看、修改和删除本地工作台数据
- 创建订单、取消订单、领券等高风险操作必须经过用户明确确认
- 热量和营养数据仅供消费记录和参考，不构成医疗或专业营养建议
