# Evals

An eval skill for agents. Create and run evals entirely within the coding agent—no external testing frameworks, no additional tooling, just prompts and expectations.

## Installation

```bash
npx skills add adriancooney/evals
```

## Usage

Walk the agent through any process—simple or arbitrarily complex. Set up a database schema, configure a build pipeline, implement an authentication flow, refactor a module. The process can span multiple files, require external API calls, or involve intricate multi-step workflows. It doesn't matter.

Once you're happy with the result, run `/evals create <name>`. The agent distills what it just did into a prompt and expectation, writing it to `<name>.eval.md`.

To verify it works, start a fresh session (`/clear` or new terminal) and run `/evals run <name>.eval.md`. The eval runs in a clean context with no memory of the original session—proving the process is reproducible.

## Why

Traditional eval frameworks require separate infrastructure, custom harnesses, and context switching between your agent and external tools. This skill keeps everything inside the coding engine:

- **Capture complexity fast** — turn any multi-step process into a replayable eval with one command
- **No external dependencies** — evals are markdown files with prompts and expectations
- **LLM-as-judge** — the agent evaluates its own output against success criteria
- **Parallel execution** — multiple evals run as subagents concurrently
- **Fresh context** — each eval runs in a subagent with no memory of prior work
- **Zero setup** — just write a `.eval.md` file and run it

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
