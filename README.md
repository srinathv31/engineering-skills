# Engineering Skills

A personal collection of reusable agentic coding skills for AI-powered development tools.

## Available Skills

| Skill | Description |
|-------|-------------|
| [drizzle-core](skills/drizzle-core) | Core Drizzle ORM schema definition patterns and best practices |
| [drizzle-mssql](skills/drizzle-mssql) | Microsoft SQL Server specific patterns for Drizzle ORM |

## Usage

### Claude Code

Copy the skill directory to your Claude Code skills directory:

```bash
cp -r skills/drizzle-core ~/.claude/skills/
```

Or reference directly in your project's `.claude/skills/` directory.

### GitHub Copilot

Add the `SKILL.md` and `AGENTS.md` files to your project's `.github/copilot-instructions/` directory.

### Codex / OpenAI

Include the skill's `AGENTS.md` content in your system prompt or project context.

### Other Tools

Most AI coding tools support custom instructions. Use the `SKILL.md` for quick reference or `AGENTS.md` for the complete expanded guide.

## Skill Structure

Each skill follows a consistent structure:

```
skills/
└── skill-name/
    ├── SKILL.md      # Quick reference with frontmatter metadata
    ├── AGENTS.md     # Complete expanded guide for agents
    └── rules/        # Individual rule files with examples
        ├── rule-1.md
        └── rule-2.md
```

### SKILL.md Format

```yaml
---
name: skill-name
description: When to apply this skill
license: MIT
metadata:
  author: community
  version: "1.0.0"
---
```

## Adding New Skills

1. Create a new directory in `skills/` using kebab-case naming
2. Add a `SKILL.md` with required frontmatter
3. Add an `AGENTS.md` with the complete expanded guide
4. Optionally add a `rules/` directory for individual rule files

See `AGENTS.md` for detailed conventions.

## License

MIT
