# CrossAgnetCoding

Version: 0.3.0-mvp

CrossAgnetCoding is a Windows GUI + CLI/TUI MVP for sharing AgentMemory and workspace handoff context across local Coding Agents. Codex and TRAE SOLO/CN are first-class targets, with additional MCP setup support for Claude Code, Claude Desktop, Gemini CLI, OpenCode, OpenClaw, and Hermes Agent.

Repository target:

```text
https://github.com/zhibuyu/CrossAgnetCoding
```

## Origin And Inspiration

CrossAgnetCoding is built around [rohitg00/agentmemory](https://github.com/rohitg00/agentmemory). AgentMemory provides the shared memory service and MCP server used by this manager.

The multi-agent setup workflow is inspired by [farion1231/cc-switch](https://github.com/farion1231/cc-switch/), especially the idea of managing multiple Coding Agent clients from one place. CrossAgnetCoding currently incorporates the practical pieces that fit this project:

- unified MCP configuration across Coding Agents
- user-level config backup before writes
- shared prompt/context files for different agents
- project-bound workspace memory that survives Codex account changes
- Codex/TRAE session bridge summaries into shared handoff files
- copyable CLI snippets for manual setup
- quick scan of installed/configured agent clients

## Features

- Installs/checks Node.js, AgentMemory, and iii-engine.
- Starts/stops the local AgentMemory service on `http://localhost:3111`.
- Detects and configures:
  - Codex
  - TRAE SOLO CN
  - TRAE SOLO
  - Claude Code
  - Claude Desktop
  - Gemini CLI
  - OpenCode
  - OpenClaw
  - Hermes Agent
- Provides MCP JSON and CLI command snippets.
- Generates shared prompt files under the CrossAgnetCoding data home.
- Supports configurable data directory migration.
- Provides CLI and TUI modes.
- Uses an About-style local environment page with per-tool AI Code cards instead of a single generic agent scan.
- Chinese/English UI switch.
- No black `cmd` window when launched from the packaged exe.

## MVP Notes

The session bridge does not force Codex and TRAE to read each other's native chat databases. Instead, it imports bounded, readable session/log snippets into CrossAgnetCoding workspace memory:

- `sessions.jsonl` stores bridge entries.
- `handoff.md` stores the latest cross-tool summary.
- generated prompt files expose the handoff to supported tools.

Workspace memory is tied to the project directory, not to the Codex account. If you log out of Codex and sign in with another account, loading the same project directory can still recover CrossAgnetCoding workspace memory.

Deferred from MVP: full provider switching, proxy routing, usage/cost dashboards, and WebDAV/cloud sync.

## GUI Usage

1. Run:

```text
CrossAgnetCoding.exe
```

2. Click `安装全部 / Install All` if Node.js, AgentMemory, or iii-engine is missing.

3. Click `启动服务 / Start Service`.

4. Confirm the service status becomes:

```text
运行中 (localhost:3111)
```

5. In `Coding Agent 接入 / Coding Agent Access`, click:

- `扫描 Agent / Scan Agents` to see installed/configured status.
- `一键配置 MCP / Configure MCP` to write user-level MCP config.
- `复制 CLI 命令 / Copy CLI Commands` for manual setup.
- `同步共享 Prompt / Sync Shared Prompt` to create shared context files.

- `Bridge Workspace` to initialize project memory and import Codex/TRAE handoff snippets.
- `Migrate Data Home` to copy CrossAgnetCoding data to a new directory and switch future reads/writes.

The main screen now mirrors the cc-switch About page structure: a product/version card, service controls, and a `Local Environment Check` grid. Each AI Code tool card shows installed status, current version when it can be read safely, latest-version placeholder, CrossAgnetCoding MCP connection status, and a per-tool configure button.

6. Restart Codex, TRAE SOLO CN, OpenCode, Claude, Gemini, OpenClaw, or Hermes after MCP config changes when required.

## CLI Usage

Run the source script directly for CLI/TUI workflows:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\src\AgentMemoryManager.ps1 -Cli env tools
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\src\AgentMemoryManager.ps1 -Cli agents scan
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\src\AgentMemoryManager.ps1 -Cli agents configure
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\src\AgentMemoryManager.ps1 -Cli workspace init "D:\path\to\project"
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\src\AgentMemoryManager.ps1 -Cli workspace bridge "D:\path\to\project"
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\src\AgentMemoryManager.ps1 -Cli config home
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\src\AgentMemoryManager.ps1 -Cli config migrate "D:\CrossAgnetCodingData"
```

TUI mode:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\src\AgentMemoryManager.ps1 -Tui
```

## Data Directory

Default:

```text
%USERPROFILE%\.CrossAgnetCoding
```

The data directory stores workspace memory, prompt files, backups, settings, and bridge logs. Use `config migrate` to copy data to a new directory and switch the app pointer. The old directory is preserved.

## Project Structure

- `src/AgentMemoryManager.ps1` - main WinForms application and manager logic.
- `src/launch.vbs` - hidden launcher used by the packaged exe.
- `scripts/build.ps1` - builds `CrossAgnetCoding.exe` with IExpress.
- `tests/selftest.ps1` - non-UI self-tests and config writer tests.
- `docs/FUNCTIONS.md` - feature and function guide for contributors.
- `trae-mcp-config.json` - standalone TRAE-compatible MCP config snippet.

## Build

Run from the project folder:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\scripts\build.ps1
```

The output file is:

```text
release\CrossAgnetCoding.exe
```

## Test

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\tests\selftest.ps1
```

The test uses temporary user directories for config writer checks, so it does not modify your real Codex/TRAE/OpenCode/Claude configuration.

## Notes

CrossAgnetCoding does not automatically copy every chat transcript. It gives multiple Coding Agents a shared AgentMemory MCP endpoint and shared prompts so they can write and read durable task context.
