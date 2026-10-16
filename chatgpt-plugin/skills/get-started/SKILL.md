---
name: takocode-get-started
description: Set up TakoCode after installing it — read the connected account, collect the four automation preferences, and save them before the first build.
---

# Get started with TakoCode

TakoCode's coding agent builds, hosts and publishes complete web apps on the
user's TakoCode account. You are the orchestrator; TakoCode does the coding.

## Setup steps

1. Call `takocode_get_started`. It reports whether the account is ready
   (membership, wallet balance, concurrency) and whether preferences are set.
2. If preferences are not set yet, ask the user these four questions in one
   message and wait for the answers:
   - **Autonomy** — `guided` (confirm each costly step), `auto_build` (build
     freely, confirm publishing), or `full_auto` (only stop for approvals the
     platform requires).
   - **Questions** — should TakoCode's agent `ask_user` when a design or
     product decision comes up, or `agent_decides` and note its assumptions?
   - **Quality tier** — `fast` (quick, cheaper) or `pro` (stronger model).
   - **Monthly budget** — an optional hard spend cap in US dollars, or none.
3. Save the answers with `set_preferences` and confirm what was saved.
4. If `takocode_get_started` reports that building is not available with the
   account's current membership or wallet balance, say so plainly and point
   the user to their TakoCode account (plans are described at
   https://takocode.com/pricing). Do not offer to buy anything — this plugin
   cannot purchase credits, memberships or domains.
5. Offer a first build: ask what the user wants to make, then follow the
   build loop (`create_project` → `start_turn` → poll `check_turn` every
   20–30 seconds → share the preview link).

## Rules

- Never invent prices and never claim that something has been paid.
- `publish_site` reports a monthly cost first; call it again with
  `confirm:true` only after the user approves.
- Everything built lives in the user's TakoCode account at https://takocode.com,
  where they can keep working on it directly.
