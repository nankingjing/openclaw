# transcript-stats (community plugin)

Read-only diagnostic tool that aggregates counts, sizes, and time spans over
OpenClaw JSONL session transcripts. Ships as an **external** community plugin
(per ClawSweeper 2026-07-04 verdict on PR #99765) and is installable from
git, npm, or ClawHub.

## Install

```bash
# From this git branch (development snapshot)
openclaw plugins install git:github.com/nankingjing/openclaw@feat/transcript-stats-plugin --path extensions/transcript-stats

# From a local checkout
openclaw plugins install ./extensions/transcript-stats
openclaw plugins enable transcript-stats
```

Once installed and enabled, agents can call the `transcript_stats` tool.

## Tool

**`transcript_stats`** — aggregate stats over JSONL session transcripts.

Parameters:

- `scope` — `"workspace"` (default), `"agent"`, or `"recent"`
- `agentId` — required when `scope=agent`
- `recentLimit` — used when `scope=recent` (default 5, max 50)
- `workspaceDir` — base workspace path; defaults to current workspace
- `sessionsDir` — explicit override for the sessions directory

Read-only: never modifies session files, never invokes plugin runtime side
effects, never touches config.

## Scopes

| scope       | reads                                                 |
| ----------- | ----------------------------------------------------- |
| `workspace` | all `*.jsonl` under `<workspace>/sessions/`           |
| `agent`     | `<sessions>/<agentId>/*.jsonl`                        |
| `recent`    | last N `*.jsonl` (newest by name) under `<sessions>/` |

## Output

The tool returns a text block with:

- `session files scanned` — number of `*.jsonl` files inspected
- `total messages` — JSONL lines where `message` is an object
- `messages by role` — counts grouped by `message.role`
- `total tool calls` / `total tool results` — array-length counts on
  `message.tool_calls` and `message.tool_results`
- `total bytes` — UTF-8 byte count of the inspected content
- `time span` — min/max timestamp in ISO-8601 plus a `Xd Yh Zm` duration
- `longest message` — character count, role, and source session id
- `top 5 sessions by message count` (scope=recent only) — per-session counts

## Security boundary

- The tool reads JSONL from operator-supplied paths only. It does not
  enumerate or write to any other location.
- The tool is recommended for operator use, not for agent-driven invocation
  on user-supplied paths, because the read surface is the local file system.
- The plugin ships no default config, no admin scope, and no network calls.

## Tests

```bash
node node_modules/vitest/vitest.mjs run extensions/transcript-stats/index.test.ts
```

11 vitest cases (computeTranscriptStats, formatTranscriptStatsReport, tool
registration, scope validation, empty/populated dir handling).

## Provenance

- Repository: `https://github.com/nankingjing/openclaw`
- Source branch: `feat/transcript-stats-plugin`
- Original PR: `openclaw/openclaw#99765` (closed 2026-07-04 by maintainer
  review as a ClawHub scope-fit item — see that PR for the full review)
- ClawSweeper verdict recommended the ClawHub publication path over the
  bundled-core path; this README implements that recommendation.
