# Codex Reference Repos

Codex Reference Repos is a Codex skill plugin for project-declared reference repositories.

The goal is to let a team check in a small `references.toml` file while keeping the referenced repositories out of the main repository. The skill tells Codex how to prepare shallow local clones in the OS cache and inspect them as read-only examples.

## V1 Shape

```toml
[references.kactis_ai]
repository = "https://github.com/Kactis/kactis-ai.git"
description = "Use for Kactis AI implementation patterns."
```

V1 intentionally keeps the contract small:

- root-level `references.toml`
- remote Git repositories only
- any cloneable Git URL
- no branch or ref field
- clone the remote default branch with `--depth 1`
- OS cache for cloned references
- direct Codex inspection through normal filesystem tools after prepare
- no MCP server or custom CLI in v1

See [docs/reference-repos-plugin-plan.md](docs/reference-repos-plugin-plan.md) for the current design notes.

## Repository Layout

```text
.agents/plugins/marketplace.json
plugins/codex-reference-repos/
  .codex-plugin/plugin.json
  skills/reference-repos/SKILL.md
```

The repo-level marketplace points Codex at `plugins/codex-reference-repos`.

## Status

This is a skill-only scaffold. The plugin manifest, marketplace entry, plan, and skill are in place.
