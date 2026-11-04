## Overview

FreeMASTER AI Support gives AI agents the capabilities to work efficiently with NXP FreeMASTER Lite: an MCP server that drives FreeMASTER Lite over its WebSocket JSON-RPC API, agent skills for FreeMASTER Lite and the FreeMASTER Driver, and a ready-to-use standalone configuration.

---

## v2.0.0 (05.11.2026)

### Changed

- FreeMASTER AI Support is now released from its own repository, [nxp-agentic-ai/freemaster-ai-support](https://github.com/nxp-agentic-ai/freemaster-ai-support). In v1.0.0 the FreeMASTER MCP server was released as part of S32 Design Studio AI Support.
- The core infrastructure - `nxp-mcp-shared`, the optional `nxp-mcp-gateway` aggregator and the optional `nxp-mcp-knowledge` server - is released separately from [nxp-agentic-ai/mcp-core](https://github.com/nxp-agentic-ai/mcp-core). `requirements.txt` installs `nxp-mcp-shared` from the mcp-core `v2.0.0` release.
- FastMCP updated to 4.1.0.
- `nxp-mcp-freemaster` depends on `nxp-mcp-shared>=2.0.0.dev0,<3` and `websockets>=16.0,<17`, using version ranges instead of exact pins. Supported Python versions: 3.11 - 3.14.
- The FreeMASTER knowledge base is published as a release asset instead of being bundled with the installer.

---

## v1.0.0 (28.09.2026)

Initial release of the FreeMASTER MCP server, as part of S32 Design Studio AI Support v1.0.0.

### Key Features

- MCP server for FreeMASTER Lite over its WebSocket JSON-RPC API
- Search/execute action model (`search_actions`, `execute_action`) for low token consumption
- Skills for FreeMASTER Lite (start/stop, command line, `.fmcfg` configuration, JSON-RPC API, web applications) and for FreeMASTER Driver integration, served as MCP resources
- Standalone mode and gateway composition
- Knowledge base corpus for offline semantic search over FreeMASTER documentation

---

## Known Limitations

- No self-installing package yet: install from source with `requirements.txt`.
- Knowledge base search and gateway mode require the optional mcp-core packages (`nxp-mcp-knowledge`, `nxp-mcp-gateway`) and, for search, the knowledge base release asset.

## Licensing

This release is distributed under the **LA_OPT_Online Code Hosting NXP_Software_License - v1.4 May 2025**, together with the **AI Addendum**.
