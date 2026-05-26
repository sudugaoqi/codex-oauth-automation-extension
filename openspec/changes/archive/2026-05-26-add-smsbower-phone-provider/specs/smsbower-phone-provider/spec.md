## ADDED Requirements

### Requirement: 用户可以选择 SMSBower 作为接码平台
系统 SHALL 在侧边栏中暴露 SMSBower 作为可选接码平台，并 SHALL 允许它参与手机号验证的 provider 回退顺序。

#### Scenario: 在 provider 设置中可以选择 SMSBower
- **WHEN** 用户打开手机号验证 provider 选择器
- **THEN** 选择器中包含一个 `smsbower` 选项
- **AND** 该选项带有人类可读的 SMSBower 标签

#### Scenario: SMSBower 可以进入 provider 回退顺序
- **WHEN** 用户编辑 provider 顺序控制项
- **THEN** SMSBower 可以与现有 provider 一起进入有序回退列表

### Requirement: SMSBower 国家列表会动态加载
系统 SHALL 从官方 API 动态获取 SMSBower 国家目录，并 SHALL 使用该远端目录填充侧边栏国家优先级 UI。

#### Scenario: 动态国家加载成功
- **WHEN** 当前接码平台是 SMSBower，且侧边栏需要国家列表
- **THEN** 系统会从官方 API 请求 SMSBower 国家目录
- **AND** 侧边栏会使用远端数据渲染可选国家项

#### Scenario: 国家标签优先显示中文
- **WHEN** 某个国家条目同时包含中文和英文标签
- **THEN** 侧边栏使用中文作为主显示标签

#### Scenario: 动态国家加载失败但此前已有保存选择
- **WHEN** SMSBower 国家目录请求失败
- **THEN** 侧边栏保留用户之前保存的国家选择标签
- **AND** 该失败不会覆盖已持久化的 SMSBower 国家选择

### Requirement: SMSBower 可以获取 OpenAI 手机号激活
系统 SHALL 使用官方 handler API、service code `dr`、所选国家顺序、配置的价格约束和 provider 回退规则，从 SMSBower 获取 OpenAI 手机激活号。

#### Scenario: OpenAI 激活使用已验证的 service code
- **WHEN** 系统为 OpenAI 链路向 SMSBower 请求手机号
- **THEN** 它会发送 SMSBower 的 OpenAI service code `dr`

#### Scenario: SMSBower 遵循已选国家顺序
- **WHEN** 用户配置了多个按优先级排序的 SMSBower 国家
- **THEN** 号码获取流程会按配置顺序尝试这些国家
- **AND** 如果启用了价格优先策略，则仍可在国家内部按价格档位重排

#### Scenario: provider 回退可以落到 SMSBower
- **WHEN** 当前首选接码平台无法提供号码
- **THEN** 如果 provider 顺序配置里包含 SMSBower，系统可以回退到 SMSBower

### Requirement: SMSBower 激活可以被轮询和关闭
系统 SHALL 轮询 SMSBower 激活以获取验证码，并 SHALL 支持仓库手机号验证流程所需的终态激活操作。

#### Scenario: SMSBower 返回验证码
- **WHEN** 系统轮询当前 SMSBower 激活且该平台已经收到短信
- **THEN** 系统提取验证码并返回给调用它的手机号验证流程

#### Scenario: 终态失败后关闭 SMSBower 激活
- **WHEN** 手机号验证流程因为超时、换号或停止处理而放弃一个 SMSBower 激活
- **THEN** 系统会发送 SMSBower 所要求的专属关闭或取消请求

#### Scenario: 成功流程结束后标记 SMSBower 激活完成
- **WHEN** 一个 SMSBower 驱动的验证流程成功完成
- **THEN** 系统会执行 SMSBower 规定的成功终态动作
- **AND** 除非后续显式增加复用支持，否则系统不会把该激活当作可复用资源

### Requirement: SMSBower 不支持的 HeroSMS 专属高级流程必须被拦截
系统 SHALL 不会把 SMSBower 静默路由到第一版并不支持的 HeroSMS 专属高级能力里。

#### Scenario: 对 SMSBower 请求激活复用
- **WHEN** 运行时尝试复用一个已保存、且 provider 为 SMSBower 的激活
- **THEN** 系统会快速失败，并给出明确的不支持 provider 提示

#### Scenario: 对 SMSBower 请求临时租号流程
- **WHEN** 当前接码平台是 SMSBower，且注册手机号流程请求临时租号路径
- **THEN** 系统不会尝试调用 HeroSMS 的临时租号 API
- **AND** 系统会自动降级为普通取号路径
- **AND** 侧边栏或运行日志会给出明确提示
