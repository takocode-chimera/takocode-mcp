# TakoCode MCP Server

[TakoCode](https://takocode.com) is an AI app-building platform. This repository documents the **official TakoCode remote MCP server**, which lets MCP clients (Claude, ChatGPT, and any other MCP-capable assistant) build, preview and publish full-stack web apps by directing TakoCode's hosted coding agent.

> **This is a hosted (remote) server.** There is nothing to install and no code in this repository to run — the server runs on TakoCode's infrastructure. This repo holds the public listing metadata (`server.json`, the ChatGPT plugin package under `chatgpt-plugin/`) and documentation only.

## Connect

|               |                                                                                                   |
| ------------- | ------------------------------------------------------------------------------------------------- |
| **Endpoint**  | `https://takocode.com/api/mcp`                                                                     |
| **ChatGPT directory endpoint** | `https://takocode.com/api/mcp/chatgpt` — same account and tools, minus in-chat purchases (see below) |
| **Transport** | Streamable HTTP (MCP protocol `2026-07-28`, `2025-11-25`, `2025-06-18`, `2025-03-26`, `2024-11-05`) |
| **Auth**      | OAuth 2.0 (RFC 8414 / 9728 discovery, PKCE, RFC 7591 dynamic client registration, RFC 8707 `resource`, RFC 9207 `iss`) — sign in with your TakoCode account |
| **Account**   | [takocode.com](https://takocode.com) — $10/mo membership + prepaid wallet credits                  |

**Claude (web & desktop):** Settings → Connectors → *Add custom connector* → paste `https://takocode.com/api/mcp` → sign in when prompted.

**Claude Code:** `claude mcp add --transport http takocode https://takocode.com/api/mcp`

**ChatGPT (custom MCP server):** chatgpt.com/plugins → **+** → *Add custom MCP server* → name `TakoCode`, URL `https://takocode.com/api/mcp`, authentication **OAuth** → *Create as a plugin* → sign in when prompted. Available in workspaces that allow custom MCP servers (Business, Enterprise, Edu — web). Then type `@TakoCode` in a chat.

**ChatGPT (directory plugin):** the submission package lives in [`chatgpt-plugin/`](chatgpt-plugin/) (`plugin.json`, `mcp.json`, onboarding skill, assets). It points at `https://takocode.com/api/mcp/chatgpt`.

Full guide: [takocode.com/connect](https://takocode.com/connect).

## What your assistant can do

| Tool                   | Kind      | Surfaces        | Description                                                                 |
| ---------------------- | --------- | --------------- | --------------------------------------------------------------------------- |
| `takocode_get_started` | read-only | all             | Account status + the onboarding interview to run with the user              |
| `get_profile`          | read-only | all             | The connected account's id, name and email (ChatGPT `openai/profile`)       |
| `set_preferences`      |           | all             | Save autonomy / question / tier / budget preferences (enforced server-side) |
| `get_wallet`           | read-only | all             | Wallet balance, membership, month spend, live turn slots                    |
| `top_up_wallet`        |           | `/api/mcp` only | Stripe checkout link for the human user ($5–$500 credits)                   |
| `list_projects`        | read-only | all             | Projects with publish status, live URL and spend                            |
| `create_project`       |           | all             | Create a new app project                                                    |
| `start_turn`           | **paid**  | all             | Send one build goal to TakoCode's coding agent (async; billed from wallet)  |
| `check_turn`           | read-only | all             | Poll build progress; final message, cost, files, optional screenshot        |
| `get_preview_link`     | read-only | all             | Shareable ~4h link to the running dev app                                   |
| `get_download_link`    | read-only | all             | Signed 1h ZIP download of the project source                                |
| `export_github`        |           | all             | Push project source to the user's connected GitHub                          |
| `publish_site`         | **paid**  | all             | Deploy to a permanent public URL (explicit user approval required)          |
| `check_publish`        | read-only | all             | Deployment status + live URL                                                |
| `search_domains`       | read-only | all             | Domain availability + yearly pricing                                        |
| `buy_domain`           |           | `/api/mcp` only | Stripe checkout link to register a domain (auto-attached after payment)     |
| `attach_domain`        |           | all             | Connect a domain the user already owns (returns DNS records)                |

## How it works

- **Your assistant orchestrates; TakoCode builds.** The billable unit is an agent *turn*: one clear goal in, working code out. Long-running work is start/poll (`start_turn` → `check_turn`), so it survives client timeouts and disconnects.
- **Money is never autonomous.** The assistant can never charge anyone: wallet top-ups and domain purchases return Stripe checkout links that only the human can complete, and publishing requires explicit user approval (plus per-user autonomy preferences and an optional monthly budget cap, enforced server-side).
- **Two surfaces, one server.** `/api/mcp` is the full tool set. `/api/mcp/chatgpt` is the ChatGPT-directory surface: OpenAI's plugin guidelines do not allow directory plugins to sell credits, subscriptions or domains, promote upgrades, or link to checkout pages, so that surface omits `top_up_wallet` and `buy_domain` and describes entitlement gaps as "not available with the account's current membership or balance" with an informational link to the plans page only. Builds and publishing use the prepaid credits already on the account on both surfaces.
- **Every tool carries MCP annotations** (`readOnlyHint` / `destructiveHint` / `openWorldHint` as explicit booleans, plus `title`) so clients can gate confirmations appropriately.

## Security & privacy

- The server is closed-source and runs entirely on TakoCode's infrastructure; connecting a client grants it scoped access to *your* TakoCode account only, via OAuth.
- Tokens are bearer-scoped to your account; disconnect an assistant any time from **Settings → Connected assistants** (revokes every token it holds) or revoke individual personal access tokens.
- [Privacy policy](https://takocode.com/privacy) · [Terms](https://takocode.com/terms) · [Support](https://takocode.com/support)
- Vulnerability reports: see [SECURITY.md](SECURITY.md).

## License

The contents of this repository (documentation and listing metadata) are MIT-licensed. The TakoCode platform and MCP server implementation are proprietary and are **not** part of this repository.
