# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A collection of Claude Code skills — reusable SKILL.md files that provide specialized workflows and reference material for Claude Code agents. Skills are installed into Claude Code's configuration and invoked via the Skill tool during conversations.

## Repository Structure

```
skills/
  <skill-name>/
    SKILL.md        # The skill definition (frontmatter + content)
```

Each skill is a directory under `skills/` containing a single `SKILL.md` file.

## Skill File Format

Every SKILL.md follows this structure:

```markdown
---
name: <skill-name>
description: <trigger description — tells Claude when to invoke this skill>
---

# Skill Title

<content: workflows, decision trees, reference tables, code examples>
```

- **name**: Must match the directory name
- **description**: Critical — this is what Claude uses to decide whether to invoke the skill. Write it as a trigger condition ("Use when..."), not a summary of contents.

## Conventions

- Skills use `dot` digraph blocks for decision flowcharts
- Skills include "When NOT to Use" or "When NOT to apply" guidance to prevent over-application
- Skills include "Red Flags" or "Common Mistakes" tables to catch process violations
- Code examples are provided in C# and TypeScript where applicable
- Skills reference other skills/agents by their tool name (e.g., `pr-review-toolkit:review-pr`)
