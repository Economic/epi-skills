# Contributing to epi-skills

## Plugin structure

Each plugin lives in `plugins/` and follows this layout:

```
plugins/my-plugin/
  .claude-plugin/
    plugin.json             # Required. Plugin manifest.
  skills/
    my-skill/
      SKILL.md              # Required. Skill instructions for Claude.
      references/           # Optional. Supplementary docs, examples.
        some-reference.md
```

A plugin can contain multiple skills, hooks, MCP servers, and other components. At minimum it needs a `plugin.json` manifest and at least one skill.

### plugin.json

```json
{
  "name": "my-plugin",
  "version": "0.1.0",
  "description": "What this plugin does",
  "author": {
    "name": "Your Name"
  },
  "keywords": ["keyword1", "keyword2"]
}
```

### SKILL.md

Each skill directory must contain a `SKILL.md` with YAML frontmatter:

```markdown
---
name: my-skill-name
description: When to activate this skill. Claude uses this to decide relevance.
---

# my-skill-name

Content here is provided to Claude when the skill is activated.
Include data sources, API patterns, code examples, and conventions.
```

### Guidelines

- **Plugin name**: lowercase, hyphenated. Must match the directory name.
- **Skill description**: one or two sentences listing key trigger words so Claude knows when to activate the skill.
- Keep `SKILL.md` focused and concise. Put lengthy reference material in `references/`.
- Include working code examples where possible.

## Submitting a plugin

1. Fork this repository.
2. Create your plugin directory under `plugins/`.
3. Add `.claude-plugin/plugin.json` and at least one skill in `skills/`.
4. Register your plugin in `.claude-plugin/marketplace.json` at the repo root.
5. Update the table in `README.md`.
6. Open a pull request.
