# Claude Code Statusline — Usage Bar (Build Guide)

## What this is (README)

Claude Code lets you run any command as a "statusline" — a line rendered above your prompt, refreshed on an interval. This guide builds one that shows, all on one line:

- **Model** in use and current working directory
- **Context window usage** — how much of the model's context you've used this conversation
- **30-day (weekly) plan usage** — your rolling 7-day Pro/Max quota
- **Session usage** — your rolling 5-hour Pro/Max quota, with a countdown to reset

Example output:

```
Sonnet 5 📁 .../code | CONTEXT ██████████────── 129.2k/200.0k 65.0% | 30d ████──────────── 28.0% | session ████──────────── 77.0% Resets in 2h43m
```

<img width="1336" height="44" alt="Statusline example" src="https://github.com/user-attachments/assets/1f455b6f-f57b-4d7e-8b9c-ab692a1063e7" />

This mirrors the "Plan usage limits" screen in the Claude app/desktop settings (Settings → Usage), but is always visible in the terminal without leaving Claude Code.

It's built on [ccstatusline](https://www.npmjs.com/package/ccstatusline), an open-source, config-driven statusline renderer for Claude Code. We don't write any custom code — just configuration.

## Instructions for Claude Code

If a user hands you this file and asks you to build this for them, do the following in order. Ask the user to confirm before overwriting any existing statusline config they already have.

1. **Check prerequisites** (see below) — confirm `npx` works and the user is logged into Claude Code with a Pro/Max/Team account (`claude`, then `/status`, check `Login method:`). If they're on API-key billing instead, tell them the `session`/`30d` segments will not populate (see Limitations) but the `CONTEXT` segment still works — ask if they want to proceed anyway.
2. **Read the user's current `~/.claude/settings.json`** (create it with `{}` if it doesn't exist) and merge in the `statusLine` block from Step 1 below — don't clobber other keys like `model` or `env`.
3. **Write `~/.config/ccstatusline/settings.json`** with the exact JSON from Step 2 below (create the `~/.config/ccstatusline/` directory if needed).
4. Tell the user to start a new prompt in Claude Code (or wait ~10s) and confirm the statusline renders. Compare against `/status` or the Claude app's Settings → Usage page if something looks off.
5. Walk through the "Known Issues" section below proactively if anything looks broken — most failures during this build were one of those five things.

## Prerequisites

- Claude Code CLI installed and on `PATH` (`claude --version`)
- `npx` available (ships with Node.js)
- Logged in via **Claude Pro, Max, or Team** (OAuth) for the `session-usage`, `weekly-usage`, and `reset-timer` widgets to populate. Run `/status` inside Claude Code — check `Login method:`. Pay-as-you-go API-key billing ("Console account") has no 5-hour/weekly quota concept, so those widgets will render blank (context usage still works fine).
- macOS: the usage widgets read the OAuth token from the macOS Keychain (service name `Claude Code-credentials`). On Linux/Windows, ccstatusline reads it from wherever Claude Code itself stores it — this should just work if `claude` itself is already logged in, no separate setup needed.

## Step 1 — `~/.claude/settings.json`

Add (or merge) this into the file:

```json
{
  "statusLine": {
    "type": "command",
    "command": "npx -y ccstatusline@2.2.27",
    "refreshInterval": 10
  }
}
```

Notes:
- The version is pinned (`@2.2.27`) rather than `@latest` on purpose — see "Known Issues #5" below.
- `refreshInterval` is in seconds. `10` is a reasonable default; going much lower increases how often the usage-API gets polled and can trigger rate-limiting (Known Issue #2).

## Step 2 — `~/.config/ccstatusline/settings.json`

This is the actual widget layout. Create the file with this exact content:

```json
{
  "version": 4,
  "colorLevel": 3,
  "flexMode": "full",
  "lines": [
    [
      { "id": "1", "type": "model", "color": "cyan", "rawValue": true },
      { "id": "1b", "type": "custom-text", "customText": " " },
      { "id": "2", "type": "current-working-dir", "color": "blue", "character": "📁", "rawValue": true, "metadata": { "segments": "1" } },
      { "id": "2b", "type": "custom-text", "customText": " " },
      { "id": "3", "type": "custom-text", "customText": "| ", "color": "brightBlack" },
      { "id": "4", "type": "custom-text", "customText": "CONTEXT ", "color": "white" },
      { "id": "5", "type": "context-bar", "color": "gradient:#8bc34a,#ffeb3b,#f44336", "rawValue": true, "metadata": { "display": "slider-only" } },
      { "id": "5b", "type": "custom-text", "customText": " ", "color": "white" },
      { "id": "5c", "type": "context-length", "color": "white", "rawValue": true },
      { "id": "5d", "type": "custom-text", "customText": "/", "color": "white" },
      { "id": "5e", "type": "context-window", "color": "white", "rawValue": true },
      { "id": "5f", "type": "custom-text", "customText": " ", "color": "white" },
      { "id": "5g", "type": "context-percentage", "color": "white", "rawValue": true },
      { "id": "6", "type": "separator" },
      { "id": "7", "type": "custom-text", "customText": "30d ", "color": "white" },
      { "id": "8", "type": "weekly-usage", "color": "gradient:#8bc34a,#ffeb3b,#f44336", "rawValue": true, "metadata": { "display": "slider-only" } },
      { "id": "8b", "type": "custom-text", "customText": " ", "color": "white" },
      { "id": "8c", "type": "weekly-usage", "color": "white", "rawValue": true },
      { "id": "9", "type": "separator" },
      { "id": "10", "type": "custom-text", "customText": "session ", "color": "white" },
      { "id": "10c", "type": "session-usage", "color": "gradient:#8bc34a,#ffeb3b,#f44336", "rawValue": true, "metadata": { "display": "slider-only" } },
      { "id": "10d", "type": "custom-text", "customText": " ", "color": "white" },
      { "id": "11", "type": "session-usage", "color": "white", "rawValue": true },
      { "id": "12", "type": "custom-text", "customText": " Resets in ", "color": "white" },
      { "id": "13", "type": "reset-timer", "color": "white", "rawValue": true, "metadata": { "compact": "true" } }
    ],
    [],
    []
  ]
}
```

### Segment cheat-sheet (widget types used)

| type | shows |
|---|---|
| `model` | current model name (e.g. "Sonnet 5") |
| `current-working-dir` | abbreviated cwd with a folder icon |
| `context-bar` / `context-length` / `context-window` / `context-percentage` | current conversation's context window usage |
| `weekly-usage` | rolling 7-day Pro/Max quota, as a bar and/or percentage |
| `session-usage` | rolling 5-hour Pro/Max quota, as a bar and/or percentage |
| `reset-timer` | countdown to the 5-hour session window resetting |
| `separator` | ccstatusline's built-in `|` divider |
| `custom-text` | literal text/spacing/labels |

`metadata.display: "slider-only"` renders just the bar (no inline percentage) — that's why bar and percentage/number are split into separate segments (e.g. `8` + `8c`) rather than one combined widget: it lets the gradient color apply only to the bar, keeping numbers plain white and legible.

## How it works

- **Context** numbers come straight from the JSON payload Claude Code itself passes to the statusline command on every render — no network call.
- **Weekly / session usage percentages** come from Anthropic's `/api/oauth/usage` endpoint (polled in the background, cached locally at `~/.cache/ccstatusline/usage.json`, ~3 min TTL), *or*, when that's stale/unavailable, from rate-limit headers Anthropic attaches to your actual chat responses in real time. This second path only updates when you send a message in that specific window — an idle Claude Code window can show a slightly stale percentage until its next message.
- **Reset timer** prefers the real `resets_at` timestamp from the usage API, but falls back to a locally-cached "block start time" (`~/.cache/ccstatusline/block-cache-<hash>.json`) to estimate a 5-hour countdown even if the network call is failing — this is why the countdown is usually accurate even when the percentage lags.

## Known issues encountered while building this (and fixes)

**1. Whole line shows `⚠ invalid config`**
Cause: a `metadata` value was a JSON boolean (`true`) instead of a string (`"true"`). ccstatusline's schema requires `metadata: Record<string, string>` — every value must be a string, even flags. Fix: always quote metadata values, e.g. `"metadata": { "compact": "true" }`, never `"compact": true`.

**2. Session/weekly percentage stuck on an old number, or `resets 0m` / `[Rate limited]`**
The usage-API poll can get rate-limited by Anthropic (this presents as a lock file at `~/.cache/ccstatusline/usage.lock` containing `{"blockedUntil": <epoch>, "error": "rate-limited"}`). While blocked, ccstatusline serves the last successfully cached `usage.json` — which can be days or weeks old if the poll has been failing for a while, producing a stale percentage and a `resets 0m` (because the cached reset timestamp is long past, and a *present-but-expired* timestamp does **not** trigger the local fallback estimate — only a *missing* one does).
- To check: `cat ~/.cache/ccstatusline/usage.lock` and compare `blockedUntil` (Unix epoch, UTC) against `date -u`.
- This is a real, server-side rate limit — it clears on its own; you can't force it early.
- Deleting `~/.cache/ccstatusline/usage.json` (not the lock) stops it from displaying stale/wrong numbers in the meantime — it'll honestly show `[Rate limited]` (or blank) until the block clears and a fresh poll succeeds.
- Root cause is usually polling too aggressively (very low `refreshInterval` across several open Claude Code windows). `refreshInterval: 10` combined with normal `/status` usage is generally fine; going much lower isn't recommended.

**3. Line gets cut off / truncated on a narrower terminal window**
Default `flexMode` is `"full-minus-40"` — it reserves 40 columns of the terminal width as a safety margin (originally for Claude Code's own auto-compact warning message). Set `"flexMode": "full"` at the top level of `~/.config/ccstatusline/settings.json` to use the full terminal width before truncating.

**4. Percentage looks lower than what the Claude app shows**
Not a bug — see "How it works" above. The live-header path only updates per message; an idle window naturally lags behind an active one, and both lag slightly behind the app's on-demand refresh. It self-corrects on the next message / next successful background poll.

**5. `npx -y ccstatusline@latest` intermittently reverts to a different width/look, or ignores your config**
Not hit on this machine, but documented at length in [[Usage bar]]: a separate global npm install (`npm install -g ccstatusline`) can shadow the npx cache copy depending on whether an exact version or `@latest` is used, and re-running the ccstatusline interactive setup TUI can silently overwrite your hand-edited config. Pinning an exact version (as done in Step 1) avoids the ambiguity. If you ever run the interactive TUI (`npx ccstatusline@2.2.27` with no piped input), don't save unless you mean to overwrite this config.

**6. Session percentage number renders in the gradient color instead of plain white**
Segment `11` (the numeric session percentage, paired with the `10c` bar) was originally copy-pasted from the bar segment and kept `"color": "gradient:#8bc34a,#ffeb3b,#f44336"` instead of `"color": "white"`. Compare against `8c` (the weekly percentage number), which was already correct. Fix: set segment `11`'s `color` to `"white"` — the JSON above already reflects this fix.

## Limitations

- `session-usage`, `weekly-usage`, and `reset-timer` require a **Claude Pro/Max/Team (OAuth) login** — they will not populate under API-key/Console billing, since pay-as-you-go usage has no such rolling-window concept.
- Numbers can lag the Claude app's Settings → Usage page by a small margin (see "How it works") — treat this as "close enough for a glance," not a billing-grade source of truth.
- This machine wires `statusLine.command` directly to `npx -y ccstatusline@2.2.27`. If you're also running the separate Headroom desktop menu-bar app (unrelated to the `headroom` CLI context-optimization proxy — same maker, different product) and want it fed the same JSON, see [[Usage bar]] for the `headroom-statusline.sh` wrapper-script pattern used on other machines — not needed for the terminal statusline alone.

## Quick troubleshooting reference

| Symptom | Check | Fix |
|---|---|---|
| `⚠ invalid config` | Any `metadata` values that aren't strings | Quote booleans: `"true"` not `true` |
| Session/weekly stuck or `resets 0m` | `~/.cache/ccstatusline/usage.lock` | Wait for `blockedUntil` (UTC epoch) to pass; optionally `rm ~/.cache/ccstatusline/usage.json` meanwhile |
| Line truncated on narrow terminal | `flexMode` in ccstatusline settings | Set to `"full"` |
| Session/weekly blank entirely | `/status` → `Login method:` | Must be Pro/Max/Team (OAuth), not API key |
| Config edits don't seem to apply | `~/.claude/settings.json` → `statusLine.command` | Confirm it points at the same ccstatusline version you're editing config for |
