# CodexReflow · 续流

**让符合条件的 GPT 请求自动接着生成。**

CodexReflow 是一个针对 [CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI)（CPA）的续接插件。当一次推理返回的 token 数命中特定规则时，插件会结合上游推理状态和续接限制，决定是否追加请求，并把多轮结果合并后返回客户端。

**项目来源：** 本项目基于 [uf-hy / CPA CodexComp v0.1.7](https://github.com/uf-hy/cpa-plugin-codexcomp/tree/v0.1.7) 独立维护，沿用源自 [dzshzx / codexcomp](https://github.com/dzshzx/codexcomp) 与 [neteroster / CodexCont](https://github.com/neteroster/CodexCont) 的续接机制与算法。

[English](README_EN.md) · [下载 v0.1.1](https://github.com/chatwithgod/cpa-plugin-codexreflow/releases/tag/v0.1.1) · [反馈问题](https://github.com/chatwithgod/cpa-plugin-codexreflow/issues/new?template=bug_report.md)

## 它能做什么？

- **自动追加续接**：根据数字规则、上游推理状态和续接限制，自动发起后继请求并合并回答，不是靠理解语义去猜回答有没有说完。
- **免手动维护名单**：新配置默认按名称匹配 GPT-5 及以上的标准文本模型，无需手写模型列表。
- **支持流式与双协议**：同时支持 HTTP/SSE 与 WebSocket 数据流。

> **提示**：追加轮次会增加 token 消耗和响应等待时间。插件不会“治愈降智”，不保证模型一定思考更多或变得更聪明，也不会无条件续接所有 516 请求。

## 安装与使用

推荐直接在 **CPA / ECPA 插件商店**安装：

1. 打开「插件 → 插件商店」，搜索 **CodexReflow**。
2. 点击安装（版本留空默认安装最新发布版，也可手动填写 `v0.1.1`）。
3. 在本地插件列表中启用 CodexReflow，确认显示为“运行中”。
4. 像往常一样在客户端使用即可。**全新配置开箱即用，不需要手写 YAML，也不用配置模型名单。**

**使用提示**：

- 如果你之前使用过 CodexComp，请先停用它，避免两个插件同时接管同一模型。
- 更新插件版本前，建议先保留旧插件并备份现有配置，等待当前正在生成的回答全部结束。

<details>
<summary>手动下载 v0.1.1（点击展开）</summary>

按操作系统与架构下载对应压缩包：

| 平台 | 下载链接 |
|---|---|
| Windows x64 | [codexreflow_0.1.1_windows_amd64.zip](https://github.com/chatwithgod/cpa-plugin-codexreflow/releases/download/v0.1.1/codexreflow_0.1.1_windows_amd64.zip) |
| Linux x64 | [codexreflow_0.1.1_linux_amd64.zip](https://github.com/chatwithgod/cpa-plugin-codexreflow/releases/download/v0.1.1/codexreflow_0.1.1_linux_amd64.zip) |
| Linux ARM64 | [codexreflow_0.1.1_linux_arm64.zip](https://github.com/chatwithgod/cpa-plugin-codexreflow/releases/download/v0.1.1/codexreflow_0.1.1_linux_arm64.zip) |
| macOS Intel | [codexreflow_0.1.1_darwin_amd64.zip](https://github.com/chatwithgod/cpa-plugin-codexreflow/releases/download/v0.1.1/codexreflow_0.1.1_darwin_amd64.zip) |
| macOS Apple Silicon | [codexreflow_0.1.1_darwin_arm64.zip](https://github.com/chatwithgod/cpa-plugin-codexreflow/releases/download/v0.1.1/codexreflow_0.1.1_darwin_arm64.zip) |

下载后建议通过 [checksums.txt](https://github.com/chatwithgod/cpa-plugin-codexreflow/releases/download/v0.1.1/checksums.txt) 校验 SHA256。各平台的测试覆盖与已知限制见[兼容性说明](docs/COMPATIBILITY.md)。

</details>

## 默认规则与配置

**绝大多数用户直接使用默认设置即可，无需改动。**

| 设置项 | 默认行为 |
|---|---|
| **模型匹配 (`model_mode: auto`)** | 按请求名称自动匹配小写的标准 GPT-5 及以上文本模型（如 `gpt-6-astra`、`gpt-6-sol`、`gpt-6-luna`，支持路由前缀与推理后缀）。 |
| **追加次数 (`max_continue: 3`)** | 最多追加 3 轮；算上首轮，一次请求最多 4 轮。 |
| **自动排除模型** | 模型名称中若带有 `image`、`audio`、`realtime` 等非文本词段，会自动跳过不匹配。 |
| **诊断日志 (`debug_log: false`)** | 默认关闭详细调试；基础续接诊断依然正常记录。 |

**需要注意的事项**：

- 自动模式依据的是**客户端请求里的模型名称**，不会同步 CPA 的模型目录，也不会自动嗅探真实能力。像 `my-coder` 这样的自定义别名，必须在手动模式中填写。
- **旧配置升级**：如果原先只写了 `models` 名单、未指定 `model_mode`，升级后会继续保持旧名单行为；想要改用自动匹配，只需显式设置 `model_mode: auto`（此时旧 `models` 会被忽略）。
- 如需自定义排除、限制轮次或配置手动名单，请参考[配置说明](docs/CONFIGURATION.md)。

## 常见问题与排查

### 为什么用量记录里依然能看到 516？

CPA 可能把首轮请求和后续追加轮次分开记录。即便插件成功续接，首轮的 516 记录依然会保留；后续轮也完全可能增加 0 个思考 token。

**看到 516 既不代表插件失效，也不代表模型被“降智”。** 插件只在满足续接规则且拿到上游有效推理状态时才会续接，不是逢 516 就无条件重发。

### 怎么判断插件是否真的触发了续接？

不要单看插件开关或者某一轮的 token 数。请查看同一次请求的续接日志：

1. 观察同一次请求是否记录了追加轮次；
2. 检查最后一轮是正常完成还是异常终止；
3. 确认客户端是否完整收到了回答（不要把“宿主已接受”误当成“客户端已送达”）。

具体日志指标与排查工具见[诊断说明](docs/DIAGNOSTICS.md)。

### WebSocket 连接能正常续接吗？

支持。但后续轮次的续接依赖完整可用的上下文、有效的上游加密推理状态，以及其余续接条件。如果上下文缺失、过期或超出了缓存大小，该请求可能不会续接。目前并非所有客户端或平台都做了全量验证。

### 遇到请求卡住或报错怎么办？

1. 先在 CPA 插件列表里临时停用 CodexReflow，用相同模型重新发一条消息。如果停用后恢复正常，把这一现象一并反馈，方便缩小排查范围。
2. 提交反馈时，请提供 CPA/ECPA 版本、插件版本、客户端名称及脱敏后的错误日志。
3. 请勿上传 API 密钥、账号凭证、完整配置文件或包含敏感信息的原始聊天记录。[前往反馈问题](https://github.com/chatwithgod/cpa-plugin-codexreflow/issues/new?template=bug_report.md)。

## 数据与隐私说明

为了将多轮回答正确拼合，插件会在本进程内存中临时缓存上下文与加密推理状态。

插件自身的默认诊断日志**不会记录**用户的提示词、模型回答、身份凭证或加密推理原文。CPA 宿主本身或并存的其他插件是否记录日志，取决于各自的配置。

## 更多文档

- [配置说明](docs/CONFIGURATION.md)：自动与手动模式、排除名单、轮次上限及旧配置升级。
- [诊断说明](docs/DIAGNOSTICS.md)：查看续接轮次、停止原因与用量统计。
- [兼容性与已知限制](docs/COMPATIBILITY.md)：测试覆盖范围、WebSocket 上下文条件及缓存限制。
- [开发指南](CONTRIBUTING.md) · [多平台编译](docs/MULTIPLATFORM.md) · [发布流程](docs/RELEASING.md) · [更新日志](CHANGELOG.md)

## 来源、致谢与许可证

CodexReflow 建立在以下开源项目的基础之上：

- **直接上游：[uf-hy / CPA CodexComp v0.1.7](https://github.com/uf-hy/cpa-plugin-codexcomp/tree/v0.1.7)**：本项目由此 CPA 插件直接 fork 并在其基础上独立维护，继承其 C ABI 桥接、配置管理、路由与 Go 续接实现。
- **算法来源：[dzshzx / codexcomp](https://github.com/dzshzx/codexcomp)**：上游 Go 版本的响应合并（fold）与续接逻辑移植自该项目的 `fold.py`。
- **机制来源：[neteroster / CodexCont](https://github.com/neteroster/CodexCont)**：原始的 `518n-2` 截断识别规则与加密推理状态重放机制。

感谢以上项目及其作者的贡献。CodexReflow 在此基础上进一步优化了 HTTP/SSE 与 WebSocket 兼容性、自动模型匹配与运行诊断，独立维护，非 OpenAI 或 CLIProxyAPI 官方项目。

本项目采用 [MIT 许可证](LICENSE)，保留原作者版权声明与许可文件。完整第三方来源与授权说明见 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md) 与 [THIRD_PARTY_LICENSES.txt](THIRD_PARTY_LICENSES.txt)。
