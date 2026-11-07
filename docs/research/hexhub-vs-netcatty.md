# HexHub 5.1.9 vs Netcatty 能力对比

> 采样对象：`/Applications/HexHub.app`（CFBundleVersion 1.0，内部版本 5.1.9 / build 202606072152，构建日期 2026-06-07）
> 对比对象：本仓库 Netcatty（Electron + React，GPL-3.0）
> 采样时间：2026-11-07

## 0. 方法与证据边界

| 来源 | 用途 |
| --- | --- |
| `Contents/Info.plist`、bundle 布局 | 版本、构成、平台 |
| `Contents/MacOS/hexhub-backend`（Go，120 MB）`strings` + 模块路径 | 后端语言、第三方库、协议栈、DB 驱动 |
| 以临时 HOME 启动后端（`127.0.0.1:<随机端口>`，未登录、无 token） | 拉取内嵌前端 `dist/`，得到完整路由/组件/文案 |
| 前端 325 个 JS chunk（8.9 MB）静态分析 | 功能面、AI 供应商、会员/账号逻辑、UI 能力 |
| 本仓库源码（`components/` 128 个、`electron/bridges/` 112 个、1700 个测试文件） | Netcatty 能力面 |

未执行：未登录 HexHub 云端、未建立真实 SSH/数据库/Docker 会话。功能存在性来自代码路径、UI 文案与依赖，深度与稳定性为**推断**，已在文中标注。

## 1. 结论先行：两者不是同一类产品

- **HexHub**：商业化「一站式 IT 工具箱」——SSH/SFTP/FTP/Telnet/串口/RDP 跳板 + 7 种数据库 GUI + Redis + Docker + 内嵌 AI Agent，账号体系 + 付费会员 + 云端加密资产。闭源。
- **Netcatty**：开源「终端工作台 + AI Agent 平台」——SSH/SFTP/Mosh/Telnet/串口/本地终端 + 分屏 workspace + 插件运行时 + MCP/CLI + FIDO2 等认证深度。GPL-3.0。

重叠区是 SSH/SFTP/终端；**不重叠区很大**：HexHub 的数据库/Docker 是 Netcatty 完全空白，Netcatty 的插件平台、MCP/CLI、终端工程深度是 HexHub 空白。

## 2. 技术构成

| 维度 | HexHub 5.1.9 | Netcatty |
| --- | --- | --- |
| 外壳 | 自研 CEF 宿主（`Chromium Embedded Framework.framework`，Chrome 138.0.7204.97，234 MB） | Electron 42 |
| 后端 | Go 单二进制 `hexhub-backend`（120 MB），本地 HTTP + Unix socket IPC | Node/Electron 主进程，112 个 `bridges/*.cjs`，按域 IPC |
| 前端 | Vue 3 + Vuetify + Pinia + vue-i18n + Dexie + axios + CodeMirror + xterm + chart.js（325 chunk） | React 19 + Tailwind + Radix + xterm 6 + Monaco + AI SDK 7 |
| 包体 | 358 MB（CEF 234 + Go 120 + 前端 8.9） | 未打包对比，量级相近（Electron ~200 MB 级） |
| 进程模型 | CEF 多 Helper 进程 + Go 后端（两套运行时） | Electron 多进程 + `terminalWorker` + `utilityProcess` 插件宿主 |
| 协议栈 | `golang.org/x/crypto/ssh`、`pkg/sftp`、`jlaffaye/ftp`、`go.bug.st/serial`、Docker `distribution`、`minio-go`、`go-git` | `ssh2`、`ssh2-sftp-client`、`serialport`、`node-pty`、mosh 外部二进制 |
| 数据库驱动 | pgx、go-sql-driver/mysql、mattn/go-sqlite3、mssqldb、godror/go-ora（Oracle）、ClickHouse、redis、达梦、人大金仓 | 无 |
| AI | `cloudwego/eino` + 自建 provider registry | Vercel AI SDK 7 + `@openai/codex-sdk` + `@ai-sdk/{openai,anthropic,google}` |

架构差异的实际含义：

- HexHub 用 CEF + Go，渲染层与业务层彻底分离，前端就是一个被 Go 服务的 SPA；**好处**是后端能力（数据库、Docker、SSH、AI）不受 Node 生态限制，Go 的并发/驱动成熟度更高；**代价**是两套运行时、两套崩溃模型、包体翻倍。
- Netcatty 全栈 TypeScript，能力靠 bridge 划分，插件与 MCP/CLI 复用同一套 capability catalog；**好处**是扩展成本低、可测试（1700 个测试文件）；**代价**是重后端能力（数据库驱动、Docker API、RDP）在 Node 侧要么缺库要么体验差。

## 3. 功能矩阵

| 能力 | HexHub | Netcatty | 谁强 |
| --- | --- | --- | --- |
| SSH 会话 / 跳板 / Proxy | ✅ 含 jump、socks 代理、`tunnel-autostart` | ✅ 含 jump、SOCKS5、代理链、连接池、idle park | 平（Netcatty 连接管理更工程化） |
| SFTP | ✅ 双栏文件浏览器、编辑、同步历史 | ✅ 双栏、拖拽、内置编辑器、断点续传、全局传输中心 | Netcatty 略强 |
| SCP / FTP | ✅（`Ftp` chunk + `jlaffaye/ftp`） | ❌ | HexHub |
| Telnet | ✅ | ✅（含自动登录） | 平 |
| 串口 | ✅ | ✅ | 平 |
| RDP | ⚠️ 有资产类型与 `rdp/open` RPC，但后端无 RDP 协议栈 → 调系统远程桌面客户端 | ❌ | HexHub（但只是壳） |
| Mosh | ❌ | ✅（`moshHandshake` + 外部二进制） | Netcatty |
| X11 转发 | ⚠️ 仅见 `x11` 文案 | ✅ `x11Forwarding` bridge | Netcatty |
| ZMODEM | ✅ | ✅（zmodem/ymodem） | 平 |
| 终端分屏 / 多标签 | ✅（`splitVertical`、FlexTabs、SshMixView） | ✅（workspace 树、任意分割、侧栏分屏） | Netcatty 更强（布局模型是纯函数 domain） |
| 内联图片 | ⚠️ 见 iTerm 相关文案，无 sixel/kitty | ✅ Kitty / SIXEL / iTerm（xterm addon-image） | Netcatty |
| 输入广播 / 批量执行 | ✅ `SshInputBroadcast`、`BatchExecDialog` | ✅（多会话广播） | 平 |
| 命令历史 / 自动补全 / 高亮 | ✅ `CmdHistoryPopup`、`command-autocomplete`、`TermHighlightAddon` | ✅（历史面板、补全、关键字高亮） | 平 |
| 数据库 GUI | ✅ MySQL / PostgreSQL / SQLite / SQL Server / Oracle / ClickHouse / 达梦 / 人大金仓 + 实时监控 + 数据/结构同步 | ❌ | **HexHub 碾压** |
| Redis GUI | ✅ 配置 + 实时信息 | ❌ | HexHub |
| Docker GUI | ✅ 容器/镜像/Compose/Registry/Swarm/实时信息/导入导出 | ❌（Agent 可代跑命令） | **HexHub 碾压** |
| 服务器监控面板 | ✅ CPU/内存/磁盘 RealtimeInfo + 图表 | ⚠️ 仅 AI 侧读上下文 | HexHub |
| AI Agent | ✅ 终端内嵌 Agent、多 provider（OpenAI/Azure/Claude/Gemini/DeepSeek/Kimi/MiniMax/Qwen/百炼/OpenRouter/Ollama/ByteDance）、细分权限（命令/读文件/写文件/只读 SQL）、后台任务、token 统计 | ✅ Catty Agent、多 SDK、compaction/预算/trace、工具审批、MCP Server + CLI、外部 MCP | Netcatty 更工程化，HexHub 更"开箱即用" |
| 插件平台 | ❌ 闭源单体 | ✅ 沙箱运行时 + 权限说明 + host-rendered UI + 契约 codegen | Netcatty |
| 账号 / 云同步 | ✅ 账号 + 会员 + 激活码 + 设备数限制（2/5 台）+ 云端加密资产 | ✅ 可选，自带 S3/WebDAV/GitHub/OneDrive 等，无账号体系 | 取向不同 |
| 认证深度 | ⚠️ 私钥/密码/Keychain；TOTP 仅见账号 2FA | ✅ FIDO2 安全密钥（`sk-ssh-keygen`/askpass/签名流）、TOTP 主机字段、known_hosts 策略 | Netcatty |
| 凭据存储 | ✅ macOS Keychain（`credentials not found in native keychain`） | ✅ 加密凭据 + 隐私桥 | 平 |
| 应用锁 | ✅（"启动时询问密码"） | ✅（生物识别/Windows Hello + 覆盖层） | 平 |
| 主题 / 个性化 | ✅ 明暗 + 图标 + 布局 | ✅ 主题、强调色、终端主题、图标 | 平 |
| 国际化 | ✅（vue-i18n，中英） | ✅ en / es / zh-CN / zh-TW / ja | Netcatty |
| 自动更新 | ✅ 含热更新状态、客户端下载页（Mac/Win/Linux） | ✅ electron-updater | 平 |
| 许可 | 商业订阅 | GPL-3.0 | — |

## 4. HexHub 做对了什么（值得抄的产品决策）

1. **资产模型统一**：所有连接类型都是 asset（`ssh-asset-operate`、`docker-asset-operate`、`mysql-asset-operate`…），左侧一棵树容纳万物。用户迁移成本极低——"我所有环境都在一个窗口里"。
2. **数据库覆盖对齐中国企业环境**：除主流 5 种，补了**达梦 / 人大金仓**，这是 Navicat/DBeaver 之外的差异化。
3. **DB 不只是查询器**：实时监控（连接数/QPS/慢查询）、表结构对比、数据同步、跨库复制、XLSX 导出（`excelize`）。
4. **AI 嵌进终端而不是另开聊天窗**：`LLMTerminal` 直接接管终端输入，Agent 的命令以终端块（`TerminalShellCommandBlock`）形式落地，权限粒度到"静默执行/静默读文件/静默写文件/只读 SQL"——比"一个侧栏对话框"更贴工作流。
5. **运维日常功能密度高**：命令历史、自动补全、输入广播、批量执行、后台任务抽屉、服务器实时面板。
6. **商业闭环完整**：试用 → 会员 → 激活码 → 设备管理 → 热更新 → 客户端下载页。可复制的是"闭环"，不是"闭源"。

## 5. HexHub 的硬伤（也是 Netcatty 的机会）

1. **账号强绑定 + 闭源**：云端加密资产绑定账号，换账号导入直接失败（前端文案"资产绑定账号与当前账号不一致"）。离线/审计/合规场景无法自证逻辑，企业采购阻力大。
2. **前端可被完整提取**：本地 HTTP 服务无鉴权直接吐 `index.html` 与全部 325 个 chunk（本次即以此获得全量前端）。"闭源"只是 LICENSE 层面的事实，不是技术保护。
3. **双运行时成本**：CEF 234 MB + Go 120 MB；崩溃/内存/升级要维护两套。对用户是启动与常驻内存开销。
4. **本地服务攻击面**：SPA 通过 `127.0.0.1:<随机端口>` + token 通信，本机任意进程可探测端口、加载 UI；IPC 走 `/tmp/hexhub.sock`。隔离性依赖约定而非系统边界。
5. **RDP 是壳**：无内建 RDP 协议栈，靠系统客户端；跨平台一致性差。
6. **终端工程深度不足**：无 Mosh、无 SIXEL/Kitty、无插件生态，终端侧只能做到"够用"。
7. **会员上限**（2/5 台设备）会劝退多机运维与极客用户。

## 6. Netcatty 的相对优势

1. **开源可审计 + 数据本地优先**：vault 全本地，云同步是可选且用用户自己的存储。
2. **终端工程质量**：xterm 6 + WebGL + ligatures + graphemes + image、`terminalWorker`、流控（flow ack/pause arbiter）、PTY 进程树、编码处理、会话日志脱敏。
3. **AI 工程化**：`AgentRuntime` 统一 turn 生命周期、上下文预算/压缩（pre-turn + 413 + step 剪枝）、工具输出句柄、trace/usage、外部 SDK 驱动；并且把能力**暴露给外部 agent**（MCP server + CLI）——这是 HexHub 没有的"平台化"。
4. **插件平台**：`.ncpkg` 校验、沙箱 BrowserWindow/utilityProcess、RPC 路由、host-rendered UI（不把插件代码放进宿主渲染器）——真正可扩展。
5. **认证与安全面宽**：FIDO2 硬件密钥全链路、askpass/passphrase、known_hosts 策略、TOTP、应用锁、隐私桥。
6. **协议广度**：Mosh、X11、ZMODEM/YMODEM、串口、Telnet、本地 shell。
7. **可维护性**：domain 纯函数 + 1700 个测试，重构与回归成本低。

## 7. Netcatty 的相对劣势

1. **没有数据库 GUI**——与 HexHub 用户重叠度最高、感知最强的差距。
2. **没有 Docker / Redis 面板**。
3. **没有 FTP / SCP 资产类型**（SCP 可用 SFTP 覆盖，但资产模型上缺一类）。
4. **没有 RDP**。
5. **没有服务器实时监控面板**（HexHub 的 RealtimeInfo + 图表是"看起来专业"的关键）。
6. **没有账号/会员体系**：对个人开源是优点，对"企业统一采购 + 团队资产"叙事是短板。
7. **首次使用引导弱于 HexHub**：HexHub 用"一站式"降低选择成本，Netcatty 的能力需要用户自己发现（插件、MCP、Agent）。

## 8. 如果只做三件事（按投入产出排序）

1. **数据库连接器 MVP**：PostgreSQL / MySQL / SQLite 三选一先做只读查询 + 结果表 + 导出 CSV。复用现有 vault 资产模型与 SSH 隧道（"数据库只在内网、必须过跳板"是 Netcatty 的天然优势场景，HexHub 也支持但你的隧道更成熟）。
2. **服务器实时面板**：复用 SSH exec 采集 `top/free/df`，前端 chart 化，挂在终端侧栏——低成本、高感知。
3. **Docker 面板**：list / start / stop / logs / exec，命令通道已有（`boundedSshExec`），只缺 UI 与状态机；Catty 已能代跑，UI 化即可形成"人机双通道"卖点。

## 9. 不要照抄

- **CEF + Go 双运行时**：收益（Go 驱动生态）可用 sidecar 或插件进程替代，不必重写外壳。
- **账号强绑定云资产**：会牺牲开源项目的信任优势；同步继续走"用户自有存储"。
- **本地 HTTP 无鉴权吐前端**：如果将来引入本地服务，必须做 per-session token + Origin/Host 校验 + 只绑 loopback（HexHub 只做到了最后一条）。
- **会员设备数限制**：对开发者工具是负口碑设计。

## 10. 待验证（本轮未做）

- HexHub 数据库 GUI 的实际编辑能力（事务、DDL、大结果集分页）与稳定性。
- Docker 面板对远程 daemon / TLS / SSH 隧道的支持程度。
- AI Agent 的工具审批是否可被绕过（`allowSilent*` 的持久化粒度）。
- 会员校验是否纯客户端（若纯前端判断，则存在绕过面）。
