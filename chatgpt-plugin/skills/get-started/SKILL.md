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
   build loop (`create_project` → `start_turn` → `check_turn` until it reports
   done → share the preview link). `check_turn` waits on the server for up to
   30 seconds per call and returns as soon as the turn finishes, so call it
   again right away whenever it still reports the turn as running — no
   pausing between calls.

## Rules

- Never invent prices and never claim that something has been paid.
- `publish_site` reports the monthly cost and an approval link first. Relay
  both exactly: the user opens the link signed in to TakoCode and taps Approve
  (the server rejects `confirm:true` until they have). Call it again with
  `confirm:true` only after the user says they approved.
- To undo work the user does not like, call `revert_turn` (latest turn by
  default, or a specific `turn_id`). It restores the files from the checkpoint
  taken before that turn, costs nothing, hands back an undo id, and asks for
  `confirm: true` when later turns would be discarded — relay that to the user
  first. Prefer it over asking the agent to undo something in a new turn.
- `query_codebase` (outline, symbol, search, file) answers questions about a
  project's code and gives exact file paths and symbol names for free. Use it
  before `start_turn` on projects with many files and for "how does X work"
  questions instead of spending a turn.
- `security_scan` checks a project's dependencies against the public
  vulnerability databases and lists packages with newer versions, for free.
  Use it when the user asks whether their app is secure or up to date (or
  before publishing). It returns a ready-made fix prompt; relay the findings,
  and only after the user agrees start the repair with `start_turn` using that
  prompt (a normal build turn).
- Everything built lives in the user's TakoCode account at https://takocode.com,
  where they can keep working on it directly.
