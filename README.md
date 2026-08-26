# TakoCode MCP Server

[TakoCode](https://takocode.com) is an AI app-building platform. This repository documents the **official TakoCode remote MCP server**, which lets MCP clients (Claude, and any other MCP-capable assistant) build, preview and publish full-stack web apps by directing TakoCode's hosted coding agent.

> **This is a hosted (remote) server.** There is nothing to install and no code in this repository to run — the server runs on TakoCode's infrastructure. This repo holds the public listing metadata (`server.json`) and documentation only.

## Connect

|              |                                                                                              |
| ------------ | -------------------------------------------------------------------------------------------- |
| **Endpoint** | `https://takocode.com/api/mcp`                                                                |
| **Transport**| Streamable HTTP (MCP protocol `2025-06-18`, `2025-03-26`, `2024-11-05`)                       |
| **Auth**     | OAuth 2.0 with dynamic client registration (RFC 7591) — sign in with your TakoCode account    |
| **Account**  | [takocode.com](https://takocode.com) — $10/mo membership + prepaid wallet credits             |

**Claude:** Settings → Connectors → *Add custom connector* → paste `https://takocode.com/api/mcp` → sign in when prompted.

## What your assistant can do

| Tool                | Kind      | Description                                                                    |
| ------------------- | --------- | ------------------------------------------------------------------------------ |
| `takocode_get_started` | read-only | Account status + the onboarding interview to run with the user             |
| `set_preferences`   |           | Save autonomy / question / tier / budget preferences (enforced server-side)    |
| `get_wallet`        | read-only | Wallet balance, membership, month spend, live turn slots                       |
| `top_up_wallet`     |           | Stripe checkout link for the human user ($5–$500 credits)                      |
| `list_projects`     | read-only | Projects with publish status, live URL and spend                               |
| `create_project`    |           | Create a new app project                                                       |
| `start_turn`        | **paid**  | Send one build goal to TakoCode's coding agent (async; billed from wallet)     |
| `check_turn`        | read-only | Poll build progress; final message, cost, files, optional screenshot           |
| `get_preview_link`  | read-only | Shareable ~4h link to the running dev app                                      |
| `get_download_link` | read-only | Signed 1h ZIP download of the project source                                   |
| `export_github`     |           | Push project source to the user's connected GitHub                             |
| `publish_site`      | **paid**  | Deploy to a permanent public URL (explicit user approval required)             |
| `check_publish`     | read-only | Deployment status + live URL                                                   |
| `search_domains`    | read-only | Domain availability + yearly pricing                                           |
| `buy_domain`        |           | Stripe checkout link to register a domain (auto-attached after payment)        |
| `attach_domain`     |           | Connect a domain the user already owns (returns DNS records)                   |

## How it works

- **Your assistant orchestrates; TakoCode builds.** The billable unit is an agent *turn*: one clear goal in, working code out. Long-running work is start/poll (`start_turn` → `check_turn`), so it survives client timeouts and disconnects.
- **Money is never autonomous.** The assistant can never charge anyone: wallet top-ups and domain purchases return Stripe checkout links that only the human can complete, and publishing requires explicit user approval (plus per-user autonomy preferences and an optional monthly budget cap, enforced server-side).
- **Every tool carries MCP annotations** (`readOnlyHint` / `destructiveHint`) so clients can gate confirmations appropriately.

## Security & privacy

- The server is closed-source and runs entirely on TakoCode's infrastructure; connecting a client grants it scoped access to *your* TakoCode account only, via OAuth.
- Tokens are bearer-scoped to your account; revoke access any time from your TakoCode settings.
- [Privacy policy](https://takocode.com/privacy) · [Terms](https://takocode.com/terms) · [Support](https://takocode.com/support)
- Vulnerability reports: see [SECURITY.md](SECURITY.md).

## License

The contents of this repository (documentation and listing metadata) are MIT-licensed. The TakoCode platform and MCP server implementation are proprietary and are **not** part of this repository.
