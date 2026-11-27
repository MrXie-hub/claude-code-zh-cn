# Windows x64 · Claude Code 2.1.332

2026-11-27，在 Windows 11 x64、Node.js 24.12.0、`node-lief@1.3.2` 上，使用 `@anthropic-ai/claude-code-win32-x64@2.1.332` 的独立副本验证。下载包的 SHA-512 与 npm 官方 registry 公布的 `dist.integrity` 一致；原始 `claude.exe` 的 SHA-256 为 `e6652f9170a07df45f1db3e0520f1df0f7b385181ff74f70f49398a67b5fd320`。

| 同一个 `2.1.332` 官方构建 | 旧译表（2.24.0） | 本次发布（2.25.0） |
| --- | ---: | ---: |
| 字节码字符串替换 | 1107 | 1112 |
| 正常退出的帮助入口 | 11 / 11 | 11 / 11 |
| 帮助审计的英文行警告 | 133 | 103 |

五条补译分别是 `--name`（含云会话名称）、`--restricted`（含插件语言服务器与写入审批边界）、`--fallback-model`、`--from-pr` 和 `--resume`。参数、权限值及 CLI 行为未改，仍使用通用字节码补丁。警告数按帮助文本换行统计，并非翻译覆盖率；本次未验证全部交互菜单、macOS、Linux 或 `2.1.328` 至 `2.1.331`。

**升级实测：** 在隔离用户目录安装插件，把 `2.1.327` 副本替换成 `2.1.332` 官方原件，不重装插件，通过 PowerShell 启动器打开帮助。新版重新显示中文，版本仍为 `2.1.332`；卸载后 SHA-256 匹配 **新版** 原件。空词表、重复打补丁和同版本原件覆盖后的启动修复也通过。

**复验：** 在仓库根目录用 PowerShell 7 运行：

```powershell
node scripts/verify-upstream-compat.js --baseline 2.1.332 --skip-latest --native-windows-x64 --fail-on-skip --verify-install --json
```

其他设备的版本选择和安装步骤见 [README](../README.md#-极速安装)。

## 跟进下一版

把下面的 `$version` 改成实际版本，在 Windows x64 仓库根目录运行。验证报告放在临时目录，不要直接提交；只有验证通过后才推广支持窗口。

```powershell
$version = '2.1.333'
$report = Join-Path $env:TEMP "cczh-win-$version.json"
node scripts/verify-upstream-compat.js --baseline $version --skip-latest --native-windows-x64 --fail-on-skip --verify-install --json > $report
if ($LASTEXITCODE -ne 0) { throw '兼容性验证失败：保留报告，不更新支持清单。' }
node scripts/promote-native-candidate.js --candidate $report --platform windows --write
node scripts/generate-plugin-support-window.js --write
node scripts/generate-support-matrix.js
node scripts/sync-readme-support-window.js --write
node scripts/sync-doc-derived-counts.js --write
```

如果有新帮助文案，再修改根目录 `cli-translations.json`，运行 `bash scripts/sync-payload.sh`，然后用新版真实 `--help` 输出复核。不要仅把版本号写进支持清单。
