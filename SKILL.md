---
name: skill-agent-writer
description: This skill should be used when the user asks to "write a skill", "create a SKILL.md", "review a skill", "refactor a skill", "write an agent", "create an agent", "review an agent", "에이전트 작성", "에이전트 만들기", "에이전트 리뷰", "스킬 작성", "스킬 만들기", "SKILL.md 만들기", "스킬 리뷰", "스킬 리팩토링", "skill best practices", "agent best practices", "스킬 베스트 프랙티스", or wants to create, review, or improve SKILL.md or agent .md files following Anthropic best practices.
version: 2.0.0
user-invocable: true
---

# Skill & Agent Writer

A skill for creating, reviewing, and refactoring high-quality `SKILL.md` and
agent `.md` files. Skills should follow the Vercel Agent Skills structure: each
skill is a directory containing `SKILL.md` at its root, with optional
`scripts/`, `references/`, and `assets/` directories. All guidance is based on
the bundled Anthropic skills guide, Vercel-style agent skill layout, and agent
design patterns in the `references/` directory.

## Vercel Skills Layout

Use this structure for skills intended to work with `npx skills add`:

```text
<skill-name>/
  SKILL.md
  scripts/       # optional executable helpers
  references/    # optional docs loaded as needed
  assets/        # optional output resources
```

For a multi-skill collection repository:

```text
skills/
  <skill-name>/
    SKILL.md
```

For a single-skill repository, the repository root may be the skill root:

```text
SKILL.md
scripts/
references/
assets/
```

## `.agents` and Symlink Strategy

The Vercel skills CLI installs skills into a canonical skills directory such as:

```text
.agents/skills/<skill-name>/
~/.agents/skills/<skill-name>/
```

Some agents read `.agents/skills/` directly. Other agents expect their own
directory, so link the canonical skill instead of duplicating files.

Project-local examples:

```bash
mkdir -p .agents/skills .claude/skills .codex/skills
ln -s ../../.agents/skills/<skill-name> .claude/skills/<skill-name>
ln -s ../../.agents/skills/<skill-name> .codex/skills/<skill-name>
```

Global examples:

```bash
mkdir -p ~/.agents/skills ~/.claude/skills ~/.codex/skills
ln -s ~/.agents/skills/<skill-name> ~/.claude/skills/<skill-name>
ln -s ~/.agents/skills/<skill-name> ~/.codex/skills/<skill-name>
```

If a project uses a singular `.agent/` directory instead of `.agents/`, keep the
same rule: maintain one canonical skill directory and symlink agent-specific
locations to it. Do not hand-edit duplicate copies.

Prefer symlinks for development because edits remain centralized. Use copy mode
only when the target agent or environment cannot follow symbolic links.

## Modes

This skill operates in five modes:

1. **Create Skill** — Interactive wizard that gathers requirements, then generates a complete Vercel-style skill directory with `SKILL.md` at the root and optional bundled resources.
2. **Review Skill** — Analyzes an existing SKILL.md against a 10-point quality checklist and produces a scored report.
3. **Refactor Skill** — Applies review findings to improve an existing SKILL.md while preserving its original intent.
4. **Create Agent** — Interactive wizard that gathers requirements, then generates an agent `.md` file following project patterns.
5. **Review Agent** — Analyzes an existing agent `.md` file against an 8-point agent quality checklist and produces a scored report.

Determine the mode from user intent. If ambiguous, ask.

## Skill Quality Checklist (10 Points)

Modes 1-3 use this checklist derived from the Anthropic skills guide:

| # | Criterion | Max Score |
|---|-----------|-----------|
| 1 | **Clear Purpose** — Concise description explaining what the skill does and when to use it | 2 |
| 2 | **Trigger Coverage** — Both English and Korean trigger keywords in the description | 2 |
| 3 | **Frontmatter** — Valid YAML with name, description, version, user-invocable | 2 |
| 4 | **Structured Workflow** — Step-by-step numbered instructions | 2 |
| 5 | **Tool Guidance** — Specifies which tools to use (Read, Write, Edit, Bash, etc.) | 2 |
| 6 | **Output Format** — Defines expected output with examples or templates | 2 |
| 7 | **Edge Cases** — Includes "When to Use" / "When to Skip" guard conditions | 2 |
| 8 | **Actionable Instructions** — Uses imperative verbs and specific actions | 2 |
| 9 | **Reference Material** — Links to reference files or provides inline examples | 2 |
| 10 | **Completeness** — Self-contained; an LLM can execute the full workflow | 2 |

**Grading**: A (18-20), B (14-17), C (10-13), D (6-9), F (0-5)

## Agent Quality Checklist (8 Points)

Modes 4-5 use this checklist derived from `docs/05-agents-deep-dive.md` and the Anthropic agent harness guide:

| # | Criterion | Max Score |
|---|-----------|-----------|
| 1 | **Clear Purpose** — One paragraph explaining what the agent does | 2 |
| 2 | **Analysis Process** — Numbered steps describing the agent's workflow | 2 |
| 3 | **Output Format** — Explicit markdown structure for results | 2 |
| 4 | **Classification/Categorization** — Structured decision logic | 2 |
| 5 | **Quality Standards** — Criteria for good output | 2 |
| 6 | **Guard Conditions** — Skip conditions or scope limits | 2 |
| 7 | **Tool Hints** — Lists tools needed (if any) | 2 |
| 8 | **Completeness** — Self-contained, executable by LLM without external context | 2 |

**Grading**: A (14-16), B (11-13), C (8-10), D (5-7), F (0-4)

---

## Create Skill Workflow

### Step 1: Gather Requirements

Use the AskUserQuestion tool to collect:

1. **Skill name** — kebab-case identifier (e.g., `code-reviewer`)
2. **Purpose** — What does this skill do? (1-2 sentences)
3. **Trigger phrases** — Natural language phrases that should activate it (English + Korean)
4. **Modes** — How many operating modes? What does each do?
5. **Tools needed** — Which tools are needed? (Read, Write, Edit, Bash, Glob, Grep, Task/spawn_agent, WebSearch/WebFetch, user input)
6. **Output format** — What should the final output look like?
7. **Reference files** — Does the skill need reference documentation?

If the user provides partial info, fill in reasonable defaults and confirm.

### Step 2: Read Reference Material

Before generating, read the relevant reference files to ensure best practices:

- Read `references/anthropic-skills-guide/03-skill-structure.md` for structure guidance
- Read `references/anthropic-skills-guide/04-writing-instructions.md` for instruction quality
- Read `references/anthropic-skills-guide/07-patterns.md` for common patterns

Use the local path beside this `SKILL.md`:
`references/anthropic-skills-guide/{filename}`. In legacy Claude Code plugin
installs, `${pluginDir}/skills/skill-agent-writer/references/...` may also be
valid.

### Step 3: Generate via Agent

Use the Task tool to launch the `skill-generator` agent with subagent_type `skill-agent-writer:skill-generator`. Pass the gathered requirements as the prompt.

The agent should produce `SKILL.md` and any requested bundled resources.
Localized variants such as `SKILL-ko.md` are optional.

### Step 4: Write Files

Use the Write tool to create the skill files:

1. Write `SKILL.md` at the user's specified path.
   - Default for Vercel-style collection repos: `skills/{name}/SKILL.md`
   - Default for single-skill repos: `SKILL.md` at repo root
   - Default for direct Claude Code installs: `.claude/skills/{name}/SKILL.md`
2. Write `scripts/`, `references/`, or `assets/` only when the skill needs them.
3. Write localized variants such as `SKILL-ko.md` only when requested or when
   the existing project already uses them.
4. If the user wants multi-agent availability, document or create symlinks from
   `.agents/skills/{name}` to agent-specific skill directories.

### Step 5: Validate

Run a quick review against the 10-point skill checklist. If any criterion scores 0, flag it and offer to fix.

---

## Review Skill Workflow

### Step 1: Identify Target

Ask the user which SKILL.md to review. If not specified, use Glob to find SKILL.md files:

```
**/SKILL.md
.claude/skills/*/SKILL.md
plugins/*/skills/*/SKILL.md
```

### Step 2: Read the File

Use the Read tool to load the target SKILL.md.

### Step 3: Read Checklist Reference

Read `references/anthropic-skills-guide/09-quick-reference.md` for the evaluation criteria.

### Step 4: Run Review Agent

Use the Task tool to launch the `skill-reviewer` agent with subagent_type `skill-agent-writer:skill-reviewer`. Pass the SKILL.md content and the checklist.

### Step 5: Present Report

Display the scored report to the user. Ask whether they want to:
- **Refactor** — Apply fixes automatically (switches to Refactor Skill mode)
- **Fix specific items** — Address only selected weaknesses
- **Done** — Accept the report as-is

---

## Refactor Skill Workflow

### Step 1: Ensure Review Exists

If no review report is available, run Review Skill first (Steps 1-4 above).

### Step 2: Read Original Files

Use the Read tool to load both:
- The original SKILL.md
- The original SKILL-ko.md (if it exists)

### Step 3: Run Refactorer Agent

Use the Task tool to launch the `skill-refactorer` agent with subagent_type `skill-agent-writer:skill-refactorer`. Pass the original content and the review report.

### Step 4: Show Diff and Confirm

Present the change summary to the user. Show what will be modified and the expected score improvement.

Use the AskUserQuestion tool to confirm before overwriting:
- **Apply all changes** — Write the refactored files
- **Apply selectively** — Let the user choose which changes to keep
- **Cancel** — Discard the refactored version

### Step 5: Write Updated Files

Use the Edit tool (preferred) or Write tool to update:
1. SKILL.md with the refactored content
2. SKILL-ko.md with the updated Korean translation

---

## Create Agent Workflow

### Step 1: Gather Requirements

Use the AskUserQuestion tool to collect:

1. **Agent name** — kebab-case identifier (e.g., `code-reviewer`)
2. **Purpose** — What does this agent analyze or generate? (1-2 sentences)
3. **Style** — Plain markdown (session-wrap style) or frontmatter (youtube-digest style)?
4. **Tools needed** — Which tools? (Read, Write, Bash, Grep, Glob, WebSearch, etc.)
5. **Workflow steps** — High-level numbered steps for the agent's process
6. **Output format** — What should the agent's output look like?
7. **Model hint** — `sonnet` (analysis/generation) or `haiku` (validation/classification)?

If the user provides partial info, fill in reasonable defaults and confirm.

### Step 2: Read Reference Material

Before generating, read the relevant reference files:

- Read `${pluginDir}/skills/skill-agent-writer/references/anthropic-agent-harness-guide.md` for agent design patterns
- Read the project's agent deep-dive guide for structural patterns:
  - Use the Glob tool to find `docs/05-agents-deep-dive.md` in the project root
  - If found, read it for the two agent styles and design patterns

Optionally read an existing agent as a template:
- Plain style example: `plugins/session-wrap/agents/doc-updater.md`
- Frontmatter style example: `plugins/youtube-digest/agents/transcript-extractor.md`

### Step 3: Generate via Agent

Use the Task tool to launch the `agent-generator` agent with subagent_type `skill-agent-writer:agent-generator`. Pass the gathered requirements as the prompt.

The agent will produce a single agent `.md` file.

### Step 4: Write File

Use the Write tool to create the agent file at:
- `plugins/{plugin-name}/agents/{agent-name}.md` (for plugin agents)
- Or the user's specified path

### Step 5: Validate

Run a quick review against the 8-point agent checklist. If any criterion scores 0, flag it and offer to fix.

---

## Review Agent Workflow

### Step 1: Identify Target

Ask the user which agent `.md` file to review. If not specified, use Glob to find agent files:

```
plugins/*/agents/*.md
.claude/agents/*.md
```

### Step 2: Read the File

Use the Read tool to load the target agent `.md` file.

### Step 3: Read Checklist Reference

Read `${pluginDir}/skills/skill-agent-writer/references/anthropic-agent-harness-guide.md` for agent quality criteria.

### Step 4: Run Review Agent

Use the Task tool to launch the `agent-reviewer` agent with subagent_type `skill-agent-writer:agent-reviewer`. Pass the agent `.md` content and the 8-point checklist.

### Step 5: Present Report

Display the scored report to the user. Ask whether they want to:
- **Fix issues** — Apply improvements manually based on findings
- **Done** — Accept the report as-is

---

## When to Use

- Creating a new skill from scratch
- Creating a new agent from scratch
- Evaluating the quality of an existing SKILL.md
- Evaluating the quality of an existing agent `.md`
- Improving a low-scoring SKILL.md
- Learning SKILL.md or agent best practices
- Ensuring bilingual (English + Korean) skill coverage

## When to Skip

- The user already has a complete, working file and doesn't want changes
- The task is about editing application code, not skill/agent files
- The user is asking about general Claude Code usage (not skill/agent authoring)

## Reference Files

### Skill Writing References

For detailed guidance on writing skills, consult the bundled Anthropic skills guide:

- **`references/anthropic-skills-guide/README.md`** — Overview and table of contents
- **`references/anthropic-skills-guide/01-fundamentals.md`** — What skills are and how they work
- **`references/anthropic-skills-guide/02-planning-and-design.md`** — Planning your skill before writing
- **`references/anthropic-skills-guide/03-skill-structure.md`** — File structure and frontmatter format
- **`references/anthropic-skills-guide/04-writing-instructions.md`** — How to write clear, actionable instructions
- **`references/anthropic-skills-guide/05-testing.md`** — Testing and validating skills
- **`references/anthropic-skills-guide/06-distribution.md`** — Sharing and distributing skills
- **`references/anthropic-skills-guide/07-patterns.md`** — Common skill patterns and templates
- **`references/anthropic-skills-guide/08-troubleshooting.md`** — Common issues and solutions
- **`references/anthropic-skills-guide/09-quick-reference.md`** — Cheat sheet and checklist

### Agent Writing References

- **`references/anthropic-agent-harness-guide.md`** — Anthropic blog summary: initializer/coding agent pattern, progress tracking, failure modes
- **`docs/05-agents-deep-dive.md`** — Project-level agent guide: two styles, execution patterns, design patterns
