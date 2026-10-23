# CodexReflow

**Automatically continue eligible GPT requests.**

CodexReflow is a continuation plugin for [CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI) (CPA). When a reasoning-token count matches a specific pattern, the plugin checks upstream reasoning state and continuation limits, decides whether to request another round, and combines the results for your client.

**Project origins:** Independently maintained fork of [uf-hy / CPA CodexComp v0.1.7](https://github.com/uf-hy/cpa-plugin-codexcomp/tree/v0.1.7), carrying forward continuation algorithms and mechanisms originally created by [dzshzx / codexcomp](https://github.com/dzshzx/codexcomp) and [neteroster / CodexCont](https://github.com/neteroster/CodexCont).

[简体中文](README.md) · [Download v0.1.1](https://github.com/chatwithgod/cpa-plugin-codexreflow/releases/tag/v0.1.1) · [Report a problem](https://github.com/chatwithgod/cpa-plugin-codexreflow/issues/new?template=bug_report.md)

## What it does

- **Automatic continuation**: Triggers follow-up rounds based on numeric rules, upstream reasoning state and continuation limits, not semantic guesses about whether an answer feels finished.
- **Zero model-list upkeep**: Out of the box, it automatically matches lowercase standard GPT-5+ text model names without requiring manual lists.
- **Dual protocol support**: Handles streaming output over both HTTP/SSE and WebSocket.

> **Note**: Continuation rounds consume additional tokens and increase response latency. The plugin does not "cure model degradation," cannot guarantee deeper reasoning or smarter answers, and does not unconditionally continue every request that shows 516.

## Installation and use

The easiest way is via the **CPA / ECPA Plugin Store**:

1. Open **Plugins → Plugin Store** and search for **CodexReflow**.
2. Click **Install**. Leave the version blank for the latest release, or enter `v0.1.1`.
3. Enable CodexReflow in your local plugin list and confirm its status shows **Running**.
4. Use your client as usual. **Fresh installations require no handwritten YAML and no model list.**

**Important reminders**:

- If you were using CodexComp, disable it first so that both plugins do not handle the same models at once.
- Before updating versions, keep a copy of the previous plugin and configuration, and wait for ongoing responses to complete.

<details>
<summary>Manual downloads for v0.1.1 (click to expand)</summary>

Download the release archive for your operating system and architecture:

| Platform | Download link |
|---|---|
| Windows x64 | [codexreflow_0.1.1_windows_amd64.zip](https://github.com/chatwithgod/cpa-plugin-codexreflow/releases/download/v0.1.1/codexreflow_0.1.1_windows_amd64.zip) |
| Linux x64 | [codexreflow_0.1.1_linux_amd64.zip](https://github.com/chatwithgod/cpa-plugin-codexreflow/releases/download/v0.1.1/codexreflow_0.1.1_linux_amd64.zip) |
| Linux ARM64 | [codexreflow_0.1.1_linux_arm64.zip](https://github.com/chatwithgod/cpa-plugin-codexreflow/releases/download/v0.1.1/codexreflow_0.1.1_linux_arm64.zip) |
| macOS Intel | [codexreflow_0.1.1_darwin_amd64.zip](https://github.com/chatwithgod/cpa-plugin-codexreflow/releases/download/v0.1.1/codexreflow_0.1.1_darwin_amd64.zip) |
| macOS Apple Silicon | [codexreflow_0.1.1_darwin_arm64.zip](https://github.com/chatwithgod/cpa-plugin-codexreflow/releases/download/v0.1.1/codexreflow_0.1.1_darwin_arm64.zip) |

Verify SHA256 hashes against [checksums.txt](https://github.com/chatwithgod/cpa-plugin-codexreflow/releases/download/v0.1.1/checksums.txt). For platform coverage and known constraints, see the [compatibility notes](docs/COMPATIBILITY.md).

</details>

## Default behavior and configuration

**Most users can run with defaults without modifying configuration.**

| Option | Default behavior |
|---|---|
| **Model matching (`model_mode: auto`)** | Matches lowercase standard GPT-5+ text model names (e.g. `gpt-6-astra`, `gpt-6-sol`, `gpt-6-luna`, including routing prefixes and reasoning suffixes). |
| **Continuation limit (`max_continue: 3`)** | Appends up to 3 additional rounds per request (maximum 4 total rounds including the initial request). |
| **Automatic exclusions** | Model names containing non-text segments such as `image`, `audio`, or `realtime` are automatically excluded. |
| **Diagnostics (`debug_log: false`)** | Detailed debug logging is disabled by default; essential continuation diagnostics remain active. |

**Key points**:

- Auto mode matches the **model name provided in the client request**. It does not sync CPA's model catalog or detect underlying model capabilities. Custom aliases like `my-coder` require manual list configuration.
- **Upgrading from older configs**: If your old configuration contains a `models` list without an explicit `model_mode`, upgrading preserves its legacy list behavior. To switch to automatic matching, explicitly set `model_mode: auto` (which ignores legacy `models`).
- For custom exclusions, round limits, or manual lists, see the [configuration reference](docs/CONFIGURATION.md).

## FAQ and troubleshooting

### Why do usage records still show 516?

CPA may log the initial request and each continuation round separately. Even after a successful continuation, the first round's 516 entry remains in the log, and follow-up rounds can legitimately add 0 new reasoning tokens.

**Seeing 516 does not mean the plugin failed, nor does it prove model degradation.** The plugin triggers only when requests meet continuation rules alongside usable upstream reasoning state—it does not blindly resend every 516 request.

### How do I verify that continuation actually happened?

Do not rely on the plugin switch status or a single token count alone. Check the logs for that specific request:

1. Did the request log any continuation rounds?
2. Did the final round complete normally or encounter an error?
3. Did your client successfully receive the combined output? (Host acceptance does not guarantee client delivery.)

For log fields and inspection tools, see the [diagnostics guide](docs/DIAGNOSTICS.md).

### Does WebSocket continuation work?

Yes. Subsequent rounds require complete, available context and valid upstream encrypted reasoning state, along with the remaining continuation conditions. If context is missing, expired, or exceeds cache limits, that request may not be continued. Not all client and platform combinations have undergone full validation.

### What should I do if requests freeze or return errors?

1. Temporarily disable CodexReflow in your CPA plugin list and send a message with the same model. If normal behavior resumes, include that observation in your report to help narrow down the cause.
2. When reporting issues, include CPA/ECPA, plugin, and client versions alongside sanitized error logs.
3. Never post API keys, account credentials, full configs, or private chat histories. [Submit a report](https://github.com/chatwithgod/cpa-plugin-codexreflow/issues/new?template=bug_report.md).

## Privacy and data handling

To stitch consecutive rounds together, the plugin temporarily caches conversation context and encrypted reasoning state in local process memory.

Default diagnostic logs **never record** prompts, model outputs, credentials, or encrypted reasoning data. Logging behaviors in CPA itself or other third-party plugins follow their own separate configurations.

## Further reading

*Note: The configuration, diagnostics and development guides are currently written in Chinese.*

- [Configuration](docs/CONFIGURATION.md): Automatic vs. manual modes, exclusions, round limits, and legacy config migration.
- [Diagnostics](docs/DIAGNOSTICS.md): Tracking continuation rounds, stop reasons, and token usage.
- [Compatibility and limits](docs/COMPATIBILITY.md): Platform test coverage, WebSocket context requirements, and cache limits.
- [Development](CONTRIBUTING.md) · [Multiplatform builds](docs/MULTIPLATFORM.md) · [Releasing](docs/RELEASING.md) · [Changelog](CHANGELOG.md)

## Attribution, acknowledgments, and license

CodexReflow builds upon the contributions of these upstream projects:

- **Direct upstream: [uf-hy / CPA CodexComp v0.1.7](https://github.com/uf-hy/cpa-plugin-codexcomp/tree/v0.1.7)**: Directly forked and independently maintained, inheriting its C ABI bridge, configuration, routing, and Go continuation logic.
- **Algorithm source: [dzshzx / codexcomp](https://github.com/dzshzx/codexcomp)**: Upstream Go response folding and merging algorithms were ported from this project's `fold.py`.
- **Mechanism source: [neteroster / CodexCont](https://github.com/neteroster/CodexCont)**: Origin of the `518n-2` truncation detection heuristic and encrypted reasoning replay pattern.

Thanks to these projects and authors for their foundational work. CodexReflow adds improved HTTP/SSE and WebSocket handling, automatic model matching, and enhanced diagnostics. It is maintained independently and is not an official OpenAI or CLIProxyAPI project.

Released under the [MIT License](LICENSE), retaining original copyright and permission notices. Full third-party notices and licenses are documented in [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md) and [THIRD_PARTY_LICENSES.txt](THIRD_PARTY_LICENSES.txt).
