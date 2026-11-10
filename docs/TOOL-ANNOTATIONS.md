# TakoCode MCP tool annotations

Every tool on `https://takocode.com/api/mcp` sets all four MCP tool hints as explicit booleans (never omitted/null). The table is generated from the server source , so it cannot drift from what `tools/list` returns.

| Tool | readOnly | destructive | idempotent | openWorld |
|---|---|---|---|---|
| `takocode_get_started` | true | false | true | false |
| `get_profile` | true | false | true | false |
| `set_preferences` | false | false | true | false |
| `get_wallet` | true | false | true | false |
| `top_up_wallet` | false | false | false | true |
| `list_projects` | true | false | true | false |
| `create_project` | false | false | false | false |
| `start_turn` | false | true | false | true |
| `check_turn` | true | false | true | false |
| `revert_turn` | false | true | true | false |
| `query_codebase` | true | false | true | false |
| `security_scan` | true | false | true | true |
| `get_preview_link` | true | false | true | false |
| `get_download_link` | true | false | true | false |
| `export_github` | false | true | false | true |
| `publish_site` | false | true | false | true |
| `check_publish` | true | false | true | false |
| `search_domains` | true | false | true | true |
| `buy_domain` | false | false | false | true |
| `attach_domain` | false | false | true | true |

## Justification per tool

### `takocode_get_started` — Get started with TakoCode
- **readOnlyHint: true** — Reads the caller's preferences, wallet, projects, caps and running turns to compose a guide; writes nothing.
- **destructiveHint: false** — No state is modified.
- **idempotentHint: true** — Same account state → same guide; repeated calls have no additional effect.
- **openWorldHint: false** — Only the TakoCode database is read; no external service is contacted.

### `get_profile` — Connected account identity
- **readOnlyHint: true** — Returns the identity of the connected account from the token and database; writes nothing.
- **destructiveHint: false** — No state is modified.
- **idempotentHint: true** — Repeated calls return the same identity with no side effects.
- **openWorldHint: false** — Internal database lookup only.

### `set_preferences` — Save user automation preferences
- **readOnlyHint: false** — Upserts the caller's own automation preferences row (autonomy, question policy, default tier, budget).
- **destructiveHint: false** — Overwrites only the caller's own preference values; nothing is deleted, no project or file is touched, and any value can be set again — fully reversible.
- **idempotentHint: true** — Calling again with the same values leaves the row unchanged.
- **openWorldHint: false** — Writes one internal database row; no external service.

### `get_wallet` — Wallet & billing status
- **readOnlyHint: true** — Reads credit balance, membership and auto-top-up status; writes nothing.
- **destructiveHint: false** — No state is modified.
- **idempotentHint: true** — Repeated calls have no additional effect.
- **openWorldHint: false** — Internal database only; Stripe is not called.

### `top_up_wallet` — Get a wallet top-up checkout link
- **readOnlyHint: false** — Creates a Stripe Checkout session (a new external object) for the human to pay in.
- **destructiveHint: false** — Nothing is charged, deleted or changed by the call itself; credits are added only after the human completes payment in Stripe.
- **idempotentHint: false** — Each call creates a new checkout session with a new URL.
- **openWorldHint: true** — Contacts Stripe, a third-party payment service.

### `list_projects` — List projects
- **readOnlyHint: true** — Lists the caller's projects from the database; writes nothing.
- **destructiveHint: false** — No state is modified.
- **idempotentHint: true** — Repeated calls have no additional effect.
- **openWorldHint: false** — Internal database only.

### `create_project` — Create a project
- **readOnlyHint: false** — Inserts a new project row and seeds an empty sandbox.
- **destructiveHint: false** — Purely additive: creates a new, empty project and never alters or removes existing projects, files or credits.
- **idempotentHint: false** — Every call creates another project.
- **openWorldHint: false** — Internal platform only; no external service or network fetch.

### `start_turn` — Start a build turn (PAID)
- **readOnlyHint: false** — Runs the coding agent on the project: it creates, edits and deletes project files and debits credits.
- **destructiveHint: true** — File edits and deletions happen inside the project (a checkpoint is taken so revert_turn can undo them) and credits are spent irreversibly; destructive by the MCP definition.
- **idempotentHint: false** — Each call starts another paid turn that does further work.
- **openWorldHint: true** — The agent installs packages from public registries and may call the web and third-party APIs while building.

### `check_turn` — Check build progress
- **readOnlyHint: true** — Reads turn status, transcript summary, cost and screenshot; the optional server-side wait is a read that blocks, not a write.
- **destructiveHint: false** — No state is modified.
- **idempotentHint: true** — Polling repeatedly has no additional effect.
- **openWorldHint: false** — Internal database and storage only.

### `revert_turn` — Undo a build turn (restore earlier files)
- **readOnlyHint: false** — Restores project files (optionally the database) to an earlier checkpoint.
- **destructiveHint: true** — Replaces the current working files with an earlier version; the pre-revert state is checkpointed first, but the live tree is overwritten.
- **idempotentHint: true** — Reverting to the same checkpoint again is detected as a no-op and makes no further change.
- **openWorldHint: false** — Operates only on platform-held checkpoints; no external service.

### `query_codebase` — Query the project codebase (structure, symbols, search)
- **readOnlyHint: true** — Reads the project tree, symbols and search hits; writes nothing.
- **destructiveHint: false** — No state is modified.
- **idempotentHint: true** — Repeated calls have no additional effect.
- **openWorldHint: false** — Reads the project's own files only.

### `security_scan` — Scan a project's dependencies for vulnerabilities and updates
- **readOnlyHint: true** — Reads the project manifest and lockfile and runs the audit in a scratch directory; the project itself is never modified.
- **destructiveHint: false** — No state is modified.
- **idempotentHint: true** — Repeated scans have no additional effect (results may differ only as advisories change).
- **openWorldHint: true** — Queries the public OSV vulnerability database and npm registry metadata.

### `get_preview_link` — Get a shareable preview link
- **readOnlyHint: true** — Mints a short-lived signed URL for the already-running development preview; no stored state changes.
- **destructiveHint: false** — No state is modified.
- **idempotentHint: true** — Each call returns a fresh signature for the same preview; nothing accumulates.
- **openWorldHint: false** — Internal platform only.

### `get_download_link` — Get a ZIP download link
- **readOnlyHint: true** — Mints a short-lived signed URL that serves a ZIP of the project source; no stored state changes.
- **destructiveHint: false** — No state is modified.
- **idempotentHint: true** — Each call returns a fresh signature for the same download; nothing accumulates.
- **openWorldHint: false** — Internal platform only.

### `export_github` — Export to GitHub
- **readOnlyHint: false** — Creates or updates a repository on the user's GitHub account.
- **destructiveHint: true** — On later exports files in the existing repository are overwritten with the current project source.
- **idempotentHint: false** — Each export pushes a new commit even if content is unchanged.
- **openWorldHint: true** — Contacts GitHub, a third-party service.

### `publish_site` — Publish to production (PAID, needs approval)
- **readOnlyHint: false** — Builds the project and deploys it to production hosting, charging the hosting fee.
- **destructiveHint: true** — Replaces the currently live deployment with the new build and spends credits; the previous version is superseded for visitors.
- **idempotentHint: false** — Each call starts a new build/deploy (unless the project is already live and unchanged, which returns without acting; force bypasses that).
- **openWorldHint: true** — The build pulls dependencies from public registries and the result is exposed on public hostnames.

### `check_publish` — Check publish status
- **readOnlyHint: true** — Reads deployment status and the live URL; writes nothing.
- **destructiveHint: false** — No state is modified.
- **idempotentHint: true** — Repeated calls have no additional effect.
- **openWorldHint: false** — Internal database only.

### `search_domains` — Search domain availability
- **readOnlyHint: true** — Checks availability and price of domain names; nothing is reserved or changed.
- **destructiveHint: false** — No state is modified.
- **idempotentHint: true** — Repeated searches have no additional effect.
- **openWorldHint: true** — Queries the external domain registrar.

### `buy_domain` — Buy a domain (checkout link for the user)
- **readOnlyHint: false** — Creates a Stripe Checkout session for the domain purchase.
- **destructiveHint: false** — The call itself charges and changes nothing; registration happens only after the human pays in Stripe. Additive even then (a new domain is added).
- **idempotentHint: false** — Each call creates a new checkout session.
- **openWorldHint: true** — Contacts Stripe and the domain registrar.

### `attach_domain` — Attach a domain the user already owns
- **readOnlyHint: false** — Registers a user-owned hostname for the project and issues DNS instructions.
- **destructiveHint: false** — Additive: adds a hostname mapping; refuses (409) if the hostname is attached elsewhere rather than taking it over. Removable by the user later.
- **idempotentHint: true** — Attaching a hostname that is already attached to this project returns the existing records without change.
- **openWorldHint: true** — Creates a custom hostname at Cloudflare and depends on the user's external DNS.
