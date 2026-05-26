## Why

当前手机号注册链路在密码页出现“创建帐户失败，请重试”时，会把它当成可恢复停留态，继续重复点击“继续”，既浪费当前轮时间，也会继续占用一个大概率已经失效的接码订单。与此同时，接码平台返回的手机号并不总是带 `+` 前缀，导致“登录或注册”页填入的号码格式不稳定。

现在需要把这两类问题都前移到共享手机号注册链路中处理，减少无效重试，并统一手机号输入格式。

## What Changes

- 在手机号注册的密码页恢复链路中，将“创建帐户失败，请重试”识别为当前手机号不可继续使用的硬失败。
- 命中该错误时，停止继续点击“继续”，改为丢弃当前注册手机号并回到步骤 1 重开当前轮。
- 如果当前手机号存在接码 activation，则在重开前 best-effort 调用对应 provider 的取消 / 释放接口。
- 在“登录或注册”页填写手机号前，统一将不带 `+` 前缀的号码规范化为带 `+` 的 E.164 风格字符串后再提交。
- 为上述行为补回归测试，并更新相关设计文档与任务状态。

## Capabilities

### New Capabilities
- `phone-signup-flow-guardrails`: 约束手机号注册链路中的不可恢复密码页错误处理，以及手机号输入前的格式归一化。

### Modified Capabilities

## Impact

- 受影响代码：
  - `content/signup-page.js`
  - `background.js`
  - `background/phone-verification-flow.js`
  - 手机号输入或手机号注册相关步骤模块
  - 对应测试文件
- 受影响系统：
  - HeroSMS / 5sim / SMSBower / NexSMS 的 activation 取消能力
- 文档：
  - 本 change 的设计与任务文档
