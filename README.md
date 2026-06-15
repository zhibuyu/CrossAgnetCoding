# CrossAgnetCoding

Version: 0.3.0-mvp

CrossAgnetCoding 是一个 Windows 桌面工具（GUI + CLI/TUI），用于在本机多个 Coding Agent 之间**共享 AgentMemory 记忆与工作区交接（handoff）上下文**。

简单说：它在本地跑一个 AgentMemory 服务，并把这个服务以 MCP 的方式接入你装的各种 AI 编码工具（Codex、TRAE、Qoder CN、Claude、Gemini 等），让它们读写**同一份长期记忆**，从而在不同工具/不同账号之间不丢失项目上下文。

仓库地址：

```text
https://github.com/zhibuyu/CrossAgnetCoding
```

## 它解决什么问题

你可能同时用好几个 AI 编码工具。每个工具的对话记忆是各自独立的，换工具、换账号、关掉重开，之前讨论的目标、决策、约定就丢了。

CrossAgnetCoding 的做法：

- 本地启动 [rohitg00/agentmemory](https://github.com/rohitg00/agentmemory) 服务（`http://localhost:3111`）作为统一的"长期记忆"。
- 一键把这个记忆服务以 MCP 形式写进各个工具的配置里，让它们都连到同一个记忆。
- 提供"共享 Prompt"和"工作区桥接"，把记忆的用法和最近的交接摘要喂给每个工具。

多工具统一管理的思路参考了 [farion1231/cc-switch](https://github.com/farion1231/cc-switch/)。

## 支持的 Coding Agent

会自动检测安装状态，并可一键写入用户级 MCP 配置：

| 工具 | 配置文件位置 |
| --- | --- |
| Codex | `%USERPROFILE%\.codex\config.toml` |
| TRAE SOLO CN | `%APPDATA%\TRAE SOLO CN\User\mcp.json` |
| TRAE SOLO | `%APPDATA%\TRAE SOLO\User\mcp.json` |
| **Qoder CN** | `%APPDATA%\QoderCN\SharedClientCache\mcp.json` |
| Claude Code | `%USERPROFILE%\.claude\mcp.json` |
| Claude Desktop | `%APPDATA%\Claude\claude_desktop_config.json` |
| Gemini CLI | `%USERPROFILE%\.gemini\settings.json` |
| OpenCode | `%USERPROFILE%\.config\opencode\opencode.json` |
| OpenClaw | `%USERPROFILE%\.openclaw\openclaw.json` |
| Hermes Agent | `%USERPROFILE%\.hermes\config.yaml` |

> **关于 Qoder CN 的说明**：Qoder CN 是 VS Code 系的 AI IDE，其 MCP 配置位于 `%APPDATA%\QoderCN\SharedClientCache\mcp.json`（已对照实际安装版本确认），采用与 TRAE SOLO 相同的 `mcpServers` JSON 结构。

每次写入配置前都会先备份原文件（生成 `*.bak-时间戳`）。

## 界面说明（各按钮做什么）

### 顶部状态区

- **安装全部**：检查并安装运行所需的 Node.js、AgentMemory、iii-engine。
- **启动服务 / 停止服务**（同一个按钮，切换）：启动或停止本地 AgentMemory 服务。服务未运行时显示蓝色"启动服务"，运行中显示红色"停止服务"。状态会自动刷新。
- **复制 MCP 配置**：把 AgentMemory 的 MCP JSON 复制到剪贴板，便于手动粘贴到工具里。
- **复制 CLI 命令**：复制各工具手动接入的命令/路径提示到剪贴板。
- **同步共享 Prompt**：见下方"功能详解"。
- **桥接工作区**：见下方"功能详解"。
- **迁移数据目录**：把整个数据目录复制到新位置，并把程序未来的读写指向新目录（旧目录保留）。

界面中间会显示**当前数据目录**和**服务日志路径**，方便你在迁移前确认数据到底存在哪里、日志去哪里看。

### 本地环境检查区

- **刷新**：重新扫描各工具的安装/配置状态。
- **全部配置**：一键把 AgentMemory MCP 写入上表所有工具的用户级配置。
- 每张卡片显示：工具名、平台、是否**已安装**（绿色）/**未安装**（灰色）、是否已检测到 MCP 配置路径，以及单独的**配置 / 重新配置**按钮。

## 功能详解

### 同步共享 Prompt

在数据目录下的 `shared\` 文件夹生成一组共享上下文文件（`AGENTS.md`、`CLAUDE.md`、`OPENCODE.md`、`TRAE.md`、`GEMINI.md`、`OPENCLAW.md`、`HERMES.md`）。

每个文件内容相同，告诉对应的 Agent：

- AgentMemory 的 MCP 入口是 `http://localhost:3111`；
- 推荐工作流：**任务开始**时先在 AgentMemory 里检索项目目标/决策/约束 →**进行中**把决策、文件路径、测试命令、交接备注写进记忆 →**任务结束**时总结并写一条简洁的记忆。

作用：让每个工具用**统一的方式**使用这份共享记忆。它只在数据目录里生成参考文件，不会自动塞进你的项目目录。

### 桥接工作区

选择一个项目目录后，CrossAgnetCoding 会：

1. 为该项目计算一个稳定的**工作区 ID**＝`SHA256(规范化路径 + Git 远程地址)` 的前 16 位。因为绑定的是项目路径（和 Git 远程），所以**即使你退出 Codex 换个账号登录，打开同一个项目仍能找回这份工作区记忆**。
2. 从 Codex 的会话目录（`~/.codex/sessions`）和 TRAE 的日志目录（`%APPDATA%\TRAE SOLO[ CN]\logs`）中，导入**有限长度**的可读片段（最多 12 个文件、每个截断到约 3000 字节）。
3. 把这些内容写进工作区记忆 `数据目录\workspaces\<工作区ID>\`：
   - `sessions.jsonl`：追加式的桥接记录；
   - `handoff.md`：最近一次跨工具交接摘要；
   - `workspace.json`：工作区元数据；
   - 以及一组生成的 prompt 文件（含共享 Prompt + 工作区信息 + 最新交接）。

作用：生成一份**绑定到项目、可被各工具读取的交接上下文**，让上下文在不同工具/账号之间延续。

> 注意：桥接**不会**强制让 Codex 和 TRAE 去读对方的原生聊天数据库，而是把有界、可读的片段导入到 CrossAgnetCoding 的工作区记忆里。

### 迁移数据目录

把当前数据目录整体复制到你选择的新目录，并更新 `settings.json` 里的 `dataHome` 指向，之后程序读写都用新目录。旧目录保留不动。

## 数据目录

默认位置：

```text
%USERPROFILE%\.CrossAgnetCoding
```

存放：工作区记忆、共享 Prompt 文件、配置备份、设置、桥接日志。服务运行日志单独位于：

```text
%USERPROFILE%\.agentmemory\agentmemory-service.log
```

这两个路径都会显示在主界面上。可用"迁移数据目录"或 `config migrate` 切换数据目录。

## 使用方法（GUI Usage）

1. 运行：

```text
CrossAgnetCoding.exe
```

2. 若缺少 Node.js / AgentMemory / iii-engine，点击**安装全部**。
3. 点击**启动服务**，确认状态变为 `运行中 (localhost:3111)`。
4. 在"本地环境检查"区点击**全部配置**写入各工具的 MCP 配置；或在单张卡片上点**配置 / 重新配置**只配某一个工具。
5. 需要时点**同步共享 Prompt**、**桥接工作区**。
6. 配置变更后，按需重启 Codex、TRAE、Qoder CN、Claude、Gemini、OpenClaw、Hermes 等工具，使其加载新的 MCP 配置。

## 命令行 / TUI（CLI Usage）

直接运行源码脚本即可使用命令行/文本界面：

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\src\AgentMemoryManager.ps1 -Cli env tools
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\src\AgentMemoryManager.ps1 -Cli agents scan
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\src\AgentMemoryManager.ps1 -Cli agents configure
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\src\AgentMemoryManager.ps1 -Cli workspace init "D:\path\to\project"
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\src\AgentMemoryManager.ps1 -Cli workspace bridge "D:\path\to\project"
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\src\AgentMemoryManager.ps1 -Cli config home
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\src\AgentMemoryManager.ps1 -Cli config migrate "D:\CrossAgnetCodingData"
```

TUI 模式：

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\src\AgentMemoryManager.ps1 -Tui
```

## 直接运行源码（开发测试用）

不打包也能测试，行为与打包后的 exe 一致：

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\src\AgentMemoryManager.ps1
```

## 构建（Build）

在项目目录下执行：

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\scripts\build.ps1
```

产物（用系统自带 IExpress 打包）：

```text
release\CrossAgnetCoding.exe
```

## 测试（Test）

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\tests\selftest.ps1
```

自检使用临时用户目录做配置写入测试，**不会**改动你真实的 Codex/TRAE/Qoder/OpenCode/Claude 配置。

## 项目结构

- `src/AgentMemoryManager.ps1` — 主程序（WinForms 界面 + 全部管理逻辑）。
- `src/launch.vbs` — 打包后 exe 使用的隐藏启动器（避免弹黑色 cmd 窗口）。
- `scripts/build.ps1` — 用 IExpress 构建 `CrossAgnetCoding.exe`。
- `tests/selftest.ps1` — 非 UI 自检与配置写入测试。
- `docs/FUNCTIONS.md` — 面向贡献者的功能与函数说明。
- `trae-mcp-config.json` — 独立的 TRAE 兼容 MCP 配置片段。

## 说明

CrossAgnetCoding 不会自动复制每一条聊天记录。它做的是：给多个 Coding Agent 一个**共享的 AgentMemory MCP 端点**和**共享 Prompt**，让它们能写入、读取持久的任务上下文。

本 MVP 暂未包含：完整的供应商切换、代理路由、用量/成本看板、WebDAV/云同步。
