# Effective Harnesses for Long-Running Agents

A structured summary of Anthropic's blog post on building effective harnesses for long-running AI agents.

> Source: [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)

## Core Problem

Long-running agents must work in discrete sessions. Each new session begins with no memory of what came before. The challenge is enabling agents to quickly understand the state of work when starting with a fresh context window.

## Two-Agent Architecture

### Initializer Agent

Runs once at the start of a project. Performs three functions:

1. **Creates `init.sh` script** — Establishes a development server startup mechanism
2. **Generates `claude-progress.txt`** — Maintains activity logs across sessions
3. **Establishes initial git commit** — Documents baseline file additions

### Coding Agent

Runs in every subsequent session. Follows a five-step startup procedure:

1. **Run `pwd`** — Confirm working directory
2. **Read git logs and progress files** — Understand recent work context
3. **Read features list file** — Identify highest-priority incomplete features
4. **Start development server** — Execute `init.sh` script
5. **Run basic end-to-end test** — Verify application stability before new work

## Feature List JSON (Progress Tracking)

```json
{
  "category": "functional",
  "description": "New chat button creates conversation",
  "steps": ["Navigate interface", "Click button", "Verify creation"],
  "passes": false
}
```

- The `passes` field tracks completion status
- Agents modify only this boolean value
- Strict instructions prevent removal or editing of test definitions to avoid missing functionality
- A comprehensive feature list (200+ features in the demo) prevents the agent from declaring work done prematurely

## Four Failure Modes & Solutions

| Problem | Initializer Solution | Coding Agent Solution |
|---------|----------------------|----------------------|
| Declares entire project done prematurely | Create comprehensive feature list (200+ features) | Read feature list; work single features sequentially |
| Leaves buggy/undocumented code | Write git repo + progress notes file | Start with progress review; end with git commits + updates |
| Marks features complete without testing | Establish feature list with testable steps | Self-verify thoroughly; mark "passing" only after testing |
| Wastes time figuring out app startup | Write executable `init.sh` | Read `init.sh` at session start |

## Progress File Pattern (`claude-progress.txt`)

This file functions as session memory, containing:

- Summary of completed features
- Current implementation status
- Known issues or incomplete work
- Contextual notes for the next agent session

The progress file, combined with git history, enables rapid context recovery for fresh sessions.

## Session Startup Sequence (Example)

```
[Assistant] Getting bearings on current project state
[Tool Use] <bash - pwd>
[Tool Use] <read - claude-progress.txt>
[Tool Use] <read - feature_list.json>
[Assistant] Checking recent git work
[Tool Use] <bash - git log --oneline -20>
[Assistant] Starting development server
<Launch via init.sh>
[Assistant] Running fundamental feature verification
<Browser automation tests>
[Assistant] Reviewing remaining work, starting next feature
```

## Key Insights

1. **Context Recovery** — Compaction alone is insufficient; structured artifacts (progress files + git history) enable agents to rapidly understand project state
2. **Incremental Work** — Single-feature-per-session approach prevents context exhaustion and leaves clean, mergeable code
3. **Failure Prevention** — Strongly-worded instructions about JSON file integrity prevented accidental test removal
4. **Human Engineering Inspiration** — The approach mirrors practices effective software engineers use during shift changes
5. **Testing Tools** — Providing browser automation tools (Puppeteer MCP) dramatically improved performance, as the agent could identify and fix bugs not obvious from code alone

## Relevance to Agent Design

When creating agents for Claude Code plugins:

- **Structured artifacts** — Agents should produce well-defined output formats that can be consumed by downstream processes
- **Self-orientation** — Include steps for agents to understand their context before acting
- **Progress tracking** — Long-running workflows benefit from explicit progress markers
- **Guard conditions** — Prevent agents from taking shortcuts (declaring done, skipping tests)
- **Specialization** — It may be better to have specialized agents for sub-tasks rather than one general-purpose agent
