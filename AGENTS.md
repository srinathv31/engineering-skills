# Agent Instructions

This repository contains reusable skills for AI coding agents. Each skill provides domain-specific patterns, rules, and examples to improve code generation quality.

## Repository Purpose

A personal collection of agentic coding skills for use across multiple AI development tools including Claude Code, Codex, GitHub Copilot, and others.

## Directory Structure

```
engineering-skills/
├── README.md           # Human-readable documentation
├── AGENTS.md           # This file - agent guidance
├── CLAUDE.md           # Claude Code specific (symlink to AGENTS.md)
└── skills/
    └── {skill-name}/
        ├── SKILL.md    # Quick reference with metadata
        ├── AGENTS.md   # Complete expanded guide
        └── rules/      # Individual rule files (optional)
```

## Skill Naming Conventions

- **Directories**: Use `kebab-case` (e.g., `drizzle-core`, `react-best-practices`)
- **Required file**: Every skill must have a `SKILL.md`
- **Rule files**: Use `{prefix}-{rule-name}.md` format (e.g., `schema-table-definition.md`)

## SKILL.md Format

Each skill's `SKILL.md` must include YAML frontmatter:

```yaml
---
name: skill-name
description: Brief description and trigger conditions
license: MIT
metadata:
  author: community
  version: "1.0.0"
---
```

### Required Sections

1. **Title** (H1) - Skill name
2. **When to Apply** - List of scenarios when to use this skill
3. **Rule Categories by Priority** - Table organizing rules by impact
4. **Quick Reference** - Categorized rule list with brief descriptions
5. **How to Use** - Instructions for accessing detailed rules

## AGENTS.md Format (Per Skill)

The expanded guide should include:

1. **Abstract** - What this skill covers
2. **Table of Contents** - Navigation to sections
3. **Rule Sections** - Each rule with:
   - Impact level (CRITICAL, HIGH, MEDIUM)
   - Explanation of why it matters
   - Incorrect code example
   - Correct code example
   - Additional context

## Context Efficiency Principles

When creating skills:

- Keep `SKILL.md` under 100 lines for quick loading
- Keep `AGENTS.md` under 500 lines total
- Use specific trigger descriptions
- Prefer individual rule files over inline examples
- Use progressive disclosure (summary → details)

## How to Apply Skills

When working in a codebase:

1. Check if the task matches a skill's trigger conditions
2. Load the relevant `SKILL.md` for quick reference
3. Consult `AGENTS.md` or specific rule files for detailed guidance
4. Apply patterns from correct code examples
5. Avoid anti-patterns from incorrect code examples

## Available Skills

| Skill | Triggers On |
|-------|-------------|
| `drizzle-core` | Drizzle ORM schema definition, table creation, relationships, constraints |
| `drizzle-mssql` | MSSQL databases, SQL Server, T-SQL, drizzle-orm/mssql-core |
| `shadcn-ui` | shadcn/ui components, Next.js UI, forms with react-hook-form, data tables, charts, Radix UI primitives |
