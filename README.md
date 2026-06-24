# Reference Repos

## What this is

- A Codex plugin for checked-in `references.toml` files that tell agents which remote repos to inspect as implementation examples.
- It keeps reference repos out of your project history by cloning shallow, read-only copies into the OS user cache.
- It is intentionally skill-only for v1: no MCP server, no custom CLI, and no background refresh behavior.

## Setup

Paste this prompt into your agent to help set this up in a project:

```text
Set up the Codex Reference Repos plugin for this repository and help me get started:
https://raw.githubusercontent.com/lukamircetic/codex-reference-repos/refs/heads/main/README.md
```

Commands to install the plugin:

```bash
codex plugin marketplace add lukamircetic/codex-reference-repos
codex plugin add codex-reference-repos@reference-repos
```

After install, ask Codex to add a reference to your project:

```text
$reference-repos Add this repo to the project references https://github.com/Effect-TS/effect-smol.git
```

The `references.toml` will look something like this:

```toml
[references.effect]
repository = "https://github.com/Effect-TS/effect-smol.git"
description = "Use for referencing effect best practice patterns"
```

Now you can reference it using the skill:
```text
$reference-repos can you double check we're following the patterns from the reference repos
```

Repos are cloned with `git clone --depth 1` using the remote default branch. Cached repos are saved outside your project: macOS uses `~/Library/Caches/codex-reference-repos`, Linux uses `${XDG_CACHE_HOME:-$HOME/.cache}/codex-reference-repos`, and Windows uses `%LOCALAPPDATA%\codex-reference-repos\Cache`. Existing valid clones are reused; the plugin only refreshes when you explicitly ask.


## Inspiration

Inspired by opencode references, check them out: https://opencode.ai/docs/references/