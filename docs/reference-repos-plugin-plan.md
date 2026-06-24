# Codex Reference Repos Plugin Plan

## Goal

Create a Codex plugin that packages a skill for project-declared reference repositories. When the
user asks Codex to inspect reference repos, the skill directs Codex to prepare local shallow clones
and inspect them with normal filesystem tools such as `rg`, `find`, and `sed`.

## V1 Contract

- The project source of truth is `references.toml` at the repository root.
- References are remote Git repositories only.
- Repository values accept any cloneable Git URL.
- V1 does not support branch, ref, tag, or local path fields.
- The plugin clones the remote default branch with `git clone --depth 1`.
- Clones are stored in the OS user cache, not in the project, not in temp, and not under `~/.codex`.
- Reference repos are read-only context. The plugin does not expose edit or execute operations.
- Cache behavior is lazy:
  - missing valid clone: clone on first prepare
  - existing valid clone: reuse
  - bad cache entry: delete and reclone
  - explicit refresh request: refresh/reclone
- On first reference-repo use, Codex clones all missing declared references.
- Codex should inspect cached repositories directly by path after prepare.

## `references.toml`

```toml
[references.kactis_ai]
repository = "https://github.com/Kactis/kactis-ai.git"
description = "Use for Kactis AI implementation patterns."
```

`description` is optional to match OpenCode, but the skill should encourage it because it helps
Codex choose relevant references.

## Plugin Capabilities

### Skill

The plugin skill tells Codex to use this plugin when the user asks to inspect or follow reference
repos. The workflow is:

1. Read root `references.toml`.
2. Create the OS cache root if it does not exist.
3. Clone missing references with `git clone --depth 1`.
4. Reuse valid cached clones.
5. Delete and reclone bad cache entries.
6. Use normal shell/file tools against the cached paths.
7. Treat reference repos as read-only examples.

### Future Tools

Add a CLI or MCP server only if skill-only usage proves too repetitive, inconsistent, or blocked by
permissions. V1 intentionally avoids both.

## Scaffold Structure

```text
codex-reference-repos/
  README.md
  docs/
    reference-repos-plugin-plan.md
  .agents/
    plugins/
      marketplace.json
  plugins/
    codex-reference-repos/
      .codex-plugin/
        plugin.json
      skills/
        reference-repos/
          SKILL.md
```

The repo-level marketplace lets the plugin be installed from this Git repository while keeping the
actual plugin bundle under `plugins/codex-reference-repos`.
