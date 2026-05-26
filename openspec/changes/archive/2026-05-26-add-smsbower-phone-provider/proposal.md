## 为什么

当前手机号验证链路依赖 HeroSMS、5sim 和 NexSMS，短信供给会受到平台和国家库存影响。新增 SMSBower 可以降低无号失败率，并给 OpenAI 验证流程增加一个可用的接码来源。

我们已经实查确认，SMSBower 的官方 handler API 兼容 OpenAI 业务，正式接口域名是 `https://smsbower.page/stubs/handler_api.php`，OpenAI 的 service code 是 `dr`，国家列表也可以通过 `getCountries` 动态获取。因此它和现有的第 2 步 / 第 9 步手机号链路匹配度很高。

## 变更内容

- 在侧边栏和 provider 回退顺序中新增 `smsbower` 作为可选接码平台。
- 为 SMSBower 增加独立的持久化配置和运行态归一化，包含 API Key 与国家顺序。
- 取消本地硬编码国家表，改为从官方 API 动态加载 SMSBower 国家列表。
- 将 SMSBower 接入后台手机号验证链路，覆盖：
  - 使用 service code `dr` 获取 OpenAI 激活号
  - 短信轮询
  - 激活完成 / 取消
  - provider 顺序回退和无号诊断
- 第一版不纳入 HeroSMS 专属能力：
  - 激活复用
  - 临时租号
  - 白嫖复用 / 自动白嫖复用
- 为新 provider 补回归测试，并更新仓库文档。

## 能力范围

### 新增能力

- `smsbower-phone-provider`：支持 SMSBower 作为接码平台，包含动态国家发现、OpenAI 号获取、短信轮询和终态处理。

### 修改能力

- 无。

## 影响面

- 受影响代码：
  - `background.js`
  - `background/phone-verification-flow.js`
  - `sidepanel/sidepanel.html`
  - `sidepanel/sidepanel.js`
  - provider 标签、归一化辅助函数及相关测试
- 外部系统：
  - `smsbower.page` 上的 SMSBower handler API
- 文档：
  - `项目文件结构说明.md`
  - 任何描述接码平台能力的使用说明或流程文档
