# Eval Inception

We just created an eval that tests the process of creating an eval.

## The Layers

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

## What Happened

1. **agent-tui** launched Claude Code in a tmp directory
2. Claude Code created `data.txt`, duplicated it, verified the copy
3. Claude Code ran `/evals create` to generate an eval of that task
4. We then created an eval in *this* repo that tests the entire flow

## Running

```bash
/evals run evals/agent-tui-eval-creation.eval.md
```

This spawns an agent that uses agent-tui to automate Claude Code, which creates files and generates its own eval—then verifies everything worked.

Evals all the way down.
