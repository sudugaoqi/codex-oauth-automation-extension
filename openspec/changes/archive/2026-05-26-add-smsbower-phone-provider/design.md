## 背景

当前仓库已经在一条共享自动化链路里支持了 3 个接码平台：

- HeroSMS
- 5sim
- NexSMS

相关接入面主要分布在 3 层：

- `sidepanel/sidepanel.html` 与 `sidepanel/sidepanel.js`：负责 provider 选择、顺序选择、国家选择和凭证输入
- `background.js`：负责持久化默认值和设置归一化
- `background/phone-verification-flow.js`：负责拿号、收码轮询、换号、回退、诊断和不支持路径拦截

当前 provider 架构只做了部分抽象。`phone-sms/providers/` 已存在，但真实行为仍大量实现于 `background/phone-verification-flow.js`。因此即使运输协议很简单，接入 SMSBower 依然是一次跨层改动。

我们已经做过实查确认：

- 正式 API 域名：`https://smsbower.page/stubs/handler_api.php`
- OpenAI service code：`dr`
- `getCountries` 会返回包含 `id / eng / chn` 的动态国家元数据
- `getTopCountriesByService`、`getPrices`、`getPricesV2`、`getPricesV3` 都支持 `service=dr`

## 目标 / 非目标

**目标：**

- 让 SMSBower 成为一个可独立选择的接码平台。
- 动态拉取 SMSBower 国家列表，并接入现有国家优先级 UI。
- 复用现有第 2 步 / 第 9 步手机号验证编排，不新增一套平行流程。
- 支持必要的激活生命周期：
  - 获取号码
  - 轮询短信
  - 标记成功 / 取消
- 当 SMSBower 存在时，仍保持 provider 回退顺序和无号诊断可用。

**非目标：**

- 重构整套手机号 provider 架构。
- 为 SMSBower 实现激活复用。
- 为 SMSBower 实现临时租号。
- 为 SMSBower 实现白嫖复用 / 自动白嫖复用。
- 在 v1 中暴露自定义 SMSBower base URL 或 service code 配置。

## 决策

### 1. 以独立 provider ID 接入，不把 SMSBower 伪装成 HeroSMS

新增 provider ID：`smsbower`。

原因：

- 协议虽然兼容 handler-api，但库存、国家目录和能力面与 HeroSMS 不同。
- 现有 HeroSMS 分支包含复用、临时租号、白嫖复用等逻辑，不能默认套用到 SMSBower。
- 独立 provider ID 能把诊断、不支持路径和测试边界写清楚。

备选方案：

- 用自定义 base URL 复用 `hero-sms`。拒绝，因为它会把不支持的能力藏进 HeroSMS 分支，后续维护更难。

### 2. 后台固定使用 OpenAI service code `dr`

实现中直接把 SMSBower 的 OpenAI service code 固定为 `dr`。

原因：

- 我们已经通过线上 `getServicesList` 实查确认了 `dr`。
- 这个仓库本身也会在别处硬编码 provider 专属 OpenAI 标识。
- 把 service code 做成可编辑项会增加 UI 和校验复杂度，但收益很小。

备选方案：

- 在侧边栏暴露 service code 输入框。拒绝，因为仓库这次只需要 OpenAI 链路，不需要通用 SMSBower 服务配置。

### 3. 在侧边栏动态加载国家目录，只持久化用户选择

当选中 `smsbower` 时，侧边栏调用 `getCountries`，把结果转换成菜单项，并沿用 HeroSMS / 5sim / NexSMS 已有的多选顺序 UI 模式。

持久化字段保持窄化：

- `smsBowerCountryId`
- `smsBowerCountryLabel`
- `smsBowerCountryOrder`

v1 不持久化完整远端国家目录。

原因：

- 动态加载能满足需求，不需要把大表直接塞进本地状态。
- 只保存已选国家标签，就算后续拉取失败，UI 也能保持稳定。
- 这样能减少 localStorage 体积，也避免陈旧目录迁移问题。

备选方案：

- 把完整国家目录缓存到 `chrome.storage.local`。拒绝，因为它会引入陈旧和迁移问题，且对首版没必要。

### 4. 国家标签优先中文，其次英文

侧边栏菜单和保存标签的解析顺序为：

1. `chn`
2. `eng`
3. `Country #<id>`

原因：

- 仓库 UI 和日志本来就是中文为主。
- 线上 API 已经返回中文标签。

备选方案：

- 英文优先。拒绝，因为和现有 UI 风格不一致。

### 5. 接入时使用 provider 专属解析，但复用现有价格编排形状

获取号码逻辑仍保留在 `background/phone-verification-flow.js`，但 SMSBower 会使用自己的解析 / 请求 helper：

- 拉国家
- 拉价格
- 通过 `getNumber` / `getNumberV2` 请求激活号
- 通过 `getStatus` 轮询短信
- 通过 `setStatus` 取消 / 完成

整体编排仍沿用当前项目形状：

- 先按当前 provider 处理
- 无号时按 provider 顺序回退
- provider 内再按国家顺序回退
- 如有价格信息则继续使用价格感知排序

原因：

- 这样能保持 auto-run 和诊断逻辑已经预期的行为模式。
- 避免把 SMSBower 的响应解析和 HeroSMS 的假设混在一起。

备选方案：

- 为 SMSBower 单独建一套完整状态机。拒绝，因为这会重复太多第 2 步 / 第 9 步控制流。

### 6. 明确约束 SMSBower 进入 HeroSMS 专属高级路径

当 active provider 是 `smsbower` 时，下列路径必须有清晰约束，而不是静默落空：

- 激活复用
- 临时租号
- 白嫖复用准备
- 自动白嫖复用

原因：

- 当前实现里，这些能力只在 HeroSMS（以及部分 5sim 复用）上有明确语义。
- 静默落空会造成隐蔽的生产失败。

实现约束：

- 激活复用、白嫖复用准备、自动白嫖复用保持明确失败。
- 临时租号不调用 HeroSMS 的 `getRentNumber` 语义，而是在侧边栏和运行时自动降级为普通取号，并给出清晰提示。

备选方案：

- 先假装 SMSBower 和 HeroSMS 一样，后续再修。拒绝，风险太高。

### 7. 把 SMSBower 加入默认 provider 顺序，而不是改掉现有首选顺序

当前默认顺序是围绕既有 provider 构建的。SMSBower 会作为可用顺序选项加入，并追加到默认回退顺序里，而不是替换现有首选顺序。

原因：

- 这样不会在升级后悄悄改变老用户的主接码行为。

备选方案：

- 让 SMSBower 成为默认第一位。拒绝，因为这次是增量接入，不是迁移出既有 provider。

## 风险 / 取舍

- `[API 域名不一致或 Cloudflare 行为差异]` → 使用已验证的官方域名 `smsbower.page`，收窄 fetch 逻辑，并在 SMSBower 失败时继续保留其它 provider 回退。
- `[动态国家拉取失败导致 UI 不可用]` → 单独持久化已选国家标签，在失败时清晰提示错误，同时保留已有选择。
- `[handler-api 兼容性并非每个响应都与 HeroSMS 完全一致]` → 使用 provider 专属请求和解析 helper，不要盲目复用 HeroSMS parser；并用回归测试覆盖预期结构。
- `[background/phone-verification-flow.js 内部分支更多]` → 把 SMSBower helper 聚合清晰，方便后续抽离 provider 层。
- `[不支持的高级流程让用户困惑]` → 在运行逻辑和必要的 UI 行为中，用明确的 provider 级提示直接拦截；对临时租号则降级为普通取号，避免整轮拿号失败。

## 迁移计划

1. 先新增 provider ID 和默认设置，credentials 和国家顺序保持空值。
2. 再加侧边栏 provider 选项和动态国家加载能力。
3. 然后补后台请求 / 轮询 / 关闭 helper，并接入 provider 选择和回退顺序。
4. 再补复用和临时租号等不支持路径的拦截。
5. 最后补回归测试并更新仓库文档。

回滚策略：

- 本次改动是增量式的，相关设置都用 `smsBower*` 命名空间隔离。
- 回滚时可以直接移除 provider 选项并忽略新增设置，不需要迁移既有用户数据。

## 未决问题

- `getNumber` 和 `getNumberV2` 哪个更稳定，最适合当前仓库的激活 payload 解析假设，需要在实现中最终确认。
- `getPricesV3` 的 provider 级详情是否值得在 v1 使用，还是 `getPricesV2` 已足够做价格档位排序，需要在实现时拍板。
- 侧边栏在 v1 是否要单独暴露一个“刷新国家”按钮，可以等 UI 接线完成后再决定。
