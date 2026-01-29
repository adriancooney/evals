# Evals

An eval skill for agents. Create and run evals entirely within the coding agent—no external testing frameworks, no additional tooling, just you and your agent.

## Installation

```bash
npx skills add adriancooney/evals
```

## Usage

Walk the agent through any process—simple or arbitrarily complex. Set up a database schema, configure a build pipeline, implement an authentication flow, refactor a module. The process can span multiple files, multiple tools, require external API calls, or involve intricate multi-step workflows. It doesn't matter.

Once you're happy with the result, ask the agent to create an eval.

```
> lets test the login flow and check that I got an email

● Opening http://localhost:3000/signup with agent-browser
● Filling in test@example.com and password
● Clicking "Create Account"
● Account created successfully, welcome page displayed

> ok now check my email in chrome

● Opening Gmail via Claude Chrome extension
● Searching for "Welcome to Acme"
● Found email from no-reply@acme.com received 12 seconds ago
● Subject: "Welcome to Acme - Please verify your email"

> perfect, lets create an eval

● Using evals skill
● What should this eval be called? signup-flow
● Writing eval to signup-flow.eval.md
```

The agent distills what it just did into a prompt and expectation:

```markdown
# Signup Flow

<prompt>
Open http://localhost:3000/signup with agent-browser. Fill in test@example.com
and a password, then click "Create Account". Open Gmail via the Chrome extension
and search for "Welcome to Acme".
</prompt>

<expectation>
The signup completes and displays a welcome page. A verification email from
no-reply@acme.com arrives with subject "Welcome to Acme - Please verify your email".
</expectation>
```

To verify it works, run the eval. It spawns a subagent with a clean context—no memory of the original session—proving the process is reproducible.

```
> /evals run signup-flow.eval.md

● Spawning subagent to run eval
● Subagent: Opening http://localhost:3000/signup with agent-browser
● Subagent: Filling in test@example.com and password
● Subagent: Account created, welcome page displayed
● Subagent: Opening Gmail, searching for "Welcome to Acme"
● Subagent: Found verification email from no-reply@acme.com

✓ SUCCESS — signup completed and verification email received
```

## Why

Traditional eval frameworks want you to spin up infrastructure, write custom harnesses, learn their DSL, configure runners, and context-switch between your agent and some external tool. It's exhausting. You just wanted to test if the thing works.

This skill is ~80 lines of markdown. It runs entirely inside your coding agent. No external tools. No dependencies. No setup. The agent does the work. Subagents run your evals in parallel. LLM-as-judge determines pass/fail. Everything happens in the same place you write code.

It's stupidly simple and unreasonably powerful:

- **One command to capture complexity** — turn a 47-step process into a replayable eval
- **Just markdown** — evals are prompts and expectations, nothing else
- **Agents all the way down** — subagents execute, observe, and judge
- **Full agent capabilities** — evals can use skills, MCP servers, browser automation, whatever your agent can do
- **Parallel by default** — run 50 evals at once, why not
- **Fresh context every time** — each eval proves reproducibility from scratch

## How It Works

An eval is a markdown file with two sections:

```markdown
# My Eval

<prompt>
Instructions for the agent to execute.
</prompt>

<expectation>
What must be true for the eval to pass.
</expectation>
```

Run it:
```
/evals run path/to/my.eval.md
```

The skill spawns a subagent that:
1. Executes the prompt
2. Observes the outcome
3. Judges whether the expectation was met
4. Reports `SUCCESS` or `FAIL` with reasoning

When running multiple evals, all subagents spawn in parallel and results are aggregated at the end.

## Examples

See the [evals/](https://github.com/adriancooney/evals/tree/main/evals) directory for real examples:

- [`skill-install.eval.md`](https://github.com/adriancooney/evals/blob/main/evals/skill-install.eval.md) — tests that this skill installs correctly
- [`agent-tui-eval-creation.eval.md`](https://github.com/adriancooney/evals/blob/main/evals/agent-tui-eval-creation.eval.md) — tests eval creation via [agent-tui](https://github.com/anthropics/claude-code/tree/main/packages/agent-tui) automation

## Commands

| Command | Description |
|---------|-------------|
| `/evals run <path>` | Run a single eval |
| `/evals run-all` | Run all `*.eval.md` files |
| `/evals create <name>` | Create a new eval interactively |

Run from the command line:
```bash
$ claude -p "/evals run-all"
```

```
## Eval Results

| Eval | Result |
|------|--------|
| `agent-tui-eval-creation.eval.md` | ✓ SUCCESS |

**1/1 evals passed**
```

## CI

Why not run it on CI?

```yaml
- name: Run evals
  run: |
    output=$(claude -p "/evals run-all")
    echo "$output"
    echo "$output" | grep -q "^eval result: pass$"
```

The skill outputs `eval result: pass` or `eval result: fail` as its final line.

⚠️ **Be careful.** LLM-based evaluation is non-deterministic.

---

## Evals All The Way Down

This repo contains an eval that tests the process of creating an eval. Using [agent-tui](https://github.com/anthropics/claude-code/tree/main/packages/agent-tui), we automate Claude Code to perform a task, then have it generate an eval of that task, then wrap the whole thing in another eval.

```
┌─────────────────────────────────────────────────────────────┐
│  Layer 3: This repo                                         │
│  evals/agent-tui-eval-creation.eval.md                      │
│  "Test that agent-tui can automate Claude Code to create    │
│   an eval"                                                  │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  Layer 2: Claude Code session (automated via agent-tui)│  │
│  │  /evals create file-copy                              │  │
│  │  "Create an eval for the file duplication task"       │  │
│  │                                                       │  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │  Layer 1: The actual task                       │  │  │
│  │  │  Create data.txt → duplicate → verify           │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

Run it yourself:
```bash
/evals run evals/agent-tui-eval-creation.eval.md
```

This spawns an agent that uses agent-tui to automate Claude Code, which creates files and generates its own eval—then verifies everything worked.
