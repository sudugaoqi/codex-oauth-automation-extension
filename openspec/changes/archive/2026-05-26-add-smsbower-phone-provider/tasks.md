## 1. 设置与侧边栏

- [x] 1.1 在 `background.js` 和相关辅助模块中添加 `smsbower` provider 常量、默认设置和持久态归一化。
- [x] 1.2 在 `sidepanel/sidepanel.html` 和 `sidepanel/sidepanel.js` 中加入 SMSBower 选项、API Key 输入框以及国家优先级 UI 接线。
- [x] 1.3 在侧边栏实现 SMSBower 国家动态加载、标签归一化、选择持久化和失败处理。

## 2. 后台手机号链路

- [x] 2.1 为 SMSBower 实现国家发现、余额 / 价格查询、激活获取、短信轮询和终态动作等 API helper，并对接 `smsbower.page`。
- [x] 2.2 将 SMSBower 接入 `background/phone-verification-flow.js`，覆盖 provider 解析、provider 顺序回退、激活归一化和 OpenAI service code `dr`。
- [x] 2.3 增加 provider 能力约束，避免 SMSBower 在 v1 中进入 HeroSMS 专属的复用、临时租号或白嫖复用分支；其中临时租号在侧边栏与运行时自动降级为普通取号。

## 3. 验证与文档

- [x] 3.1 为设置归一化、侧边栏动态国家、provider 顺序行为、激活获取、短信轮询和不支持能力拦截补充或更新回归测试。
- [x] 3.2 更新仓库文档以反映新 provider，并在 `项目文件结构说明.md` 中登记这组 OpenSpec change 文件。
