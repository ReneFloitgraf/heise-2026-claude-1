# Session 5 Storybook

**Classroom: Agentische Entwicklung mit Claude Code, Mastra & CopilotKit, Session 5**

## Live-coding script for session 5

This is the script for session 5. Each step gives you a **Goal**, the commands or the
**Prompt** for the step, **Teaching points** to narrate while the agent works, and a
**Verify** checklist. The steps keep counting from session 4, so today starts at step 25.
Appendix A has the recipe for rehearsing every step headless, and appendix B lists what
can go wrong in front of the room.

Today is different from the first four sessions. Until now you sat in front of Claude
Code's window and typed prompts. Today the window goes away. You run the same agent from
the shell with `claude -p`, from a TypeScript program with the Agent SDK, and from a
GitHub Actions runner. Most steps are commands you type, not prompts. Steps 29 to 31 and
36 to 37 hand the agent a real task, and steps 32 to 35 walk through a program that is
already in the starter.

Every run in this storybook costs money or quota. The durations and dollar figures are
for Claude Opus 5.5 (`claude-opus-5-5`), priced at list price. On a Claude subscription
they count against your plan's usage limits. On an API key or through OpenRouter they
are what you pay. The whole day comes to roughly 16 USD per person at those prices, and
about a third of that is two steps, 30 and 34.

## Where we start

Session 4 ended with **ai-tutor** after step 24. Bartholomew keeps a to-do list that has
five ways in: the chat, a REST API, the `ai-tutor` CLI, the CLI's stdio MCP server, and
an MCP server over Streamable HTTP. The chat also draws A2UI cards, and the HTTP MCP
server ships an MCP App. That code is the starter for today:

```
https://github.com/rstropek/2026-claude-classroom-5-starter
```

The starter adds three things to the session 4 result:

- `demos/` holds the JSON schemas, the jq filter, and the scripts the commands below use,
  so nobody types a schema from a PDF. `demos/README.md` lists them.
- `agents/` is a new npm workspace with `tutor-maintainer`, a TypeScript program built
  on the Agent SDK. Steps 32 to 35 walk through it, and `.tours/agent-sdk.tour` is its
  CodeTour.
- The MCP configuration from session 4 is gone, because MCP is not a topic today.

## What we build today

Nothing in the app changes today, apart from one security fix in step 34. What you build
is the machinery around the app: shell one-liners that ask the agent questions, a report
and a test protocol written by the agent, a program that scripts the agent, and a CI
pipeline where the agent reviews every pull request.

All of it drives the same binary. The svgbob source is in `images/headless-harness.bob`
and the render in `images/headless-harness.svg`:

```
+--------------------+    +-----------------------+    +--------------------------+
| claude             |    | "claude -p"           |    | "query() in a program"   |
| "(the TUI)"        |    | "steps 26 to 31, 37"  |    | "steps 32 to 35"         |
+---------+----------+    +-----------+-----------+    +------------+-------------+
          |                           |                             |
          | keyboard                  | "flags, stdin"              | "options, hooks"
          v                           v                             v
+-----------------------------------------------------------------------------------+
| "Claude Code binary: agent loop, tools, permissions, hooks, sessions, skills"     |
+----------------------------------------+------------------------------------------+
                                         |
                                         | "Anthropic Messages API"
                                         v
                  +----------------------------------------------+
                  | "Model endpoint: Anthropic or OpenRouter"    |
                  +----------------------------------------------+
```

## What we teach today

1. **`claude -p` is the whole agent without the window.** It loads the same `AGENTS.md`,
   skills, and settings as the TUI, and it asks nobody before it does. What loads costs
   tokens on every call.
2. **A JSON schema turns the agent into a function.** `--json-schema` gives you data a
   script can test, and `jq -e` turns that data into an exit code.
3. **Permissions are flags.** `--tools` decides what exists, `--allowedTools` what runs
   without asking, and `--permission-mode dontAsk` denies the rest. The prompt and the
   allowlist have to agree, or the agent stops.
4. **Harness and model are separate choices.** The same model scores differently in two
   harnesses, and you find out why by reading the event streams.
5. **The Agent SDK is Claude Code as a library.** `query()` spawns the same binary. What
   your code adds are callbacks: custom tools, hooks, a permission function, and a
   rewind.
6. **Subagents buy separate contexts, not speed.** A review team costs more and takes
   longer than one agent, and it finds a vulnerability the single agent misses.
7. **In CI, the agent is a step like any other.** It reads untrusted input, so it runs
   with `--bare`, read-only tools, a budget, and a schema, and branch protection turns
   its verdict into a merge gate.

---

## Step 25: setup and two ways to pay

**Goal:** everybody has the starter running from a fork, knows how their Claude Code is
billed, and has the long-running protocol job from step 30 started in a second terminal.

### Fork, clone, run

```bash
gh repo fork rstropek/2026-claude-classroom-5-starter --clone
cd 2026-claude-classroom-5-starter
gh repo set-default            # pick your fork
npm install
cp .env.example .env           # then fill in OPENROUTER_API_KEY and BETTER_AUTH_SECRET
npm run db:migrate
npm run dev
```

Sign up at <http://localhost:3000/signup> and ask Bartholomew to put one thing on the
list. Step 34 needs a chat to exist.

`npm install` prints a block of `allow-scripts` warnings about `esbuild` and
`@scarf/scarf`. Recent npm versions no longer run install scripts of dependencies unless
you allow them. Nothing today needs those scripts, and the workspace `prepare` script
that builds the CLI still runs.

Step 31 needs the pi coding agent, and step 35 needs Docker:

```bash
npm install -g @earendil-works/pi-coding-agent
pi --version                   # 0.87.1 or later
docker --version
```

### Claude Code through OpenRouter

Every command today works with a Claude subscription. If you don't have one, route
Claude Code through OpenRouter for the current shell:

```bash
source demos/openrouter.env.sh
```

The script reads `OPENROUTER_API_KEY` from `.env` and exports three variables:
`ANTHROPIC_BASE_URL=https://openrouter.ai/api`, `ANTHROPIC_AUTH_TOKEN` with your key, and
an empty `ANTHROPIC_API_KEY`. It never prints the key. `source demos/openrouter.env.sh
--off` undoes it, and other terminals are not affected.

Claude Code prints two notices in that mode: claude.ai connectors are disabled because
another auth source is set, and auto-mode classifier requests are billed through
OpenRouter too. Both are expected.

**Teaching points**

- **The harness is separate from the endpoint.** OpenRouter speaks the Anthropic
  Messages API, so Claude Code does not notice the difference. The base URL has no
  `/v1`, and `ANTHROPIC_API_KEY` has to be empty, or Claude Code authenticates against
  Anthropic instead.
- **Model ids stay as they are.** `claude-opus-5-5` works through OpenRouter without an
  `anthropic/` prefix. Non-Claude models work too, which step 31 uses.

### Start step 30 now

Step 30 walks through the app in a browser and takes about eight minutes. Open a second
terminal in the repository and start the command from step 30 there. It keeps running
while steps 26 to 29 happen.

**Verify:** the app runs at <http://localhost:3000> from your fork, your account has one
to-do, `pi --version` prints a version, and step 30 is running in a second terminal.

## Step 26: the same agent without a window

**Goal:** you have piped data into `claude -p` and seen everything it loads before it
answers.

### A question and a pipe

```bash
claude -p "What does this app do? Five bullets."
```

One turn, about five seconds. The answer comes from `AGENTS.md`, which `-p` loads just
like the TUI.

Now pipe something in. `npm audit` reports 13 advisories for the starter, one of them
high. Ask the agent which of them matter:

```bash
npm audit --json | claude -p --tools "Read,Grep,Glob" \
  "This is npm audit output for this repository. Which of these advisories can an attacker actually reach in this app, and which only affect build or dev tooling? Five lines at most."
```

Expect about 30 seconds and nine turns. The agent reads the audit JSON from stdin, then
opens the lockfile and the code to find out who imports what. Expect it to say that the
`undici` advisory, the only high one, sits in provider adapters the app never calls, and
that the `esbuild` advisory only affects `drizzle-kit` at development time.

### What loads before the first token

```bash
claude -p "hi" --output-format stream-json --verbose |
  jq 'select(.subtype == "init") | {model, apiKeySource, skills, mcp_servers: [.mcp_servers[].name], tools: (.tools | length)}'
```

The first event of every run is `system/init`. It lists the project skills from
`.claude/skills/` next to your own user skills, every MCP server, and the number of
tools. If your claude.ai account has connectors, such as mail, calendar, or a wiki, they
are in `mcp_servers` too, in a repository that has nothing to do with them.

Compare the same question in safe mode:

```bash
claude -p --output-format json "Which file defines the Mastra agent, and which tools does it register? Two sentences." |
  jq '{result, num_turns, total_cost_usd}'
claude -p --safe-mode --output-format json "Which file defines the Mastra agent, and which tools does it register? Two sentences." |
  jq '{result, num_turns, total_cost_usd}'
```

Expect the same answer twice. The normal run takes about three turns and 0.18 USD, and
the safe-mode run about four turns and 0.08 USD. `--safe-mode` switches off `CLAUDE.md`,
skills, hooks, and MCP servers, so it has to search a little more, and every call is
cheaper because no connector schemas travel with it.

**Teaching points**

- **`-p` trusts the checkout.** Without further flags it loads `CLAUDE.md` and
  `AGENTS.md`, skills, hooks from `.claude/settings.json`, and MCP servers from
  `.mcp.json`, with no trust dialog and no per-server approval. Running `claude -p` in a
  repository you just cloned runs that repository's hooks. Step 37 comes back to this.
- **Loaded context is a cost on every call.** Tool schemas and instructions go out with
  every request. For a small question, the connectors of the person who runs it cost more
  than the project memory saves.
- **`--bare` is the switch for CI, and it needs an API key.** It skips hooks, `CLAUDE.md`,
  plugins, and auto-memory, and it never reads the claude.ai login. With only a
  subscription, `claude -p --bare "hi"` answers `Not logged in`. With
  `ANTHROPIC_API_KEY` or through OpenRouter it works. `--safe-mode` is the variant that
  keeps your login.
- **The prompt is the last argument, and stdin is data.** Claude Code accepts up to
  10 MB on stdin. It waits three seconds for input before it starts without, which is
  why a backgrounded `claude -p` warns about stdin unless you redirect `</dev/null`.

**Verify:** you have seen the init event with your own skills and servers, and you know
whether `--bare` works with your login.

## Step 27: the agent as a typed function

**Goal:** the agent returns JSON that matches a schema, a jq expression turns a review
into an exit code, and a live stream shows each tool call as it happens.

### A schema for the answer

`demos/a2-doors.schema.json` describes an inventory of the ways into the to-do list:
one entry per way, with its entry file, its authentication, and its operations.

```bash
claude -p --tools "Read,Grep,Glob" --output-format json \
  --json-schema "$(cat demos/a2-doors.schema.json)" \
  "List the ways into the todo list: the chat, the REST API, the CLI, the stdio MCP server, and the HTTP MCP server. One entry per way, with its entry file, how the caller is authenticated, and which todo operations it offers." |
  jq -r '.structured_output.doors[] | [.door, .entryFile, .auth] | @tsv' | column -t -s $'\t'
```

Expect about 25 seconds, 16 turns, and 0.70 USD, then a five-row table in the terminal.
The prompt names the five ways on purpose. Without that, a model is free to split the
REST API into three rows or count the AG-UI bridge as its own door, and the schema can't
stop it.

### A review as an exit code

`demos/a2-findings.schema.json` describes review findings, each with a severity, a
file, a line, a title, and a fix.

```bash
mkdir -p reports
claude -p --tools "Read,Grep,Glob" --output-format json \
  --json-schema "$(cat demos/a2-findings.schema.json)" \
  "Review app/api/todos and lib/api-route.ts for security problems. Report only what you can point at in the code." \
  > reports/findings.json
jq -r '.structured_output.findings[] | [.severity, "\(.file):\(.line)", .title] | @tsv' reports/findings.json | column -t -s $'\t'
```

Expect about 30 seconds and a short list, typically one medium finding about the missing
length limit on titles and one or two low ones. Now make it a gate:

```bash
jq -e '[.structured_output.findings[] | select(.severity == "high")] | length == 0' reports/findings.json; echo "exit $?"
jq -e '[.structured_output.findings[] | select(.severity != "low")] | length == 0' reports/findings.json; echo "exit $?"
```

The first gate passes with exit 0. The second one fails as soon as there is one medium
finding. `jq -e` sets the exit code from the last value, so a shell script or a CI runner can
act on it. Step 37 runs this gate on every pull request.

### A flight recorder

```bash
claude -p --tools "Read,Grep,Glob" --output-format stream-json --verbose \
  "Where is a todo's owner checked before it is marked done? Follow every way into the todo list. Five lines at most." |
  jq -r -f demos/a3-tool-calls.jq
```

One line per tool call scrolls by while the agent works, and a summary line closes the
run, typically six turns, 15 seconds, and 0.55 USD. `demos/a3-tool-calls.jq` is 14 lines
long, so open it. It picks `tool_use` blocks out of `assistant` events and indents calls
that carry a `parent_tool_use_id`, which marks a subagent.

Add a budget and run it once more:

```bash
claude -p --tools "Read,Grep,Glob" --max-budget-usd 0.05 --output-format stream-json --verbose \
  "Where is a todo's owner checked before it is marked done? Follow every way into the todo list. Five lines at most." |
  jq -r -f demos/a3-tool-calls.jq
```

The run stops after the first call with `error_max_budget_usd`, and the summary line
shows about 0.45 USD. That is not a bug in the cap. The budget is checked between model
calls, and the first call of a session on Opus writes the prompt cache for everything
that loaded, which alone costs more than five cents.

**Teaching points**

- **The schema is validated after the agent is done.** The agent works in as many turns
  as it needs, and its final answer has to match the schema. An invalid schema is a hard
  error before the first call.
- **`jq -e` is the whole gate.** A schema, a jq filter, and an exit code are all it
  takes. Tightening the gate from `high` to `!= "low"` is a policy change, and it lives in
  one line you can review.
- **stream-json is the protocol, not a debug format.** The same events arrive in the
  Agent SDK in step 32 and in pi's JSON mode in step 31. Anything you build on them works
  across all three.
- **A budget is a circuit breaker, not a price tag.** `--max-budget-usd` counts subagents
  too and ends the run cleanly. It cannot split a model call, so set it with a margin, and
  add `--max-turns` when the number of steps is what worries you.

**Verify:** the door table prints, the first gate exits 0 and the second exits 1 on your
findings, and you have seen the budget stop a run.

## Step 28: change the prompt, fork the session

**Goal:** the same question answered in three voices, and one explored session reused
for two follow-up questions.

### Three system prompts, one question

```bash
Q="Is there anything risky in app/api/todos? Keep it under 200 words."
claude -p --tools "Read,Grep,Glob" "$Q"
claude -p --tools "Read,Grep,Glob" \
  --append-system-prompt "You are a security engineer reviewing this codebase before a production release. Report vulnerabilities with severity, file:line, and a fix." "$Q"
claude -p --safe-mode --tools "" \
  --system-prompt "You are Bartholomew, a butler who keeps his employer's to-do list. You answer briefly, politely, and a little old-fashioned." "$Q"
```

The first two take about 17 seconds each. Expect the same facts in both: authentication
and per-user scoping hold, titles have no length limit, and writes have no rate limit.
The shape differs. The default run answers in prose with a "what looks right" list, and
the security persona answers with a severity table. The persona changes how the answer
looks far more than what the agent finds.

The third run takes seven seconds and about a cent. `--system-prompt` replaces Claude
Code's system prompt entirely, `--tools ""` removes the built-in tools, and
`--safe-mode` keeps skills and MCP servers out. What is left is a plain chat model with
the app's own persona, and Bartholomew politely admits that he cannot open any files.

Run the third command once without `--safe-mode`. Expect the butler to offer Outlook,
Teams, or whatever your claude.ai connectors provide, and to quote `AGENTS.md`. `--tools
""` only removes built-in tools, and project memory loads regardless of the system
prompt.

### Explore once, fork twice

```bash
SID=$(claude -p --tools "Read,Grep,Glob" --output-format json \
  "Explore how a todo travels from each way in to the database. Read what you need, then answer in one sentence." |
  jq -r '.session_id')
claude -p --resume "$SID" --fork-session --tools "Read,Grep,Glob" --output-format json \
  "Write an onboarding note for a developer who joins tomorrow: eight bullets, each naming a file." |
  jq -r '.result, .total_cost_usd, .num_turns'
claude -p --resume "$SID" --fork-session --tools "Read,Grep,Glob" --output-format json \
  "List the five biggest test gaps on that path, one line each, naming the test file a test belongs in." |
  jq -r '.result, .total_cost_usd, .num_turns'
```

The exploration takes about ten seconds and 0.56 USD. The onboarding note then comes
back in two turns and about ten seconds, because the fork starts with everything the
first run read. A fresh run that explores and writes the note in one go takes about 18
turns and 37 seconds.

Read `total_cost_usd` carefully. A resumed or forked session starts from the total its
transcript saved, so the fork reports about 0.62 USD, and 0.56 of that is the parent. The
fork's own cost is the difference, about five cents.

**Teaching points**

- **`--append-system-prompt` adds, `--system-prompt` replaces.** Appending keeps Claude
  Code's tool instructions and safety rules. Replacing turns the harness into something
  else, which is exactly right for a persona and wrong for a coding task.
- **A fork is a branch of the conversation.** `--fork-session` gives the resumed run a
  new session id, so the two follow-ups don't see each other and the explored session
  stays untouched for a third question.
- **Cost fields are cumulative across a resume.** Subtract the parent's total, or you
  bill the exploration twice in every report you build on these numbers.

**Verify:** the three answers differ in shape, the butler admits he has no tools, and
the fork answers in a few turns.

## Step 29: a code review as a Markdown report

**Goal:** one command produces a review report with a diagram, a findings table with
clickable locations, and test gaps, and the agent can write that one file and nothing
else.

`demos/review-rubric.md` says what the review looks at, what each severity means, and
how the report is laid out. It goes in as an appended system prompt, so the rubric is a
versioned file and not a paragraph somebody retypes.

```bash
claude -p "Review the repository and write the report to reports/code-review.md." \
  --append-system-prompt-file demos/review-rubric.md \
  --permission-mode dontAsk \
  --allowedTools "Read,Grep,Glob,Edit(./reports/**)" \
  --output-format stream-json --verbose |
  jq -r -f demos/a3-tool-calls.jq
```

Expect about two minutes, 40 turns, and 1 USD. The flight recorder shows about 30 reads
across all five ways in, a handful of greps, and one `Write reports/code-review.md` at
the end.

Open `reports/code-review.md` in VS Code and switch to the Markdown preview. The Mermaid
diagram shows the five ways in and where each one establishes identity. Click a location
in the findings table, and VS Code jumps to the line. Expect a verdict of "ready with
fixes" and two medium findings: titles have no length limit, and the chat agent's
`addTodo` tool checks `min(1)` before it trims, so a title of only spaces is stored as
an empty to-do. Step 33 fixes the second one.

To see what the permission mode refused, rerun with `--output-format json` instead of
the stream and read `.permission_denials`. Expect one entry: a `Bash` call with `cat -n`
over a list of files. The agent wanted line numbers the fast way. After the denial it read the
files one by one.

**Teaching points**

- **`Edit(./reports/**)` covers creating files too.** Edit rules apply to every built-in
  tool that writes files, so the one rule lets the agent create the report and nothing
  else. A path rule is the smallest write permission there is.
- **`dontAsk` turns every unlisted call into a denial.** In `-p` mode nobody can answer a
  permission prompt. Without `dontAsk`, a call outside the allowlist waits for an answer
  that never comes.
- **The rubric is the reusable part.** Swap the prompt and the same rubric reviews a
  pull request. Swap the rubric and the same command writes an architecture review. Keep
  it next to the code, reviewed like code.
- **One agent in one pass has blind spots.** The report is good, and it misses the
  vulnerability that step 34's review team finds. Remember the verdict "ready with
  fixes".

**Verify:** `reports/code-review.md` exists with a Mermaid diagram, a findings table
with working links, and test gaps, and `git status` shows no other change.

## Step 30: a test protocol with annotated screenshots

**Goal:** the agent drives the running app through four user journeys and writes a test
protocol with screenshots that it annotated and checked itself.

The dev server from step 25 has to run on port 3000. Start this in a second terminal
during step 25, because it takes about eight minutes:

```bash
claude -p "$(cat <<'EOF'
Write a test protocol with annotated screenshots for the app running at http://localhost:3000. Four journeys: sign up a new account, ask Bartholomew in the chat to add a todo and see it appear in the sidebar, open /projects/new, and sign out. Drive the browser with an ad-hoc Node.js script that uses the repo's Playwright. After each step, save a screenshot and the bounding boxes of the elements the step is about, as JSON read from the DOM. Then write an ad-hoc Python script that draws numbered boxes, arrows, and short labels onto copies of the screenshots. Open every annotated image yourself and fix annotations that point at the wrong place. Write test-protocol/PROTOCOL.md with one row per step: action, expected, actual, pass or fail, and the annotated image. Everything goes under test-protocol/, and nothing else in the repository changes. Your shell runs exactly two kinds of commands, so use no others: node test-protocol/<script>.mjs, and uv run --with pillow python test-protocol/<script>.py. The chat turn spends a fraction of a cent of OpenRouter credit, which is fine, but run the chat journey only once.
EOF
)" \
  --permission-mode dontAsk \
  --allowedTools "Read,Grep,Glob,Write(./test-protocol/**),Edit(./test-protocol/**),Bash(node test-protocol/*),Bash(uv run --with pillow python test-protocol/*)" \
  --output-format stream-json --verbose |
  jq -r -f demos/a3-tool-calls.jq
```

Expect about 110 turns, eight to nine minutes, and 3 USD. Most of the turns are the
last phase. The agent opens each annotated PNG, which works because the Read tool is
multimodal, and redraws until the labels sit right.

When it is done, open `test-protocol/PROTOCOL.md` in the Markdown preview. Expect 13
steps in four journeys, all passing, each with its annotated screenshot and numbered
boxes that the "actual" column refers to. Open the screenshot of the chat step. The
boxes sit on the tool-call row, the reply, the new sidebar item, and the counter, and the
counter's box covers the CopilotKit dev inspector's button, because that button really
does sit on top of it in development.

**Teaching points**

- **The prompt and the allowlist must name the same commands.** The allowlist permits
  `node test-protocol/*` and `uv run --with pillow python test-protocol/*`, and the
  prompt tells the agent that these are the only two. Leave that sentence out, and the
  agent tries `SKIP_CHAT=1 node ...` or `uv --version`, gets denied, writes its scripts,
  and stops with an honest report of what it could not run, without inventing a
  protocol.
- **Throwaway code is cheap now.** The agent writes a Playwright script and an image
  annotator for one run. Nobody reviews them line by line, because the protocol and the
  images are the product, and you can check those with your eyes.
- **`uv run --with pillow` is a dependency without an install.** Pillow is not a project
  dependency and not installed globally. `uv` resolves it into a cached environment for
  that one command.
- **Coordinates come from the DOM, not from the model.** The agent reads bounding boxes
  with `getBoundingClientRect()` and draws with them. Asking a model to guess pixel
  positions in a screenshot is how you get arrows that point at nothing.
- **The agent checks its own drawings.** The Read tool returns images to the model, so the
  agent sees a label that covers the element it describes and moves it. That loop is the
  expensive part of the run, and it is also why the images are right.

**Verify:** `test-protocol/PROTOCOL.md` lists 13 steps with images, the annotations
point at the right elements, the sidebar of your account shows the to-do the protocol
added, and `git status` shows nothing outside `test-protocol/`, which `.gitignore`
ignores.

## Step 31: same model, two harnesses

**Goal:** Claude Code and the pi coding agent answer the same research question on the
same open-weight model, pi answers it once more on a second model, and Claude Code
measures and grades all three.

pi is a minimal open-source coding agent. It has its own agent loop and tools, and a
JSON event stream of its own. Like Claude Code, it loads `AGENTS.md`, so the question has to
need facts that `AGENTS.md` does not give away:

```text
For each of the ways into the todo list, show where the user's identity is established
and where the query is scoped to that user, with file and line.
```

`demos/b1-bakeoff.sh` asks it three times in parallel and writes each event stream to
`bakeoff/`:

| Contestant | Harness | Model | Key flags |
|---|---|---|---|
| `claude-code-glm` | Claude Code | `z-ai/glm-5.3-flash` | `--tools "Read,Grep,Glob" --effort medium`, through OpenRouter |
| `pi-glm` | pi | `openrouter/z-ai/glm-5.3-flash` | `--tools read,grep,find,ls --thinking medium --no-extensions` |
| `pi-kimi` | pi | `openrouter/moonshotai/kimi-k3` | same as `pi-glm` |

Open the script before you run it. It is 34 lines, and step 31's teaching points come back to several of them.

Start `claude` in the repository and give it this prompt. This step runs in the TUI, so
the room sees the agent reason about three event streams:

> **Prompt 31.1**
>
> Run a bake-off with bash demos/b1-bakeoff.sh. It asks three contestants the same
> question about this repository and writes each event stream to bakeoff/: Claude Code
> on glm-5.3-flash, pi on glm-5.3-flash, and pi on kimi-k3, all through OpenRouter. From
> each JSONL file, compute with jq: turns, tool calls by name, input and output tokens,
> and cost. Take the seconds from the script's output. Claude Code prices non-Claude
> models wrong, so price its tokens with the per-token rates pi reports for the same
> model. Then check every file-and-line claim in each final answer against the code, and
> grade each way into the todo list per contestant as correct, wrong, or missing. Show
> one table with a row per contestant, then a verdict in three sentences: what the
> harness changed, what the model changed. Write nothing outside bakeoff/.

Expect about four minutes and 0.90 USD for the grader, and a few cents of OpenRouter
credit for the contestants. The script finishes in one to two minutes. Claude Code on
glm is fastest at 15 to 20 seconds, and pi on kimi takes about a minute.

Read the table with the room. Expect Claude Code on glm to get all five ways in right,
pi on glm to get most of the line numbers wrong, and pi on kimi to get them right again
at more than ten times the price of glm. Then read the verdict, because it names the
mechanism. Claude Code's `Read` tool returns every line with its line number, and pi's
`read` returns bare text. On glm under pi, the line numbers are only right where a grep
happened to print them. kimi runs more greps on its own and pays for them in turns.

Expect one correction in the cost column. Claude Code reports about 0.17 USD for its glm
run, because it doesn't know the model and falls back to a Claude price. The grader
reprices the tokens with pi's rates, which gives less than a cent.

**Teaching points**

- **The harness is half the result.** Same model, same question, same tools on paper,
  and the scores differ because of what one tool puts into the model's context. When you
  compare models, hold the harness fixed. When you compare harnesses, hold the model
  fixed.
- **Event streams make harnesses comparable.** Claude Code's stream-json and pi's
  `--mode json` look different, and both carry turns, tool calls, and token usage per
  message. jq is enough to put them into one table.
- **Claude grading other models is a model-graded eval.** The grader checks every claim
  against the code and writes down how it counted. Read the counting rules before you
  trust the table. The grader is a model too, and it can be wrong in the same ways.
- **pi reads stdin until it ends.** A backgrounded `pi -p` without `</dev/null` waits
  forever for input that never comes. That is why every contestant in the script gets
  `</dev/null`, and why a hanging harness is the first thing to check when a background
  job never finishes.
- **Pin what you compare.** The script pins the thinking level in both harnesses and
  switches off pi's extensions, so your installed packages don't change the result.

**Verify:** three JSONL files in `bakeoff/`, a table with turns, tool calls, tokens,
cost, and grades per contestant, and a verdict that names a mechanism.

---

## Step 32: the Agent SDK is claude -p as a library

**Goal:** you have run the A2 question from step 27 through `query()`, seen the same
event types, and added a custom tool with typed output.

### What the Agent SDK is

`@anthropic-ai/claude-agent-sdk` is not a generic agent framework. `query()` spawns the
Claude Code binary that ships with the package as a subprocess and talks stream-json to
it over stdio. You get Claude Code's agent loop, its built-in tools, its permission
system, its sessions, and its `CLAUDE.md` loading, and your code steers it. In this
classroom, Mastra is the generic framework that runs Bartholomew inside the app. The
Agent SDK is the coding agent you script.

The binary comes through npm optional dependencies, one package per platform. `npm ci
--omit=optional` breaks it.

### Walk the tour

Open `.tours/agent-sdk.tour` from the CodeTour view: "Agent SDK: the tutor-maintainer
program". It has 17 steps in sections C1 to C6, one per subcommand. Walk the C1 and C2
sections now, and the others in the steps that use them.

`agents/` is its own npm workspace with `tutor-maintainer` in it. `npm run maintainer --
<subcommand>` runs `agents/src/index.ts` with `tsx`. Every `query()` in it runs with the
repository root as `cwd`, so the spawned Claude Code loads the app's `AGENTS.md` and
skills.

### C1: hello

```bash
npm run maintainer -- hello
```

The program prints its own process id and a `pgrep` line. Run that line in a second
terminal while the agent works, and you see a `claude` child process. Every event prints
as its type: `system/init` first, then `assistant` and `user` turns, `result` last. The
init line shows `apiKeySource none`, which means the subprocess uses your claude.ai
login. Expect about 17 seconds, 13 turns, and 0.20 USD, and a table of the five ways in.

That is a third cheaper than the same question with `claude -p` in step 27, and the
reason is one option in `agents/src/shared.ts`. `strictMcpConfig: true` loads only the
MCP servers the program passes, so your claude.ai connectors stay out.

The program also works through OpenRouter. `source demos/openrouter.env.sh`, then
`MAINTAINER_MODEL=claude-haiku-4-5 npm run maintainer -- hello`. The SDK passes the
environment to the subprocess, and Haiku answers correctly in about 20 turns.

### C2: a custom tool and typed output

```bash
npm run maintainer -- tests
```

The agent has Read, Grep, and Glob, no Bash, and one tool of yours: `run_tests`, defined
in `agents/src/run-tests.ts`. It runs Vitest with the JSON reporter and returns the
counts and the failing tests per file. The agent runs the suite through it, reads the
test files, and answers in the shape of a zod schema: per way in, the test files that
cover it and up to three behaviors no test pins.

Expect about 70 seconds, 27 turns, and 0.50 USD. The output lists 54 passing tests and
then the gaps per way in. Expect the REST API and the HTTP MCP server to show "scoping to
the caller" as untested, which is worth remembering for step 37.

**Teaching points**

- **The options are the flags.** `tools`, `allowedTools`, `permissionMode`,
  `outputFormat`, `persistSession`, and `model` map one to one onto the flags from steps
  26 to 29. Anything you tried with `claude -p` moves into a program without a
  translation step.
- **A custom tool is an in-process MCP server.** `tool()` takes a name, a description,
  a zod shape, and a handler. `createSdkMcpServer()` wraps it, and the subprocess calls
  it over the SDK's control channel, without a port. Claude Code names it
  `mcp__maintainer__run_tests`.
- **The tool is the policy.** Without Bash, `run_tests` is the only thing this agent can
  execute. That is a tighter boundary than any allowlist of shell commands.
- **zod writes the schema, with one trap.** `z.toJSONSchema()` emits a `$schema` for draft
  2020-12, and the CLI rejects it with "no schema with key or ref". Draft 7 passes, so
  `jsonSchema()` in `shared.ts` asks for that. The program parses `structured_output`
  with the same zod schema and gets typed data.
- **`options.env` replaces the environment.** In TypeScript, whatever you pass as `env`
  is the subprocess's whole environment. Spread `process.env` into it, or the child has
  no `PATH` and no login.
- **Your own scripts may use your subscription, products may not.** Anthropic does not
  allow third-party developers to offer claude.ai login or its rate limits for their
  products, including agents built on the Agent SDK. Anything you host or hand to other
  people uses an API key.

**Verify:** `hello` prints the event types, `apiKeySource`, and the table, `pgrep` shows
the `claude` child, and `tests` prints typed gaps per way in.

## Step 33: fix proposals with a human in the loop

**Goal:** the agent proposes a fix, you approve every edit in the terminal with the diff
in front of you, a hook keeps the agent away from protected files, and a rewind undoes
everything the agent changed.

Walk the C3 section of the tour first. Three layers decide about every edit:

1. A `PreToolUse` hook in `agents/src/guard.ts` denies writes to `.env*`, `drizzle/`,
   `lib/auth*.ts`, and every `package.json`. It runs before anybody is asked.
2. `canUseTool` in `agents/src/gate.ts` prints the edit as a red and green diff and asks
   `Allow this change? [y/N]`.
3. After the run, the program runs the whole suite itself. A red suite, or a no to `Keep
   the changes? [Y/n]`, rewinds every file to its state before the first edit.

### A fix, and a rewind

Step 29's report found that the chat agent's `addTodo` tool accepts a title of only
spaces:

```bash
npm run maintainer -- fix "The chat agent's addTodo tool in lib/todo-tools.ts accepts a title of only spaces and stores an empty to-do, because it validates with its own z.string().min(1) instead of CreateTodoRequest from the contract."
```

Expect three questions within about 20 seconds: one edit that replaces the inline schema
with `CreateTodoRequest`, one that adds the import, and one that adds a test for a title
of three spaces. Answer `y` to each. The agent runs the affected test file through
`run_tests`, then the program runs the whole suite (55 tests now) and asks `Keep the
changes? [Y/n]`.

Look at `git diff` in a second terminal. The fix is right. Answer `n` anyway. The last
line reads `Declined: rewound lib/todo-tools.ts, tests/unit/todo-tools.test.ts (+4 -11)`
or close to it, and `git status` is clean. The program resumed the session with an empty
prompt and called `rewindFiles()` on the first checkpoint, once as a dry run to report
what would change, and once for real. Step 35 fixes this finding for good.

The first line of output is a warning, `CLAUDE_SDK_CAN_USE_TOOL_SHADOWED`. Read and
`run_tests` are in `allowedTools`, so `canUseTool` never sees them, and the SDK says so
at startup. That is the intended setup: reads are free, writes are gated.

### A fix the hook refuses

```bash
npm run maintainer -- fix "Sessions use Better Auth's default lifetime of seven days. Make them expire after one day."
```

The right change is one line in `lib/auth-config.ts`, and the hook denies it. Expect a
line `[hook] denied Edit on lib/auth-config.ts: it is the authentication setup`, maybe a
test edit or two that you approve and the agent later takes back, and a final answer
like: "I couldn't fix this: the change belongs in `lib/auth-config.ts`, but a hook
blocks edits to that file, and moving the setting into another file would just get
around that." Expect about 30 seconds and 0.17 USD.

**Teaching points**

- **A hook is policy, `canUseTool` is a person.** The hook runs first and denies without
  asking. A path on the list is off limits even for a human who would have typed y, and
  the reason goes back to the model as text it can act on.
- **The agent respects a denial it understands.** The hook's message says why the path
  is protected. With that, the agent explains the situation instead of hunting for a way
  around. A bare "denied" invites creativity.
- **Checkpoints cover Edit and Write, nothing else.** `enableFileCheckpointing` backs up
  each file before its first edit. A file changed through Bash or by a subagent is not
  in a checkpoint, which is one more reason to keep Bash away from a fixer.
- **The checkpoint ids come from the stream.** `extraArgs: { "replay-user-messages":
  null }` makes Claude Code echo the user messages with their UUIDs, and those are what
  `rewindFiles()` takes.
- **The program checks the agent's claim.** The agent says the tests pass. The program
  runs the suite itself before it asks you to keep anything.

**Verify:** every edit showed its diff before it happened, the declined run left a clean
tree, and the auth run ended with an explanation instead of an edit.

## Step 34: a review team finds what one agent missed

**Goal:** four subagents review the repository in parallel, their findings and costs
come back per reviewer, one of them is a high-severity vulnerability that nobody has
noticed since session 1, and you reproduce it and fix it.

Walk the C4 section of the tour. `agents/src/review.ts` defines four reviewers:
architecture and tests on Sonnet 5, security on Opus 5.5, docs on Haiku 4.5, all with
read-only tools. The orchestrating agent gets the `Agent` tool and the instruction to
start all four in one message.

```bash
npm run maintainer -- review
```

This takes about five minutes and 1.60 to 2.00 USD. While it runs, the flight recorder
shows four indented streams of tool calls, one per subagent. When it is done, the program
prints the cost per model and tokens, tool uses, and seconds per subagent, then about 14
findings sorted by severity. The first one reads like this:

```text
high   security     app/api/copilotkit/[...all]/route.ts:31  Any signed-in user can read other users' chats: the route checks only that a session exists, then passes every sub-path to CopilotKit ...
```

### The same review with one agent

`npm run maintainer -- review --single` gives the same four briefs to one agent. It takes
two to three minutes and about 1 USD, returns about 18 findings, and does not report the
chat leak. Don't run it live. The numbers for the table:

| | Team of four | One agent |
|---|---|---|
| Wall-clock | about 5.2 min | 2 to 2.7 min |
| Cost | 1.60 to 2.00 USD | 0.90 to 1.20 USD |
| Findings | about 14 | about 18 |
| Chat thread leak (high) | found | not found |

### Reproduce the leak

Make sure your own account from step 25 has chatted with Bartholomew at least once, then
run:

```bash
bash demos/c4-thread-leak.sh
```

The script signs up a fresh user and uses nothing but that user's cookie. It lists every
chat thread on the server through `GET /api/copilotkit/threads?agentId=tutor`, then
prints each thread's messages from `GET /api/copilotkit/threads/<id>/messages`. Your
own conversation with Bartholomew scrolls by, tool calls included, read by a stranger.

The cause is in two places. `app/api/copilotkit/[...all]/route.ts` checks that a session
exists and hands every sub-path to CopilotKit's runtime handler. The runtime's default
`InMemoryAgentRunner` keeps one thread store for the whole process, keyed by thread id
alone, and serves list, read, connect, stop, and clear for any id. The comment in the
route about `resourceId` is right about Mastra's memory, and Mastra's memory is not what
leaks.

### Fix it

```bash
npm run maintainer -- fix "Any signed-in user can read other users' chats. app/api/copilotkit/[...all]/route.ts only checks that a session exists, then hands every sub-path to CopilotKit, whose process-wide in-memory runner serves threads, threads/<id>/messages, agent/<id>/connect, agent/<id>/stop/<threadId>, and threads/clear for any thread id. The browser only needs its own thread, tutor:<userId>."
```

Expect about 70 seconds and 0.50 USD before the first question, most of it spent in
`node_modules/@copilotkit/runtime` finding out which sub-paths exist. Expect an allowlist
in the route: `info`, the run route, and `connect` and `stop` only for the caller's own
thread, with everything else answered as 404. Expect a test for the new rule, and
possibly an edit to `.tours/use-render-tool.tour`, whose line anchors moved. Approve,
keep, and run the repro again:

```bash
bash demos/c4-thread-leak.sh; echo "exit $?"
```

The script signs up its user and stops at the thread list with exit 22, which is curl's
code for an HTTP error. The chat still works. Check it in the browser, or run `npm run
test:e2e:llm`, which spends about a cent.

**Teaching points**

- **Subagents buy separate context windows, not speed.** The team takes longer than one
  agent, because the orchestrator waits for the slowest reviewer and then merges. Each
  reviewer starts with an empty context and one brief. The security reviewer spends its
  whole budget on identity and scoping, and that is where it follows the route into
  `node_modules`.
- **The SDK prices per model, not per subagent.** `modelUsage` gives dollars per model,
  and each `task_notification` gives tokens, tool uses, and duration per subagent. Giving
  each role its own model is also what makes the bill readable.
- **One finding justified the team.** The team costs about 0.80 USD more per run. The
  finding it adds is a cross-user data leak that four sessions of agents and reviews
  walked past. Price the review against what it finds, not against the other review.
- **Verify a high finding before you act on it.** The finding came from a model. The
  script is the evidence, and nobody should merge a security fix without a repro that
  goes red before and green after.
- **"Scoped by construction" was true for the wrong layer.** The route's comment
  explained why Mastra's memory cannot leak, and it was right. The leak is one layer up,
  in a store the app never configured. A default you did not choose is still yours.
- **Never deploy the results of sessions 1 to 4 as they are.** Every earlier result
  folder has this leak.

**Verify:** the team's report lists the thread leak as high, the repro script prints
another user's chat before the fix and exits 22 after it, `npm test` is green, and the
chat still works. Commit the fix.

## Step 35: a race in two worktrees, and traces

**Goal:** two agents on different models fix the same finding at the same time, each in
its own git worktree. The program keeps the cheapest fix whose tests pass. Then Jaeger
shows what an agent run looks like as a trace.

### C5: the race

Walk the C5 section of the tour. `AgentDefinition` has no worktree option, so
`agents/src/race.ts` runs `git worktree add` itself and gives each `query()` its own
`cwd`. The worktrees are siblings of your repository, with names ending in
`-race-sonnet` and `-race-haiku`.

The whitespace finding from step 33 is still open, because you rewound that fix. Race on
it:

```bash
npm run maintainer -- race "The chat agent's addTodo tool in lib/todo-tools.ts accepts a title of only spaces and stores an empty to-do, because it validates with its own z.string().min(1) instead of CreateTodoRequest from the contract."
```

Both fixers log their edits with a `[sonnet]` or `[haiku]` prefix. Expect under a minute,
two green fixers with nearly identical diffs, and a result like this:

```text
sonnet  claude-sonnet-5      46 s  0.286 USD  green  2 files changed, 12 insertions(+), 3 deletions(-)
haiku   claude-haiku-4-5     50 s  0.081 USD  green  2 files changed, 13 insertions(+), 4 deletions(-)

Kept haiku's fix on branch race/haiku-20260927125946.
```

Review the kept branch with the `git diff` line the program prints, then merge it or
delete it. Both worktrees are gone, and so is the losing branch.

### C6: traces in Jaeger

```bash
bash demos/jaeger.sh
npm run maintainer -- hello --otel
```

Open <http://localhost:16686>, pick the service `tutor-maintainer`, and open the newest
trace. Expect one `claude_code.interaction` span, about six `claude_code.llm_request`
spans, and one `claude_code.tool` span per tool call, each with two children:
`claude_code.tool.blocked_on_user` for the permission wait and
`claude_code.tool.execution` for the work. For a busier trace, run `review --otel` and
see the subagents' requests next to each other.

**Teaching points**

- **Isolation is a directory.** Two agents in one checkout would edit the same files. Two
  worktrees share the repository's history and nothing else, and a `cwd` per `query()` is
  all the SDK needs to keep them apart.
- **Mind what a worktree does not have.** A new worktree has no `node_modules`. The
  program symlinks the main checkout's, which is fast and has one catch: the workspace
  packages resolve to the main checkout's copy. A fix to `packages/api-contract` would be
  tested against the old contract.
- **The program picks, not an agent.** Tests decide who is green, and a sort by cost
  decides the winner. No model votes on another model's work.
- **A cheaper model can be the right one.** Haiku's fix costs a quarter of Sonnet's and
  passes the same tests. The race makes that visible per task instead of per opinion.
- **Telemetry lives in the CLI, the SDK only switches it on.**
  `CLAUDE_CODE_ENABLE_TELEMETRY=1`, `CLAUDE_CODE_ENHANCED_TELEMETRY_BETA=1`, and
  `OTEL_TRACES_EXPORTER=otlp` produce the traces, and the standard `OTEL_EXPORTER_OTLP_*`
  variables say where they go. The same variables work for `claude -p` and the TUI.
- **The permission wait is a span of its own.** In a gated run like `fix`, the
  `blocked_on_user` spans show how long the agent waited for you. That is the number to
  watch when an approval flow feels slow.

**Verify:** the race keeps one green branch and removes both worktrees, `git worktree
list` shows only your checkout, and Jaeger shows a trace with tool spans. Stop Jaeger
with `docker rm -f jaeger`.

---

## Step 36: continuous integration without AI

**Goal:** the fork has a GitHub Actions workflow that runs lint, unit tests, the build,
and the Playwright suite on every pull request, green on the first run.

GitHub disables Actions on a fresh fork. Open the Actions tab of your fork and enable
workflows before you start.

```bash
git switch -c ci
```

> **Prompt 36.1**
>
> Add continuous integration: a GitHub Actions workflow in .github/workflows/ci.yml that
> runs on pull requests and on pushes to main. One job on ubuntu-latest with Node 24: npm
> ci, npm run lint, npm test, npm run build, and then the Playwright suite without the
> LLM spec. Nothing in CI calls a model or needs a real secret, so write a .env for the
> run with a generated BETTER_AUTH_SECRET and a placeholder OPENROUTER_API_KEY, and
> migrate the database before the build. Install only Chromium with its system
> dependencies, cache npm, and upload the Playwright report when the job fails. Add the
> workflow to the AGENTS.md map. AGENTS.md and playwright.config.ts have what you need.
> Don't run the suites here; the run on GitHub is the test.

Expect about a minute and 0.40 USD. Read the workflow before you push it. Expect one job
named `ci`, a `.env` written from a heredoc with `openssl rand -base64 32`, the e2e step
with `--reporter=list,html` because the config only has the list reporter, and an
upload step guarded by `if: failure()`. Expect `permissions: contents: read` and a
`concurrency` group that cancels a superseded run.

```bash
git add -A && git commit -m "Add CI" && git push -u origin ci
gh pr create --fill
gh pr checks --watch
```

The run takes about three minutes and is green.

**Teaching points**

- **The agent read the repository's own traps.** `AGENTS.md` says that `npm install`
  builds the CLI and that the e2e server has its own port and dist folder.
  `playwright.config.ts` says that `*.llm.spec.ts` only runs with `E2E_LLM`. The workflow
  follows both without being told.
- **No secret in the plain CI job.** The app needs an OpenRouter key to start, and no
  test calls the model, so a placeholder is enough. A secret that CI does not need
  cannot leak from CI.
- **"Don't run it here" is a time lever.** Building and running the e2e suite locally
  would double the step's duration and prove less than the run on a clean runner.

**Verify:** the pull request shows a green `ci` check.

## Step 37: the agent as a pipeline step, and a gate

**Goal:** every pull request gets a security review of its diff by `claude -p`, the
findings land in the job summary, a high finding fails the job, and branch protection
makes that failure block the merge.

### The secret

The review job needs an Anthropic API key, and a subscription login does not work on a
runner. Add it to your fork as a repository secret. Type it into the prompt of this
command, never into a chat or a file:

```bash
gh secret set ANTHROPIC_API_KEY
```

Pull requests from forks of your fork do not get secrets, so the review job fails there
by design.

### The job

> **Prompt 37.1**
>
> Add a second job ai-review to .github/workflows/ci.yml that runs on pull requests
> only. It installs Claude Code with npm and pipes the pull request's diff against its
> base branch into claude -p, asking for a security review of that diff. Claude gets
> read-only tools, so it can read the code around a change. Use --bare, so nothing from
> the checkout's CLAUDE.md, skills, or hooks loads, and --json-schema with
> demos/a2-findings.schema.json. Cap the run with --max-turns and --max-budget-usd. The
> job writes the findings as a Markdown table into the job summary and fails when a
> finding has severity high. The key comes from the repository secret
> ANTHROPIC_API_KEY. Add the workflow to the AGENTS.md map.

Expect about two minutes and 0.60 USD. Read the new job with the room. Expect:

- `actions/checkout` with `fetch-depth: 0`, so the merge base exists, and
  `persist-credentials: false`, because Claude can read every file in the checkout,
  `.git/config` included.
- `git diff "origin/$BASE_REF...HEAD" | claude --bare -p "$PROMPT"` with `--tools
  "Read,Grep,Glob"`, `--permission-mode dontAsk`, `--output-format json`, the findings
  schema, `--max-turns`, and `--max-budget-usd`.
- A prompt that tells the model the diff is data and to ignore instructions written
  inside it.
- A report step with `if: ${{ !cancelled() }}`, so a run that hits its budget still
  writes a summary, and a final `jq` count of high findings that sets the exit code.

The job does not pin a model, so it uses Claude Code's default for API keys. Add
`--model` when you want the cost per review to stay predictable.

Commit, push, and watch both checks:

```bash
git add -A && git commit -m "Add AI review" && git push
gh pr checks --watch
```

On this pull request, whose diff is YAML and one line of `AGENTS.md`, expect the review to
finish within seconds, cost a cent or two, and report nothing high. A low finding about the
unpinned `npm install -g` is fair. Open the run's summary page to see the findings table
and the cost line.

### The gate

Make both checks required for `main`:

```bash
gh api -X PUT "repos/{owner}/{repo}/branches/main/protection" --input - <<'EOF'
{"required_status_checks":{"strict":false,"contexts":["ci","ai-review"]},"enforce_admins":true,"required_pull_request_reviews":null,"restrictions":null}
EOF
```

`enforce_admins` makes the rule apply to you as the owner too. Merge the CI pull request
once both checks are green.

Now open a pull request that removes the per-user filter from `setTodoDoneFor` in
`lib/todo-tools.ts`, so the `where` clause checks only the item id, with a commit
message that calls it a simplification. Expect two red checks within about three
minutes. `ci` fails, because `tests/unit/todo-tools.test.ts` has a test named
"setTodoDone cannot reach another student's item". `ai-review` fails after about ten
seconds and three cents with one high finding on the changed line. Open the job summary
and read the finding and its fix with the room. `gh pr view --json mergeStateStatus`
reports `BLOCKED`, and the merge button agrees. Close the pull request without merging.

**Teaching points**

- **`--bare` is the answer to "`-p` trusts the checkout".** A pull request can change
  `.claude/settings.json`, add a hook, or edit `CLAUDE.md`, and a plain `claude -p` on
  the runner would load all of it with the API key in the environment. `--bare` loads
  none of it.
- **The diff is untrusted input.** Anyone who can open a pull request writes part of the
  prompt. Read-only tools, no persisted Git credentials, a budget, a turn cap, and a
  schema are what keep a hostile diff from turning the reviewer into something else.
- **Two gates, two kinds of evidence.** The unit test is deterministic and knows exactly
  one rule. The review explains the problem in words and catches changes no test pins,
  such as the HTTP MCP server's scoping, which step 32 listed as untested. Keep both.
- **Branch protection is the human-in-the-loop part.** The agent does not merge or
  block anything itself. It produces a check, and a repository rule that a person
  configured decides what a red check means.
- **The polished products do the same thing with more glue.**
  `anthropics/claude-code-action` runs Claude Code in your workflow and answers `@claude`
  mentions in issues and pull requests. Code Review is a managed multi-agent review on
  Anthropic's infrastructure. Routines run a saved prompt on a schedule, an API call, or a
  GitHub event. All three are the harness from this morning plus GitHub integration.

**Verify:** a pull request shows `ci` and `ai-review`, the job summary holds the
findings table, the pull request that drops the per-user filter cannot be merged, and
branch protection lists both checks as required.

---

## Wrap-up

The same agent ran four ways today:

+-----------------+---------------------+---------------------+-----------------------+
|                 | Who decides         | What you get back   | Where it fits         |
+=================+=====================+=====================+=======================+
| `claude` (TUI)  | You, prompt by      | A conversation      | Work at your desk     |
|                 | prompt              |                     |                       |
+-----------------+---------------------+---------------------+-----------------------+
| `claude -p`     | Flags: tools,       | Text, JSON that     | Scripts, npm scripts, |
|                 | allowlist, schema,  | matches a schema,   | one-off reports and   |
|                 | budget              | or an event stream  | protocols             |
+-----------------+---------------------+---------------------+-----------------------+
| Agent SDK       | Your code: hooks,   | Typed messages and  | Tools you build for   |
|                 | callbacks, custom   | a rewind            | your team, with a     |
|                 | tools               |                     | human gate            |
+-----------------+---------------------+---------------------+-----------------------+
| CI step         | The workflow and    | A check that is red | Every pull request    |
|                 | branch protection   | or green            |                       |
+-----------------+---------------------+---------------------+-----------------------+

Pick the lightest one that does the job. A question you ask once is a `claude -p`
one-liner. A question you ask every week with a schema and a gate is a script around
`claude -p`. When you need a callback in the middle of the loop, a person to approve, a
tool of your own, or a rewind, write a program with the SDK.

What is left for the talk that closes the day: deployment, rollback, and hosting an
agent for other people, where the subscription stops and API keys, budgets, and
monitoring start.

---

## Appendix A: rehearsing this storybook headless

To rehearse the day without the TUI, work in a copy of your fork with `npm run dev`
running, and run each step's commands as written. Pin the model so your numbers match
the ones in this storybook:

```bash
export ANTHROPIC_MODEL=claude-opus-5-5
```

The TUI prompts, 31.1, 36.1, and 37.1, run headless with a scoped allowlist. That is
safer than `--dangerously-skip-permissions`, and it shows the same permission denials
the TUI would ask about:

```bash
claude -p "<Prompt 31.1>" --permission-mode dontAsk \
  --allowedTools "Read,Grep,Glob,Bash(bash demos/b1-bakeoff.sh),Bash(jq *),Write(./bakeoff/**),Edit(./bakeoff/**)" \
  --output-format stream-json --verbose | jq -r -f demos/a3-tool-calls.jq
claude -p "<Prompt 36.1>" --permission-mode dontAsk \
  --allowedTools "Read,Grep,Glob,Write(./.github/**),Edit(./.github/**),Edit(./AGENTS.md)" --output-format json
claude -p "<Prompt 37.1>" --permission-mode dontAsk \
  --allowedTools "Read,Grep,Glob,Write(./.github/**),Edit(./.github/**),Edit(./AGENTS.md)" --output-format json
```

The steps 33 and 34 fixes ask questions in the terminal. To rehearse them unattended,
feed the answers from a script that watches for `Allow this change? [y/N]` and `Keep the
changes? [Y/n]` on the program's output. A plain `yes y |` works for a fix you keep.

Steps 36 and 37 need a GitHub repository with Actions enabled and the
`ANTHROPIC_API_KEY` secret. Use a throwaway repository, because branch protection with
`enforce_admins` also blocks your own direct pushes to `main`.

In the live session, use the TUI for prompts 31.1, 36.1, and 37.1. Tool calls, permission
questions, and the agent's reasoning scrolling past are what the room learns from.

## Appendix B: live-demo insurance

- **Pin what worked.** The day ran on Claude Code 2.1.283, `@anthropic-ai/claude-agent-sdk`
  0.3.283 (which bundles the same CLI), pi 0.87.1, and Jaeger 2.21.0. The starter's
  lockfile pins the SDK. Don't update Claude Code on the morning of the session.
- **Run steps 30 and 34 early, or show a finished run.** Together they take 13 minutes of
  agent time and about 5 USD. Keep `reports/`, `test-protocol/`, and a branch with the
  step 34 fix from your rehearsal, and show those if a live run stalls.
- **The review team usually finds the leak, not always.** The security reviewer finds
  the thread leak reliably, and it is still a model. If a run misses it, show the
  rehearsal's findings and run `demos/c4-thread-leak.sh` anyway. The script is the
  evidence, not the review.
- **A backgrounded pi hangs without `</dev/null`.** If a bake-off contestant never
  finishes, check that its line in the script still redirects stdin.
- **`--bare` needs an API key.** With only a claude.ai login it answers `Not logged in`.
  Use `--safe-mode` on stage, and `--bare` in CI.
- **Budgets on Opus.** The first call of a session writes the prompt cache and costs about
  0.45 USD on this machine, more with many connectors. A budget below that stops every
  run after one call.
- **The OpenRouter cost field for glm.** Claude Code logs `[claude-code:unrecognized_model]`
  and prices glm like a Claude model. Take real costs from OpenRouter's activity page or
  from pi's per-message `cost`.
- **Leftover worktrees.** An interrupted race leaves sibling directories ending in
  `-race-sonnet` and `-race-haiku` and branches named `race/...`. `git worktree remove
  --force <dir>`, `git worktree prune`, and `git branch -D` clean up.
- **Actions on forks.** A fork has Actions disabled until someone enables them in the
  Actions tab, and pull requests from forks get no secrets.
- **The auto-mode classifier.** An agent that runs with auto mode may refuse to write a
  deliberately weakened authorization check for the red pull request in step 37. Make
  that one-line change by hand.
- **Relative dates and prices move.** The `npm audit` counts in step 26 and the
  OpenRouter prices in step 31 change with time. Rehearse on the day before the session.
