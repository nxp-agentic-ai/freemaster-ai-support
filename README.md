# FreeMASTER AI Support

![python-badge](https://img.shields.io/badge/Python-3.11--3.14-green)
![fastmcp-badge](https://img.shields.io/badge/FastMCP-4.x-orange?logo=fastmcp)

An MCP server, skills and configuration that give AI agents the capabilities to work efficiently with NXP FreeMASTER Lite.

The server connects to FreeMASTER Lite through its WebSocket JSON-RPC API and can be consumed by any MCP-compatible agent. When connected to assistants such as Claude, GitHub Copilot, VS Code Copilot, Cursor, Goose, or Codex, developers can use natural language to start FreeMASTER Lite, open the communication with a target board, load ELF or TSA symbols, define, read and write variables, integrate the FreeMASTER Driver into an embedded project, or build a web application on top of FreeMASTER Lite.

> **Note** This repository holds the FreeMASTER MCP server, its skills and its standalone configuration. The shared core library `nxp-mcp-shared` is released separately from [nxp-agentic-ai/mcp-core](https://github.com/nxp-agentic-ai/mcp-core) and installed from its GitHub release. The `nxp-mcp-gateway` aggregator and the `nxp-mcp-knowledge` server come from the same release and are optional.

## Features

- Single MCP server for FreeMASTER Lite over its WebSocket JSON-RPC API
- Search/execute model - two MCP tools (`search_actions`, `execute_action`) front a catalog of FreeMASTER actions, keeping the tool list short and token-efficient
- Session handling - one stable session per FreeMASTER Lite endpoint, transparent reconnect, `session_id` optional when a single session is open
- Skills system - markdown skill files served as MCP resources for agent guidance
- Standalone mode - the server runs on its own with its own config file
- Optional gateway composition via mcp-core (`nxp.mcp.servers` entry-point, namespace-prefixed tools)
- Optional vectorized knowledge base (LanceDB + local `bge-base-en-v1.5` embeddings) for offline semantic search over FreeMASTER documentation - no cloud calls, no data leaving the machine

## Table of Contents

- [FreeMASTER AI Support](#freemaster-ai-support)
  - [Features](#features)
  - [Table of Contents](#table-of-contents)
  - [Prerequisites](#prerequisites)
  - [How to Install](#how-to-install)
    - [Step 1 - Install FreeMASTER Lite](#step-1---install-freemaster-lite)
    - [Step 2 - Install the MCP server](#step-2---install-the-mcp-server)
    - [Step 3 - Register the MCP server with your AI coding agent](#step-3---register-the-mcp-server-with-your-ai-coding-agent)
    - [Step 4 - Register the skills with your AI coding agent](#step-4---register-the-skills-with-your-ai-coding-agent)
    - [Step 5 - Verify the installation (Optional)](#step-5---verify-the-installation-optional)
  - [Repository Layout](#repository-layout)
  - [Available Actions](#available-actions)
  - [Registering the MCP server into AI Agents](#registering-the-mcp-server-into-ai-agents)
    - [Claude Desktop / Cursor](#claude-desktop--cursor)
    - [VS Code](#vs-code)
    - [Linux / macOS](#linux--macos)
  - [Skills](#skills)
    - [How skills are served](#how-skills-are-served)
    - [Better experience with agents that natively support skills](#better-experience-with-agents-that-natively-support-skills)
  - [Configuration](#configuration)
    - [Logging](#logging)
    - [Running the Standalone Server](#running-the-standalone-server)
    - [Running through the Gateway (Optional)](#running-through-the-gateway-optional)
  - [Knowledge Base](#knowledge-base)
  - [Building from Source](#building-from-source)
  - [Troubleshooting](#troubleshooting)
  - [License](#license)
  - [Community](#community)

---

## Prerequisites

| Prerequisite | Version | Link |
|--------------|---------| ---- |
| Python | 3.11 - 3.14 | [Download](https://www.python.org/downloads/) |
| FreeMASTER Lite | 1.5.0+ | [Download](https://www.nxp.com/design/design-center/software/development-software/freemaster-run-time-debugging-tool:FREEMASTER) |
| Git | latest | [Download](https://git-scm.com/downloads) |
| AI Agent (MCP compatible) | latest | Any MCP-compatible agent that can launch a standalone MCP server - such as [Claude Desktop](https://claude.ai/download), [VS Code](https://code.visualstudio.com/) or [Cursor](https://www.cursor.com/), among others |

> **Note** FreeMASTER Lite provides the JSON-RPC API (over WebSocket) that the MCP server connects to. FreeMASTER Lite must be running before the agent uses the FreeMASTER tools. The `freemaster-lite-start-server` skill describes how the agent can start it for you.

> **Note** Please consider changing the default port of the FreeMASTER Lite API if it is not available on your system, and align `ws_port` in the server configuration accordingly.

| Product | Default MCP API Port |
|--------------|---------|
| FreeMASTER Lite | 8090 TCP |

---

## How to Install

### Step 1 - Install FreeMASTER Lite

Install FreeMASTER Lite (see [Prerequisites](#prerequisites)). Note the port its JSON-RPC API listens on if you change the default (8090).

### Step 2 - Install the MCP server

Clone the repository and install the server into a virtual environment:

```shell
git clone https://github.com/nxp-agentic-ai/freemaster-ai-support.git
cd freemaster-ai-support

python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # Linux / macOS

pip install -r requirements.txt
```

`requirements.txt` installs `nxp-mcp-shared` from the [mcp-core](https://github.com/nxp-agentic-ai/mcp-core) GitHub release (tag `v2.0.0`) and the FreeMASTER server (`packages/mcp_freemaster`) in editable mode.

> **Note** If the mcp-core release is not publicly accessible for your account, pip cannot download the `nxp-mcp-shared` wheel. Store your GitHub credentials in `~/.netrc` (`%USERPROFILE%\_netrc` on Windows) for `github.com` and `api.github.com`; never embed a token in the URL inside `requirements.txt`.

### Step 3 - Register the MCP server with your AI coding agent

Add an MCP server entry to your agent that launches `run-mcp.bat` (Windows) or `run-mcp.sh` (Linux/macOS) with the standalone config. See [Registering the MCP server into AI Agents](#registering-the-mcp-server-into-ai-agents) for the per-agent JSON.

### Step 4 - Register the skills with your AI coding agent

Skills are always served as MCP resources, but agents with native skill support work best when the skill files are also on disk. Copy the folders under `skills/freemaster/` into your agent's skills folder. See [Skills](#skills) for per-agent skill folders and copy commands.

### Step 5 - Verify the installation (Optional)

Start the server once from the repository root and confirm it comes up without errors:

```shell
run-mcp.bat nxp.mcp.freemaster configs/freemaster.standalone.yaml
```

The server talks MCP over stdio, so it waits silently for a client; stop it with `Ctrl+C`. Startup details are written to `.tmp/logs/freemaster.log`.

Then restart your AI agent and ask it to list the available tools - `search_actions` and `execute_action` from the FreeMASTER server should be listed. If something is missing, see [Troubleshooting](#troubleshooting).

---

## Repository Layout

| Path | Description |
|---|---|
| `packages/mcp_freemaster/` | The FreeMASTER MCP server Python package (`nxp-mcp-freemaster`); owns its own `pyproject.toml` and declares the `nxp.mcp.servers` entry-point |
| `configs/` | `freemaster.standalone.yaml` - ready-to-use standalone configuration |
| `skills/freemaster/` | Bundled agent skills for FreeMASTER Lite and the FreeMASTER Driver |
| `scripts/` | Build utilities (`build_modules.py`) |
| `run-mcp.bat` / `run-mcp.sh` | Cross-platform launcher used by agent registrations |
| `requirements.txt` | Development / source install: mcp-core dependency, editable server install, build and test tools |
| `pyproject.toml` | Root meta-package that pulls in the FreeMASTER server |
| `SBOM-FreeMASTER-AI-Support.spdx` | SPDX software bill of materials for the release |
| `LICENSE`, `LICENSE_ADDENDUM` | NXP Online Code Hosting Software License and the AI Addendum |
| `RELEASE_NOTES.md` | Version history and known limitations |

## Available Actions

The server exposes two MCP tools:

- `search_actions` - search the action catalog (BM25 or regex) and return matching actions with their input schemas.
- `execute_action` - run an action by its name with a `params` object that matches its schema.

The catalog contains the following actions. Every `rpc.*` action also accepts an optional `session_id`, which can be omitted when exactly one session is open. `ws.disconnect` needs the `session_id` returned by `ws.connect`.

| Action | Parameters | Description |
|---|---|---|
| `ws.connect` | `host`, `port` | Open a WebSocket connection to FreeMASTER Lite; returns a `session_id` |
| `ws.disconnect` | `session_id` | Close the WebSocket connection |
| `rpc.GetAppVersion` | - | Retrieve the FreeMASTER Lite version |
| `rpc.Exit` | - | Stop the FreeMASTER Lite server |
| `rpc.StartComm` | `name` | Open the communication with the target board, using a connection name from the `.fmcfg` file or a full connection string |
| `rpc.StopComm` | - | Close the communication with the target board |
| `rpc.IsCommPortOpen` | - | Check whether the communication is open |
| `rpc.IsBoardDetected` | - | Check whether FreeMASTER Lite detects the board |
| `rpc.GetDetectedBoardInfo` | - | Retrieve the board information encoded in the FreeMASTER Driver |
| `rpc.ReadELF` | - | Load symbols from the ELF file defined in the `.fmcfg` file |
| `rpc.ReadTSA` | - | Load symbols from the board via Target Side Addressing |
| `rpc.DefineVariable` | `variable` | Define a project variable |
| `rpc.GetVariableInfo` | `name` | Retrieve a project variable definition |
| `rpc.ReadVariable` | `name` | Read a project variable value |
| `rpc.WriteVariable` | `name`, `value` | Write a value into a project variable |

A typical sequence is `ws.connect` -> `rpc.StartComm` -> `rpc.IsBoardDetected` -> `rpc.ReadELF` / `rpc.ReadTSA` -> `rpc.ReadVariable` / `rpc.WriteVariable` -> `rpc.StopComm` -> `ws.disconnect`. The `freemaster-lite-jsonrpc-api` skill documents the full workflow and the error handling.

---

## Registering the MCP server into AI Agents

The FreeMASTER server can be registered in any MCP-compatible agent that can launch a stdio server, for example:

 * [Claude Desktop](https://claude.com/download)
 * [VS Code](https://code.visualstudio.com/)
 * [Cursor](https://cursor.com/download)

Use the absolute path of `run-mcp.bat` (Windows) or `run-mcp.sh` (Linux/macOS) from your clone. The launcher changes into the repository root and activates `.venv`, so the config path can stay relative.

### Claude Desktop / Cursor

Add the entry to `claude_desktop_config.json` (Claude Desktop) or `~/.cursor/mcp.json` (Cursor):

```json
{
  "mcpServers": {
    "nxp-freemaster": {
      "command": "C:/NXP/freemaster-ai-support/run-mcp.bat",
      "args": ["nxp.mcp.freemaster", "configs/freemaster.standalone.yaml"],
      "disabled": false
    }
  }
}
```

### VS Code

Add the entry to `.vscode/mcp.json` in your workspace (or to your user MCP configuration):

```json
{
  "servers": {
    "nxp-freemaster": {
      "type": "stdio",
      "command": "C:/NXP/freemaster-ai-support/run-mcp.bat",
      "args": ["nxp.mcp.freemaster", "configs/freemaster.standalone.yaml"]
    }
  }
}
```

### Linux / macOS

Use `run-mcp.sh` through `bash`:

```json
{
  "mcpServers": {
    "nxp-freemaster": {
      "command": "bash",
      "args": ["/opt/nxp/freemaster-ai-support/run-mcp.sh", "nxp.mcp.freemaster", "configs/freemaster.standalone.yaml"]
    }
  }
}
```

> **Note:** Some agents may need to be restarted after registration.

## Skills

Skills are markdown documents bundled with the MCP server that describe how to interact with FreeMASTER Lite and the FreeMASTER Driver in a structured format agents can reason about.

| Skill | Description |
|---|---|
| `freemaster-driver-integration` | Integrate the FreeMASTER Driver into an embedded application: include/source paths, `freemaster_cfg.h`, transport, `FMSTR_Init()` / `FMSTR_Poll()` |
| `freemaster-lite-command-line-arguments` | Command-line options of `fmlite.exe` / `fmlite.bin` |
| `freemaster-lite-configuration-file` | Create, edit and validate `.fmcfg` configuration files (connections, ELF, variables, mock API) |
| `freemaster-lite-jsonrpc-api` | Drive FreeMASTER Lite over its WebSocket JSON-RPC API |
| `freemaster-lite-start-server` | Locate, launch and verify the FreeMASTER Lite service |
| `freemaster-lite-stop-server` | Stop FreeMASTER Lite gracefully or recover from a stuck instance |
| `freemaster-lite-web-application` | Build browser or Node.js applications with `freemaster-client.js` and the PCM API |

### How skills are served

All skills are **automatically loaded as MCP resources** by the server at startup. Every skill file is exposed as a resource URI (e.g. `skill://nxp_freemaster/freemaster_lite_start_server`) and is accessible to any MCP-compatible agent without any additional setup.

### Better experience with agents that natively support skills

Some agents have native support for skill/prompt directories and will pick up skills automatically when placed in a specific directory. For the best experience with those agents, copy the skills from the `skills/freemaster/` directory to the appropriate location for your agent.

| Agent | Skills folder | Notes |
|---|---|---|
| **Claude Code** | `~/.claude/skills/` | Place skill folders directly inside; Claude Code loads `.md` files from this directory |
| **Codex (OpenAI)** | `~/.codex/skills/` | Copy the skill subdirectories; Codex reads skills from the user profile skills dir |
| **Cursor** | `.cursorrules` / `.cursor/skills/` | Add skill content to `.cursorrules` in the project root, or place skill folders under `.cursor/skills/` |

**Copying skills manually**

```shell
# Windows (Claude Code example)
xcopy /E /I skills\freemaster %USERPROFILE%\.claude\skills

# Linux / macOS (Claude Code example)
mkdir -p ~/.claude/skills && cp -r skills/freemaster/. ~/.claude/skills/
```

## Configuration

This section walks you through configuring the MCP server via its configuration file.

> **Note:** Relative paths in a config file (`logging.file`, `skills`) are resolved against the directory of that config file, not against the current working directory.

### Logging

```yaml
logging:
  level: INFO                         # DEBUG | INFO | WARNING | ERROR
  timestamp_format: "%Y-%m-%d %H:%M:%S"
  format: "%(asctime)s [%(name)s] %(levelname)s %(message)s"
  file: ../.tmp/logs/freemaster.log   # optional - omit to log to stderr only
```

| Field | Default | Description |
|-------|---------|-------------|
| `level` | `INFO` | Log verbosity |
| `timestamp_format` | `%Y-%m-%d %H:%M:%S` | Date/time format in log lines |
| `format` | `%(asctime)s [%(name)s] %(levelname)s %(message)s` | Full Python logging format string |
| `file` | _(none)_ | Path to log file; omit to write to stderr only |

> **Note:** In the gateway, `logging` settings are inherited from the top-level `logging` block unless a per-server `logging` override is provided.

---
### Running the Standalone Server

A standalone config has three sections:

- `logging` - logging settings
- `settings` - server-specific settings
- `skills` - path to the skills directory for this server (optional)

**Start the server:**
```shell
# Run from the repository root
python -m nxp.mcp.freemaster configs/freemaster.standalone.yaml
```
or
```shell
run-mcp.bat nxp.mcp.freemaster configs/freemaster.standalone.yaml
```

**Config file (`configs/freemaster.standalone.yaml`):**
```yaml
logging:
  level: DEBUG
  timestamp_format: "%Y-%m-%d %H:%M:%S"
  format: "%(asctime)s [%(name)s] %(levelname)s %(message)s"
  file: ../.tmp/logs/freemaster.log   # optional - omit for stderr only

settings:
  ws_host: "localhost"                # host where the FreeMASTER Lite JSON-RPC API is running
  ws_port: 8090                       # FreeMASTER Lite JSON-RPC API port (default: 8090)

skills: ../skills/freemaster
```

| Setting | Default | Description |
|---------|---------|-------------|
| `ws_host` | `localhost` | Host of the FreeMASTER Lite JSON-RPC API |
| `ws_port` | `8090` | Port of the FreeMASTER Lite JSON-RPC API; must match the FreeMASTER Lite configuration |

---
### Running through the Gateway (Optional)

The `nxp-mcp-gateway` aggregator from [mcp-core](https://github.com/nxp-agentic-ai/mcp-core) loads several servers and exposes them through one MCP connection. Each tool is prefixed with the server name (e.g. `nxp_freemaster_search_actions`) to avoid naming conflicts. Use it when you want to combine FreeMASTER with the knowledge base or with other NXP MCP servers.

1. Uncomment the `nxp-mcp-gateway` line in `requirements.txt` and run `pip install -r requirements.txt` again.
2. Create a gateway config, for example `configs/gateway.stdio.yaml`:

```yaml
logging:
  level: INFO
  file: ../.tmp/logs/gateway.log      # optional - omit for stderr only

gateway:
  strict_startup: true                # fail to start if any enabled server fails to load

servers:
  freemaster:
    settings:
      ws_host: "localhost"
      ws_port: 8090
    skills: ../skills/freemaster
```

3. Start the gateway (or register this command in your agent instead of the standalone one):

```shell
run-mcp.bat nxp.mcp.gateway configs/gateway.stdio.yaml
```

> **How opt-out works:** Servers installed but not listed in the gateway config are still discovered automatically. To explicitly prevent a server from loading, add `disabled: true` to its entry.

---

## Knowledge Base

The FreeMASTER documentation index is optional and is not part of the source tree. It is published as an asset of each [GitHub release](https://github.com/nxp-agentic-ai/freemaster-ai-support/releases) and served by the `nxp-mcp-knowledge` server from [mcp-core](https://github.com/nxp-agentic-ai/mcp-core):

- **Storage** - LanceDB table `freemaster` under `knowledge_db/bge-base-en-v1.5/`.
- **Embeddings** - `BAAI/bge-base-en-v1.5` (768 dimensions), executed locally on CPU (or CUDA / MPS if configured).
- **Tools** - `kb_search` (semantic search) and `kb_list_corpora`; through the gateway they appear as `nxp_knowledge_kb_search` and `nxp_knowledge_kb_list_corpora`.

To enable it:

1. Download the knowledge base archive from the release assets and extract it in the repository root, so that `knowledge_db/bge-base-en-v1.5/freemaster.lance/` exists.
2. Uncomment the `nxp-mcp-knowledge` (and `nxp-mcp-gateway`) lines in `requirements.txt` and run `pip install -r requirements.txt` again.
3. Add the knowledge server to the gateway config:

```yaml
servers:
  knowledge:
    settings:
      db_path: "../knowledge_db/bge-base-en-v1.5"   # extracted release asset
      corpora:
        - name: "freemaster"                        # LanceDB table name
      chunk_size: 1200
      chunk_overlap: 200
      embedder:
        provider: "sentence_transformers"
        model: "BAAI/bge-base-en-v1.5"
        dimension: 768
        device: "cpu"
        cache: "../models"                          # directory (config-relative) where the model is cached
        offline: true                               # set to false once to download the model into the cache
      mutable: false
```

Everything runs offline once the model is cached - no documentation, source code or query text is sent to any external service by the knowledge base.

## Building from Source

The FreeMASTER server under `packages/mcp_freemaster/` owns its own `pyproject.toml` and declares an `nxp.mcp.servers` entry-point, so it can be installed and used independently. The root `pyproject.toml` is a convenience meta-package that pulls it in.

`nxp-mcp-shared` is not built here. It is released from [nxp-agentic-ai/mcp-core](https://github.com/nxp-agentic-ai/mcp-core) and installed from the release referenced in `requirements.txt` (currently `v2.0.0`). The FreeMASTER package requires `nxp-mcp-shared>=2.0.0.dev0,<3`, so the mcp-core tag must stay inside that range.

```shell
# 1. Create and activate a virtual environment (Python 3.11+)
python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # Linux / macOS

# 2. Install dependencies (nxp-mcp-shared from mcp-core v2.0.0 + the server in editable mode)
pip install -r requirements.txt

# 3. Build the wheels for the FreeMASTER server and the root meta-package
python scripts/build_modules.py
```

The wheels are written to the `dist/` directory. Useful options of `build_modules.py`: `--list` (show discovered packages), `--clean` (remove `dist/` first), `--no-root` (skip the root meta-package), `--sdist` (also build source distributions).

To run the server directly from a source checkout, complete step 2 and point the module at its standalone config:

```shell
python -m nxp.mcp.freemaster configs/freemaster.standalone.yaml
```

## Troubleshooting

| Symptom | Likely cause and fix |
|---|---|
| Agent shows no FreeMASTER tools | The MCP server entry was not picked up. Check that `command` holds the absolute path of `run-mcp.bat` / `run-mcp.sh`, then restart the agent. Standalone, the tools are named `search_actions` and `execute_action`; through the gateway they carry the `nxp_freemaster_` prefix. |
| `pip install -r requirements.txt` fails with 401 / 404 on the mcp-core wheel | The mcp-core release is not accessible with your credentials. Configure `~/.netrc` (`%USERPROFILE%\_netrc` on Windows) for `github.com` and `api.github.com`, then re-run the install. |
| `No module named 'nxp.mcp.shared'` | `nxp-mcp-shared` was not installed, or the agent launches a different Python. Re-run `pip install -r requirements.txt` inside `.venv` and register `run-mcp.bat` / `run-mcp.sh`, which activates `.venv`. |
| `No module named nxp.mcp.gateway` (or `nxp.mcp.knowledge`) | The gateway and the knowledge server are optional and released from [mcp-core](https://github.com/nxp-agentic-ai/mcp-core). Uncomment them in `requirements.txt` and re-install, or start the server standalone: `run-mcp.bat nxp.mcp.freemaster configs/freemaster.standalone.yaml`. |
| Gateway exits immediately at startup | With `gateway.strict_startup: true`, one failing server aborts the whole gateway. Check the log file configured under `logging.file`, then either fix the server's `settings` or mark it `disabled: true`. |
| FreeMASTER tools cannot connect | FreeMASTER Lite is not running or listens on another port. Align `ws_host` / `ws_port` with the `.fmcfg` configuration (default 8090). |
| `No active session; call connect first.` | Run the `ws.connect` action before any `rpc.*` action. |
| `session_id required: multiple active sessions` | More than one FreeMASTER Lite endpoint is connected. Pass the `session_id` returned by `ws.connect`. |
| Knowledge search returns nothing | The knowledge base asset was not extracted, `db_path` points to the wrong folder, or the corpus name does not match. Confirm that `knowledge_db/bge-base-en-v1.5/freemaster.lance/` exists and that the corpus name matches the `kb_list_corpora` output. |
| Skills are not offered by the agent | Skills are always exposed as MCP resources, but agents with native skill support need the files on disk. Copy the folders under `skills/freemaster/` into the agent's skills folder (see [Skills](#skills)). |

Enable `logging.level: DEBUG` and set `logging.file` in the relevant config to capture a full trace before reporting an issue.

## License

This software is licensed under the **LA_OPT_Online Code Hosting NXP_Software_License - v1.4 May 2025**, together with an **AI Addendum** that governs the agentic / AI-assisted functionality of this toolkit.

- Full license text: [`LICENSE`](LICENSE)
- AI Addendum: [`LICENSE_ADDENDUM`](LICENSE_ADDENDUM)
- Third-party components and their licenses: [`SBOM-FreeMASTER-AI-Support.spdx`](SBOM-FreeMASTER-AI-Support.spdx)

By downloading, installing or using this software you accept the terms of the license agreement and the addendum. If a separate license agreement for this software has been signed by you and NXP, that agreement governs your use and supersedes the terms above.

## Community

Questions, feedback, feature requests and discussions about this toolkit are welcome on the NXP Community:

[Agentic AI Development - NXP Community](https://community.nxp.com/t5/Agentic-AI-Development/tkb-p/agentic-ai-development)

When reporting a problem, please include the toolkit version (see [`RELEASE_NOTES.md`](RELEASE_NOTES.md)), your operating system and Python version, the FreeMASTER Lite version, the agent you are using, and the relevant portion of the server log captured with `logging.level: DEBUG`.
