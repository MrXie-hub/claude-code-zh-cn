# Windows x64 · Claude Code 2.1.327

## 实测结论

2026-11-18，使用 Windows 11 x64、Node.js 24.12.0、`node-lief@1.3.2`，在独立配置目录和 `@anthropic-ai/claude-code-win32-x64@2.1.327` 的 `claude.exe` 副本上验证。原件 SHA-256：`d6c8075bfb3b87f1c1254e64b6c35dadc628d21e75930ae51e47047b3f05a4a2`。不需要账号或模型请求来验证帮助输出。

| 验收项 | 结果 |
| --- | --- |
| 原生格式、版本 | Bun 字节码；打补丁后 `--version` 仍为 `2.1.327 (Claude Code)` |
| 翻译命中 | 共 1140 处字符串替换；仅使用通用字节码补丁，不套用 2.1.289 的显示模块重写 |
| 显示面 | 11 个帮助入口退出码均为 0；显示审计为 **PARTIAL**，128 条英文行警告（不是 128 个启动错误，也不是覆盖百分比） |
| 失败与还原 | 空词表不改写程序；重复打补丁后字节不变；还原后 SHA-256 与原件相同 |
| 插件识别 | 同一副本在支持清单更新后标记为 `verified`，不是 `provisional` |
| PowerShell 安装链 | 隔离用户目录内的安装、doctor 诊断、重复安装、原版覆盖后由启动器自动重补、卸载恢复均通过。未更改日常使用的安装 |

逐项核对了主帮助的 `--add-dir`、`--append-system-prompt`、`--autocompact`、`--brief`、`--chrome`、`--no-chrome`、`--disable-slash-commands`，以及 `agents --help` 的 `--add-dir`、`--json`、`--plugin-dir`。这 10 项现在显示中文，参数拼写不变。这里验证的是命令行帮助，不是对交互菜单、权限弹窗或其他平台的全量验证。

仓库回归：Windows 上运行所有 `tests/*.test.js`，422 项通过、0 项失败、92 项跳过；旧 npm 稳定矩阵的 6 个代表版本均通过。跳过项不计作通过。

## 安装这份 fork 的适配

先关闭所有 Claude Code 窗口，再在独立的 PowerShell 中，进入**包含本适配提交的仓库根目录**运行以下命令。不要使用上游 Release 的一键安装脚本，它不会包含 fork 中尚未合并的改动。

```powershell
.\install.ps1
claude --version
claude --help
claude agents --help
```

例如，主帮助的 `--chrome` 应显示“启用 Chrome 集成”，`--disable-slash-commands` 应显示“禁用所有技能和命令”。卸载使用同一目录的 `.\uninstall.ps1`。

## 跟进下一个版本

在 Windows x64 仓库根目录，用 PowerShell 7、Node.js 和 `node-lief@1.3.2` 运行。验证器下载指定版本的官方平台包并对副本操作；临时报告不入库。

```powershell
$version = '2.1.327' # 下次改成实际待测的版本
$report = Join-Path $env:TEMP "cczh-win-$version.json"
node scripts/verify-upstream-compat.js --baseline $version --skip-latest --native-windows-x64 --fail-on-skip --json > $report
if ($LASTEXITCODE -ne 0) { throw '兼容性验证失败：保留报告，不更新支持清单。' }
node scripts/promote-native-candidate.js --candidate $report --platform windows --write
if ($LASTEXITCODE -ne 0) { throw '验收证据不足，不更新生成文件。' }
node scripts/generate-plugin-support-window.js --write
node scripts/generate-support-matrix.js
node scripts/sync-readme-support-window.js --write
node scripts/sync-doc-derived-counts.js --write
```

新文案改根目录的 `cli-translations.json`，再运行 `bash scripts/sync-payload.sh`，用新版的真实帮助输出复核。若要声明 macOS 或 Linux 支持，分别在对应平台运行仓库的 `Native Latest Candidate` 工作流验证；不要仅修改版本上限。
