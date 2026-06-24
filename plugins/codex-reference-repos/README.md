# Codex Reference Repos Plugin

This directory is the Codex plugin bundle.

## Contents

- `.codex-plugin/plugin.json`: plugin manifest
- `skills/reference-repos/SKILL.md`: skill-only reference repository workflow

The plugin intentionally has no MCP server or CLI in v1. The skill gives Codex the exact shell
workflow for reading `references.toml`, preparing OS-cache clones, and inspecting references
directly.
