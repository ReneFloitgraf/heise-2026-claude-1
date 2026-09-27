# Session 5 storyboard (working draft)

Working document for designing session 5. It holds every finding, probe result, and
decision from the concept phase, so another agent on another computer can continue
without the original conversation. The final `storybook-05.md` gets written from it.

This file is deliberately not named `storybook-*.md`, because `_quarto.yml` renders every
file matching that glob into a PDF.

## Status and how to resume

- **Concept phase: done** (2026-09-27). Rainer: "we have enough material for the day; it
  is fine to drop some of the detail agenda items." All decisions are in "Decisions"
  (1 to 21).
- **Next phase: build and validate.** Follow "Work plan" below, in order.
- **Only the probes marked as such have been run.** No storybook prompt has been
  dry-run yet, and the part C program does not exist yet.
- **Before you start on another computer**, read "Handoff: environment and working
  rules". The starter is on GitHub (`rstropek/2026-claude-classroom-5-starter`).

## Handoff: environment and working rules

### Where things are

| What | Where | State |
|---|---|---|
| This draft | classroom repo, `storyboard-05-draft.md` | committed with this handoff |
| Session 5 starter | `https://github.com/rstropek/2026-claude-classroom-5-starter` (public); first-Mac clone at `/Users/rstropek/live/2026-claude-classroom-5-starter` | pushed, `main` at `8c18441` (commits `e8de70b`, `b9db988`, `efc22ba`, `8c18441`) |
| Starter content | session 4 result flattened to the repo root, plus `demos/` | see decision 10 |
| Earlier storybooks | `storybook-01.md` to `storybook-04.md` in the classroom repo | format reference |
| Session 4 validation artifacts | `/Users/rstropek/live/2026-claude-classroom-4-validation` (first Mac) | reference for run logs, `runs.md` |

Clone the starter from GitHub. Local commits for restore points are fine; **ask Rainer
before every further push**, because students fork this repository.

### Tools the demos need

- Claude Code (the probes ran on 2.1.283), `jq` (1.8), `gh`, Node 24, `uv` (for
  `uv run --with pillow` in A7), Docker (Jaeger for C6).
- pi coding agent (probes ran on 0.85.0, see pi.dev) with OpenRouter credentials. On the
  first Mac pi's own settings load the packages `pi-subagents` and `pi-mcp-adapter` and
  default to thinking `high`; demos pin `--no-extensions` and a thinking level so students
  without those packages see the same thing.
- Playwright Chromium comes with the starter's `npm install`.

### Keys and secrets

- **OpenRouter key:** Rainer put one into `session-4-useRenderTool/.env` in the classroom
  repo on the first Mac (`OPENROUTER_API_KEY=`). On another computer, ask him where it is.
  Use it only through `source demos/openrouter.env.sh <path-to-.env>` and never print it.
  The auto-mode classifier blocks extracting the key from pi's credential store
  (`pi auth print-api-key`), so don't try that route.
- **Anthropic API key for CI:** Rainer provides it (decision 19). It goes into the GitHub
  repo secret `ANTHROPIC_API_KEY` via `gh secret set`, typed by Rainer, never into a chat.
- **Claude subscription:** local `claude -p` and Agent SDK runs use the logged-in account
  unless an API key or the OpenRouter variables are set.

### Working rules carried over from sessions 1 to 4

- **Audience:** professional developers with years of experience, little agentic AI
  experience (classroom `CLAUDE.md`). Storybook text in English, American spelling.
- **Storybook format:** each step has **Goal**, **Prompt**, **Teaching points**,
  **Verify**; appendix A = headless rehearsal recipe, appendix B = live-demo insurance.
  Step numbers continue: session 4 ended at step 24, so session 5 starts at **step 25**.
- **Writing:** load the classroom repo's `writing-guide` skill before drafting storybook
  text, and run its audit greps before committing. Diagrams: `svgbob` skill, `.bob`
  source plus rendered `.svg` in `images/` (svgbob_cli is in `~/.cargo/bin` on the first
  Mac).
- **Time budget per live prompt (Rainer's rule):** 5 min fine, 10 min "ok-ish", longer
  only when it adds real value. Levers that worked in session 4: put plain code in the
  starter; tell the prompt to run `npm test` and `npm run lint` and skip `next build` and
  e2e; point at the `AGENTS.md` section that holds the context.
- **Validation pattern:** copy the starter to a sibling `-work` directory, run every live
  prompt headless with `claude --model claude-opus-5-5 --dangerously-skip-permissions -p
  "<prompt>"`, one branch per step, merge between steps, log runtime and cost per run. The
  Bash classifier blocks writing a script file that contains
  `--dangerously-skip-permissions`, so run that command inline. Verify in a real browser
  with an Opus subagent where a UI is involved; use Opus subagents for verification and
  chores, and the main model for judgment and prose (Rainer's cost rule).
- **The coding agent avoids spending OpenRouter credit** and will not notice when the app's
  own model misbehaves. Prompts that need the app's chat (A7) must allow it explicitly.
- **`session-5-result/` is not produced during preparation.** Rainer adds it after the live
  session; the README row says "follows" until then.
- **Working style Rainer asked for:** brainstorm in rounds, give a recommendation, ask
  concrete questions, record every answer under "Decisions". Keep demos "behind the
  scenes" rather than polished product facades.

## Session facts

- Heise classroom session 5, **29.09.2026, 09:00 to 13:00** (four hours).
- Title: "Agent SDK, Subagents und CI/CD - deinen Agenten produktionsreif machen".
- Announced agenda (short form): Claude Code vs. Agent SDK vs. API; CI/CD for the
  Next.js/Mastra app on GitHub Actions; Claude Code for issue triage, PR reviews and
  change proposals; Agent SDK workflows in TypeScript (analysis, test run, fix proposals,
  reports); subagents (architecture, security, test review, docs; isolation, worktrees,
  when they help and when they cost); evals and OpenTelemetry; a higher orchestration
  layer across providers; human-in-the-loop (approvals, review gates, rollback,
  accountability); deploying the finished agent.
- How the agenda is covered (decision 21): parts A, B, C, E below carry it. Evals, the
  orchestration layer, and deployment are talk only or dropped; ACP is dropped.

## Running order (proposal, not yet agreed)

| Block | Content | Rough time |
|---|---|---|
| 0 | Recap, fork the starter, `npm install`, `.env` | 10 min |
| A | `claude -p`: A0 to A7 | about 60 min |
| B | Bake-off: Claude Code and pi on open-weight models | about 15 min |
| C | Agent SDK program, C1 to C6, walked through | about 60 min |
| E | CI with `claude -p` as a pipeline step | about 35 min |
| | Breaks | 20 min |
| | Talk: deployment, rollback, hosting, what was left out; wrap-up | 20 min |

Total about 220 min of 240. Parts A and B together were estimated at 75 min (decision 5).

## The starter (what exists today)

- Session 4 result at the repo root: Next.js 16, Mastra agent Bartholomew (`lib/tutor.ts`,
  tools `listTodos`, `addTodo`, `setTodoDone`), Better Auth, SQLite via `DATABASE_URL`
  (`drizzle-orm/libsql/node` and Mastra `LibSQLStore`), REST API, CLI with stdio MCP
  server, HTTP MCP server with OAuth, A2UI wizard, MCP App, Vitest, Playwright, Biome,
  `AGENTS.md` map, skills under `.agents/skills/` and `.claude/skills/`, CodeTours.
- **No MCP config** (`.mcp.json` removed, decision 10) and **no `.github/`** yet.
- `demos/` (commit `b9db988`), plumbing tested on Haiku 4.5 through OpenRouter:

| File | Used in | What it is |
|---|---|---|
| `demos/README.md` | all | index of the demo files |
| `demos/openrouter.env.sh` | A0 | `source demos/openrouter.env.sh [path/.env]` exports `ANTHROPIC_BASE_URL=https://openrouter.ai/api`, `ANTHROPIC_AUTH_TOKEN=<key from .env>`, `ANTHROPIC_API_KEY=""` for the current shell; `--off` unsets them; never prints the key |
| `demos/a2-doors.schema.json` | A2 | `--json-schema` for the inventory of ways into the todo list (`doors[]` with `door`, `entryFile`, `auth`, `operations[]`) |
| `demos/a2-findings.schema.json` | A2, A6, E2 | `--json-schema` for findings (`severity` high/medium/low, `file`, `line`, `title`, `fix`) |
| `demos/a3-tool-calls.jq` | A3 | `jq -r -f` filter for `stream-json`: one line per tool call (paths relative to `$PWD`, subagent calls indented via `parent_tool_use_id`), one summary line with turns, seconds, USD |

## Part A: Claude Code non-interactive (`claude -p`)

### Findings from the docs

Sources: `https://code.claude.com/docs/en/headless.md` and `cli-reference.md`. The
`platform.claude.com/llms.txt` index covers the API; the Claude Code pages are indexed at
`https://code.claude.com/docs/llms.txt`.

- The docs call `claude -p` "the Agent SDK via the CLI": same loop, tools, and context
  management as the TUI. That is the bridge to part C.
- **Without `--bare`, `-p` loads everything the TUI loads** (`CLAUDE.md`/`AGENTS.md`,
  skills, hooks from `.claude/settings.json`, MCP servers from `.mcp.json`) **without a
  workspace trust dialog and without per-server approval.** Teaching point for CI: running
  `-p` in an untrusted checkout runs that checkout's hooks.
- `--bare` skips all auto-discovery for reproducible CI runs and never reads the
  subscription login. It works with `ANTHROPIC_API_KEY`, an `apiKeyHelper`, or (probed)
  `ANTHROPIC_AUTH_TOKEN` through OpenRouter. The docs say `--bare` will become the default
  for `-p`.
- `--output-format json`: `result`, `session_id`, `total_cost_usd`, `modelUsage` per model,
  `num_turns`, `duration_ms`, `permission_denials`, `subagent_stats`, `structured_output`.
- `--json-schema '<schema>'` validates the final answer after multi-turn tool use; invalid
  schema is a hard error.
- `--output-format stream-json --verbose`: one JSON event per line, `system/init` first
  (model, tools, plugins, `plugin_errors`, `mcp_server_errors`). `--include-partial-messages`
  adds token deltas. Subagent events carry `parent_tool_use_id`;
  `--forward-subagent-text` adds their text.
- Prompt control: `--append-system-prompt[-file]` adds, `--system-prompt[-file]`
  replaces, `--append-subagent-system-prompt` (print mode only), `--agents '<json>'`
  inline subagents.
- Tool control has three layers: `--tools` (what exists), `--allowedTools` (pre-approved,
  rule syntax such as `Bash(git diff *)`), `--permission-mode dontAsk` (deny the rest).
  `--permission-prompts none` tells Claude nobody can answer.
- Guard rails: `--max-budget-usd` (subagents count), `--max-turns`, `--fallback-model`,
  `--effort`, `--no-session-persistence`.
- Sessions: capture `session_id`, `--resume <id>`, `--continue`, `--fork-session`.
- `/skill-name` in the prompt invokes a skill; stdin is piped in (10 MB cap); exit code 0
  on success.

### Probes (2026-09-27, Claude Code 2.1.283, run in the session 4 result)

| Probe | Command essentials | Result |
|---|---|---|
| Typed output | `claude -p --model haiku --output-format json --tools "Read,Grep,Glob" --json-schema '{file, tools[]}' "Which file defines the Mastra agent, and which tools does it register?"` | `{"file":"lib/tutor.ts","tools":["listTodos","addTodo","setTodoDone"]}`, 5 turns, 10.3 s, 0.13 USD |
| Security persona (Opus 5) | `--append-system-prompt "You are a security engineer reviewing this codebase before a production release. Report vulnerabilities with severity, file:line, and a fix."`, question "Is there anything risky in app/api/todos? Keep it under 200 words." | **No refusal**, 6 turns, 25.5 s, 0.76 USD. Nothing high; three low items (no `Cache-Control: private, no-store` on the per-user GET, no rate limit on POST, `z.prettifyError` echoed in 400s). Defensive review of your own code is fine; the official `review.sh` docs example uses the same persona |
| OpenRouter, Haiku | `source demos/openrouter.env.sh ...; claude -p --model claude-haiku-4-5 ...` "Which file defines the Mastra agent? One sentence." | correct from `AGENTS.md`, 1 turn, 4 s, 0.017 USD |
| OpenRouter, glm | same with `--model z-ai/glm-5.3-flash` | correct, 1 turn, 8 s; Claude Code logs `[claude-code:unrecognized_model]` and carries on; its `total_cost_usd` (0.032) is wrong for non-Claude models |
| OpenRouter, `--bare` | same as Haiku plus `--bare` | correct but **13 turns, 25 s, 0.045 USD**: without `AGENTS.md` the agent has to search |
| A2 plumbing | Haiku via OpenRouter, `--json-schema "$(cat demos/a2-doors.schema.json)"`, piped through `jq -r '.structured_output.doors[] \| [...] \| @tsv' \| column -t -s $'\t'` | table renders; 18 turns, 30 s, 0.05 USD. Haiku split REST into three rows and listed the AG-UI bridge separately, so the prompt must name the five doors: chat, REST, CLI, stdio MCP, HTTP MCP |
| A3 plumbing | `--output-format stream-json --verbose ... \| jq -r -f demos/a3-tool-calls.jq` | one line per tool call, summary line; 6 turns, 16 s, 0.03 USD |

OpenRouter notes: the plain Anthropic model id works (no `anthropic/` prefix); Opus 5.5
through OpenRouter is not probed yet. Harmless stderr notices: "claude.ai connectors are
disabled because ANTHROPIC_API_KEY or another auth source is set", and a note that
auto-mode classifier requests stay billed through openrouter.ai.

### Demos (all decided: decisions 2, 3, 5)

All demos run at the starter root and leave the source untouched; A6 and A7 write only
into a report folder.

**A0. Claude through OpenRouter.** For students without a subscription: `source
demos/openrouter.env.sh`, then any demo works. Recipe from OpenRouter's docs
(`cookbook/coding-agents/claude-code-integration`): base URL without `/v1`,
`ANTHROPIC_API_KEY` must be empty. Model overrides also via
`ANTHROPIC_DEFAULT_OPUS_MODEL` and friends. Teaching point: the harness is separate from
the model endpoint.

**A1. Same agent, no window.** `claude -p "What does this app do? Five bullets."`, then
`git log --oneline -40 | claude -p "..."` for stdin. Then
`claude -p "hi" --output-format stream-json --verbose | jq 'select(.subtype=="init")'`
shows the skills and tools that loaded without a trust prompt (no `.mcp.json` in the
starter, so the point rests on skills, `AGENTS.md`, and project hooks). Contrast with
`--bare`: tell the measured numbers (1 turn vs. 13, six times the wall time), which put a
number on session 1's project memory.

**A2. The agent as a typed function.** `demos/a2-doors.schema.json` inventory of the five
doors into the todo list as a terminal table (recap of session 3). Second half:
`demos/a2-findings.schema.json` and
`jq -e '[.structured_output.findings[] | select(.severity=="high")] | length == 0'` as a
gate that sets the exit code. E2 reuses exactly this.

**A3. Flight recorder.** `stream-json` piped through `demos/a3-tool-calls.jq`: one line
per tool call while the agent works, cost line at the end. Then `--max-budget-usd 0.05`
for a hard stop.

**A4. Change the prompt.** One question ("Is there anything risky in `app/api/todos`?"),
three runs: default; `--append-system-prompt` security persona (probed, no refusal; the
persona changes the shape of the answer more than its content); `--system-prompt "You
are Bartholomew, a butler..." --tools ""`, which turns Claude Code into a plain chat model
with the app's own persona.

**A5. Explore once, fork twice.** First run explores and returns `session_id`; two
`--resume $sid --fork-session` runs write an onboarding note and a list of test gaps.
Compare `total_cost_usd`: the forks reuse the cached context.

**A6. Code review as a Markdown report (required by Rainer).**

- Rubric file `demos/review-rubric.md` (to write) passed with
  `--append-system-prompt-file`, so the review is repeatable and versioned.
- Read-only tools plus one write target, for example `--permission-mode dontAsk
  --allowedTools "Read" "Grep" "Glob" "Edit(./reports/**)"`. **Verify the rule path
  syntax** before relying on it.
- Output `reports/code-review.md`: summary, Mermaid diagram of the five doors, findings
  table (severity, `file:line` links clickable in VS Code, fix), what is good, test gaps.
- Optional: the same run also returns `--json-schema` findings, so one analysis feeds the
  human report and the A2 gate.

**A7. Test protocol with annotated screenshots (required by Rainer).**

- Prerequisite: `npm install`, `.env` with the OpenRouter key, `npm run db:migrate`,
  `npm run dev` on a known port (routine for students since session 1).
- Four journeys: sign up; ask Bartholomew to add a todo and see the sidebar update; open
  `/projects/new`; sign out.
- Claude writes an **ad-hoc Node.js script** with the repo's Playwright: screenshots plus
  bounding boxes of the elements under test as JSON (coordinates from the DOM, not
  guessed).
- Claude writes an **ad-hoc Python script** run with `uv run --with pillow` (Pillow is not
  installed globally) that draws numbered boxes, arrows, and labels.
- Claude **opens the annotated PNGs** (the Read tool is multimodal) and fixes misplaced
  annotations.
- Output `test-protocol/PROTOCOL.md` with action, expected, actual, pass/fail, image per
  step. The prompt must allow the one chat turn (fraction of a cent of OpenRouter credit).
- Fallback if the live walk is fragile: annotate screenshots from the existing Playwright
  specs (`auth.spec.ts`, `project-wizard.spec.ts`).

Bonus one-liners if time allows: `claude -p` as an npm script
(`"lint:claude": "git diff main | claude -p ..."`), `/security-review` by name,
`--max-turns 3`.

## Part B: a second harness with open-weight models (pi)

### Findings

- pi non-interactive: `pi -p`; machine output `--mode json` (event stream) or
  `--mode rpc`. Read-only: `--tools read,grep,find,ls`. Reproducibility: `--no-session`,
  `--no-extensions`, `--no-skills`, `--no-context-files`, `--thinking <level>`.
- pi loads `AGENTS.md`/`CLAUDE.md` by default, like Claude Code, so the research question
  must need facts `AGENTS.md` does not give away.
- Both models are in pi's catalog: `openrouter/z-ai/glm-5.3-flash` and
  `openrouter/moonshotai/kimi-k3` (1M context each).
- Probe (glm-5.3-flash, "Which file defines the Mastra agent?"): 2 s, correct, zero tool
  calls (answered from `AGENTS.md`), 7,337 input and 31 output tokens, 0.00056 USD. JSONL
  event types: `session`, `agent_start`, `turn_start`, `message_start/update/end`,
  `turn_end`, `agent_end`, `agent_settled`. Every `message_end` carries `usage` with a
  `cost` object, so jq can sum tokens, cost, and tool calls.

### B1: harness vs. model bake-off (decisions 4 and 6)

Claude Code, interactive so the room sees it work, gets one prompt: run the three
contestants in parallel as background jobs on the same question, save the JSONL, compute
runtime, turns, tool calls by name, tokens, and cost with jq, check every claim against
the code, grade (correct, wrong, missing), present one table and a verdict.

Question (decision 4): "For each of the ways into the todo list, show where the user's
identity is established and where the query is scoped to that user, with file and line."

| Run | Harness | Model | Command essentials |
|---|---|---|---|
| 1 | Claude Code | glm-5.3-flash | `source demos/openrouter.env.sh`; `claude -p --model z-ai/glm-5.3-flash --tools "Read,Grep,Glob" --output-format stream-json --verbose` |
| 2 | pi | glm-5.3-flash | `pi -p --mode json --no-session --no-extensions --tools read,grep,find,ls --model openrouter/z-ai/glm-5.3-flash` |
| 3 | pi | kimi-k3 | as run 2 with `--model openrouter/moonshotai/kimi-k3` |

Open details: pin the thinking level in both harnesses; take run 1's cost from OpenRouter
(generation API or activity page), not from `total_cost_usd`; probably no `--bare` for
run 1, since pi loads `AGENTS.md` too.

Teaching points: harness and model are separate choices; same model in two harnesses
isolates the harness effect; the JSONL event stream makes harnesses comparable; Claude
grading another model's answer is a model-graded eval (the only evals content left in the
day, decision 21).

## Part C: Agent SDK

### What it is

- Docs: "Build production AI agents with Claude Code as a library". `query()` in
  `@anthropic-ai/claude-agent-sdk` (0.3.283, bundles and pins CLI 2.1.283) **spawns the
  Claude Code binary as a subprocess** and talks stream-JSON over stdio. One session = one
  subprocess with its own shell, cwd, and JSONL transcript. The binary comes through npm
  optional dependencies (`npm ci --omit=optional` breaks it).
- **An agent harness as a library, not a generic agent framework.** The OpenAI Agents SDK
  runs the loop in your process against the Responses API and you bring every tool. The
  Claude Agent SDK gives you Claude Code's loop, built-in tools, compaction, permissions,
  sessions, skills, and `CLAUDE.md` loading, and your code steers it. The closer OpenAI
  analogue is the Codex SDK wrapping `codex exec` (**double-check before class**). In this
  classroom, Mastra is the generic framework (the app's tutor); the Agent SDK is the coding
  agent you script.
- `query()` emits the same message types as `claude -p --output-format stream-json`.
- **What code adds over `-p` flags:** `canUseTool` callback (the permission prompt becomes
  your function, `updatedInput` can rewrite a call); hooks as TypeScript callbacks
  (`PreToolUse`, `PostToolUse`, `SubagentStart/Stop`, ...); custom tools as an in-process
  MCP server (`tool()` + `createSdkMcpServer()`, zod input schema); `outputFormat` with a
  JSON schema; `agents` with `AgentDefinition` (prompt, tools, model, effort, maxTurns,
  permissionMode, skills); `enableFileCheckpointing` plus `rewindFiles(checkpointId)` (only
  Write/Edit/NotebookEdit; not Bash, not subagent edits; needs `replay-user-messages` in
  `extraArgs` to receive checkpoint UUIDs); `sessionStore`; `streamInput()`;
  `settingSources: []` to load nothing from disk.
- **OpenTelemetry is built into the CLI**; the SDK passes env vars:
  `CLAUDE_CODE_ENABLE_TELEMETRY=1`, `OTEL_METRICS_EXPORTER`, `OTEL_LOGS_EXPORTER`, and for
  traces `OTEL_TRACES_EXPORTER=otlp` plus `CLAUDE_CODE_ENHANCED_TELEMETRY_BETA=1`, then the
  standard `OTEL_EXPORTER_OTLP_*`. In TypeScript `options.env` replaces the environment, so
  spread `process.env`.
- `AgentDefinition` has **no `isolation: "worktree"`** (the CLI's subagent frontmatter
  has). From the SDK, your code runs `git worktree add` and passes a separate `cwd` per
  `query()`; `projectConfigRoot` names the trusted checkout the worktree belongs to.
- Peer dependencies: `zod ^4`, `@anthropic-ai/sdk >=0.93`, `@modelcontextprotocol/sdk
  ^1.29`. The app uses the MCP v2 split packages (`@modelcontextprotocol/server` 2.0), so
  the program gets its own npm workspace. **Verify** that npm keeps both apart.
- Docs pages used: `https://code.claude.com/docs/en/agent-sdk/{overview,typescript,
  custom-tools,subagents,user-input,permissions,hooks,file-checkpointing,observability,
  cost-tracking,hosting,sessions}.md`.

### Subscription (probed 2026-09-27)

Probe script (Node, ESM, run with the target repo as argument, no `ANTHROPIC_API_KEY`):

```js
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const m of query({
  prompt: "Which file defines the Mastra agent? One sentence.",
  options: { cwd: process.argv[2], model: "claude-haiku-4-5", tools: ["Read", "Grep", "Glob"], persistSession: false },
})) {
  if (m.type === "system" && m.subtype === "init") console.log({ apiKeySource: m.apiKeySource, skills: m.skills?.length });
  if (m.type === "result") console.log({ text: m.result, turns: m.num_turns, cost: m.total_cost_usd });
}
```

Result: `apiKeySource: 'none'`, so the spawned CLI used the claude.ai login. Correct
answer, 1 turn, 0.087 USD. It loaded 70 skills: the SDK loads user, project, and local
settings by default, like the CLI.

- Technically the SDK reuses your subscription for your own local scripts. The docs'
  policy line is about products: "Anthropic does not allow third party developers to
  offer claude.ai login or rate limits for their products, including agents built on the
  Claude Agent SDK." Anything hosted or given to others uses an API key. For your own CI,
  `claude setup-token` creates `CLAUDE_CODE_OAUTH_TOKEN`; the docs recommend an API key
  for secrets shared across repositories.
- OpenRouter should work with the same three variables (OpenRouter has an Agent SDK
  guide). **Not probed with the SDK yet**; do it when writing C1.

### The program (decisions 11 to 17)

One TypeScript program, working name `tutor-maintainer`, in its own npm workspace
`agents/` at the starter root, one CLI with subcommands (working shape
`npm run maintainer -- <subcommand>`). It runs with `cwd` = repo root, so it loads the
app's `AGENTS.md` and skills. **All code is pre-written into the starter and walked
through** (CodeTour per step, as in session 4, to confirm with Rainer). No live coding
prompts in part C.

| Step | Adds | Agenda item |
|---|---|---|
| C1 | `query()` hello: the A2 question, same events as `stream-json`, `apiKeySource`, `ps` shows the spawned `claude` | Claude Code vs. SDK vs. API |
| C2 | Custom tool `run_tests` (in-process MCP, wraps Vitest's JSON reporter), `Bash` removed; typed `outputFormat` via zod (`z.toJSONSchema`) | Code analysis, test run |
| C3 | Fix proposals with a human gate: `canUseTool` shows the diff and asks **y/n in the terminal**; `PreToolUse` hook hard-denies `.env*`, `drizzle/`, `lib/auth*.ts`, `package.json`; checkpointing plus `rewindFiles()` when tests go red after a fix | Fix proposals; human-in-the-loop, rollback |
| C4 | Review team: `agents` with architecture, security, test, and docs reviewers, model and tools per role, in parallel; report with cost per subagent. Live: team only. The single-agent comparison is **shown as numbers from the dry run** | Subagents, when they help and when they cost |
| C5 | Two fixers in two `git worktree`s via separate `cwd`; the program keeps the fix whose tests pass | Isolation, worktrees |
| C6 | OTel env vars; traces of a C4 run in **Jaeger** (one Docker container, OTLP in) | Observability |

Expect C4 on Opus 5.5 to cost several USD per run; log it in the dry run.

## Part D: ACP (dropped, decision 21)

Rainer's existing demo is `github.com/rstropek/2026-ai-protocols/typescript/acp`. It was not
reviewed, because part D is out of the day.

## Part E: Claude + GitHub, CI only (decisions 18 to 20)

### Findings

- The starter has no `.github/` yet.
- Three Anthropic products touch GitHub:
  - `anthropics/claude-code-action@v1`: a step in your own workflow, "built on the SDK".
    Interactive mode (`@claude` in issues/PRs, answers in a comment) and automation mode
    (`prompt` set, any event incl. `schedule`). `claude_args` passes CLI flags. Auth
    `anthropic_api_key` or `claude_code_oauth_token`. Setup `/install-github-app`. Guards:
    trigger user needs write access, bots rejected unless in `allowed_bots`, no secrets on
    fork PRs.
  - **Code Review** (managed): multi-agent PR review on Anthropic infrastructure, severity
    markers, `REVIEW.md`, about 20 min per review. Research preview, Team and Enterprise
    only.
  - **Routines** (managed): saved prompt plus repo, triggered by schedule, API POST, or
    GitHub events. Research preview, Pro and up. `/schedule` in the CLI.
- Starter repos are public and students fork them; their secrets are their own.

### Shape: behind the scenes, not a polished integration

Rationale (Rainer): the session shows what happens behind the scenes; attendees get links
for the polished products. `claude-code-action` is the same harness as `-p` plus GitHub
App glue.

- **E1, live-coding prompt:** plain `.github/workflows/ci.yml`: `npm ci`, Biome, Vitest,
  `next build`, Playwright e2e without the LLM spec. No AI in it.
- **E2, `claude -p` as an ordinary step:** the A2 gate on a runner (`--bare`,
  `--json-schema "$(cat demos/a2-findings.schema.json)"`, `jq -e` on `high`), report into
  `$GITHUB_STEP_SUMMARY`. Optionally the part C program the same way. Secret
  `ANTHROPIC_API_KEY` only.
- **E3, human in the loop with plain GitHub mechanics:** branch protection makes the E2
  job a required check, so a red review blocks the merge.
- **Links only, one sentence each:** `claude-code-action`, managed Code Review, routines.
- **One security point:** the runner loads the checkout's `CLAUDE.md`, skills, and hooks
  without a trust prompt; a PR that edits `.claude/settings.json` is code in your
  pipeline; `--bare` in CI is the answer.
- **No CD.** Deployment, rollback, and hosting are in Rainer's talk.

## Decisions

All from 2026-09-27.

1. **Students decide how to follow.** Experienced students follow live, others re-run the
   storybook later. Every demo must be reproducible from the storybook alone and proven on
   the class model (Opus 5.5, decision 9). Rainer can hand out keys.
2. **Part A keeps A1 to A5** plus the required A6 and A7. A4's security persona is safe.
3. **A7 walks the live app** with an ad-hoc Playwright script, including one chat turn.
4. **B1 question: identity per door** (text in part B).
5. **Parts A and B stay live**, about 75 minutes.
6. **B1 is a harness-vs-model matrix with three runs** (table in part B). Cost from
   OpenRouter.
7. **Demo assets live in the session 5 starter** under `demos/`.
8. **Students without a subscription source `demos/openrouter.env.sh`.**
9. **Dry runs use Opus 5.5** (`claude-opus-5-5`), not Opus 5.
10. **Starter:** app flattened to the repo root; no MCP config (`.mcp.json` and
    `.claude/settings.local.json` removed, stale mentions in `AGENTS.md` and `docs/mcp.md`
    fixed). Pushed by Rainer at `8c18441`; ask before every further push.
11. **Part C is one growing program, C1 to C6.**
12. **All part C code is pre-written and walked through**, no live prompts in part C.
13. **Jaeger** shows the OpenTelemetry traces.
14. **The starter may be changed or simplified** where that makes session 5 demos easier.
15. **The program is one CLI with subcommands.**
16. **C3 gate is a terminal y/n** with the diff; hook deny list `.env*`, `drizzle/`,
    `lib/auth*.ts`, `package.json`.
17. **C4's team-vs-single comparison is shown as dry-run numbers**; only the team runs
    live.
18. **Part E is minimal and "behind the scenes":** plain CI, `claude -p` as a pipeline
    step, branch protection as the gate, links for the managed products.
19. **CI uses an Anthropic API key from Rainer**, as repo secret `ANTHROPIC_API_KEY`.
20. **CI only, no CD.** Deployment, rollback, and hosting the finished agent are covered
    in Rainer's talk.
21. **Enough material for the day; detail agenda items may be dropped.** ACP (part D) is
    dropped. Evals, the higher orchestration layer, and OpenTelemetry for the Mastra app
    agent get no demo; B1's grading is the evals example, C6 the observability example.

## Work plan (next phase, in order)

1. **Clone the starter** (`gh repo clone rstropek/2026-claude-classroom-5-starter`), then
   `npm install` and a `.env` from `.env.example` (OpenRouter key from Rainer).
2. **Part A:** write the prompts for A0 to A7 and B1 and `demos/review-rubric.md`; dry-run
   each on Opus 5.5 in a `-work` copy; log runtime and cost; fix prompts that exceed the
   time budget. Verify the `Edit(./reports/**)` rule syntax (A6) and pin pi's thinking level
   (B1).
3. **Part C:** write `agents/` (C1 to C6) into the starter, one commit per step, plus
   CodeTours; probe the SDK through OpenRouter in C1; verify the MCP SDK v1/v2 dependency
   split; measure C4 team vs. single agent on Opus 5.5 and keep the numbers for the
   storybook; add a Jaeger `docker run` line to `demos/`.
4. **Part E:** write the E1 prompt and the E2 job; test on a GitHub repo with Rainer's
   `ANTHROPIC_API_KEY` secret (a fork or a throwaway repo; ask before pushing anything).
5. **Write `storybook-05.md`** (steps from 25, writing-guide skill and audit greps,
   appendix A headless recipe with `claude-opus-5-5`, appendix B demo insurance), update
   the README session table ("follows" for the result folder).
6. **Double-check before class:** the Codex SDK analogy; model ids on the day; OpenRouter
   prices for glm-5.3-flash and kimi-k3.
