---
name: firebase-hol-guard
description: >-
  Protect local AI coding-agent sessions before Firebase CLI, MCP, and Agent
  Skills workflows with HOL Guard. Use when an agent can run commands or mutate
  a Firebase project and you want Guard-owned harness protection and evidence.
---

# Protect Firebase agent workflows with HOL Guard

Use HOL Guard at the local coding-agent boundary before starting Firebase work.
It protects supported local agent harnesses before their tools run. It does not
run inside Firebase services and does not replace Firebase Authentication,
Security Rules, App Check, IAM, emulator validation, or normal change review.

## Set up protection

Install HOL Guard in an isolated Python application environment:

```bash
pipx install hol-guard
```

Check the machine and discover supported local harnesses:

```bash
hol-guard status
hol-guard detect --json
```

For the harness that will perform Firebase work, bootstrap and install Guard,
then verify a protected launch before making project changes:

```bash
hol-guard bootstrap
hol-guard install <harness>
hol-guard run <harness> --dry-run
hol-guard run <harness>
hol-guard status
```

Use the harness identifier reported by `hol-guard detect`. Supported harnesses
include Codex, Claude Code, Copilot CLI, Cursor, Gemini, Hermes, OpenClaw,
OpenCode, and Antigravity. Do not claim a workspace is protected until Guard
reports the harness setup successfully.

## Use with Firebase Agent Skills

Start the coding agent through `hol-guard run <harness>` first. From that
protected session, use the Firebase skills and their documented Firebase CLI or
MCP workflows normally. Keep Firebase's own safety instructions in force,
including project-selection checks, user confirmation where a skill requires
it, Security Rules review, and emulator or preview steps.

HOL Guard owns the local harness protection boundary. This skill does not claim
that every Firebase CLI subcommand has a dedicated Guard classifier or that
Guard intercepts hosted Firebase services directly.

## Handle Guard decisions

If Guard blocks or queues work, inspect the request before proceeding:

```bash
hol-guard approvals
hol-guard approvals open
hol-guard receipts
hol-guard diff <harness>
```

Only approve after reviewing the risk reason and requested scope. Never bypass
Guard by launching an unprotected copy of the agent or editing around
Guard-owned hooks.

## Diagnose protection

If the harness does not appear protected, stop mutating Firebase resources and
inspect the setup:

```bash
hol-guard doctor
hol-guard doctor <harness> --json
hol-guard detect --json
hol-guard settings show
```

Resume Firebase mutations only after Guard output proves the local harness is
protected, or continue without claiming HOL Guard protection.
