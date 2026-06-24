---
name: reference-repos
description: Use project-declared reference repositories as local read-only implementation examples. Use when the user asks Codex to use, inspect, prepare, sync, refresh, clone, or add reference repos declared in root references.toml, or asks to compare the current project against reference repository patterns.
---

# Reference Repos

Use root `references.toml` as the source of truth. Do not use MCP or a helper CLI for v1; run ordinary shell commands and inspect cached repositories directly.

## Config

References are declared in root-level `references.toml`:

```toml
[references.kactis_ai]
repository = "https://github.com/Kactis/kactis-ai.git"
description = "Use for Kactis AI implementation patterns."
```

`description` is optional. V1 supports only remote Git repositories. Do not add or rely on `branch`, `ref`, `tag`, or local `path`.

## Cache Root

Use the OS user cache, never temp, never the project directory, and never `~/.codex`.

```bash
case "$(uname -s)" in
  Darwin)
    CACHE_ROOT="$HOME/Library/Caches/codex-reference-repos"
    ;;
  Linux)
    CACHE_ROOT="${XDG_CACHE_HOME:-$HOME/.cache}/codex-reference-repos"
    ;;
  MINGW*|MSYS*|CYGWIN*)
    CACHE_ROOT="${LOCALAPPDATA:-$HOME/AppData/Local}/codex-reference-repos/Cache"
    ;;
  *)
    CACHE_ROOT="$HOME/.cache/codex-reference-repos"
    ;;
esac

mkdir -p "$CACHE_ROOT"
```

If the sandbox blocks creating or reading the cache root, request approval for the specific command.

## Prepare References

When the user asks to use reference repos, first prepare all missing references:

1. Resolve the project root with `git rev-parse --show-toplevel`.
2. Read `$ROOT/references.toml`. If it is missing, say no reference repos are configured.
3. Parse TOML with a structured parser. Prefer Python `tomllib` on Python 3.11+.
4. For each `[references.<alias>]`, require a string `repository` value.
5. Compute the cache path as `<cache-root>/<alias>-<first-12-sha256-of-repository>/repo`.
6. If the repo path is missing, run `git clone --depth 1 <repository> <repo-path>`.
7. If the repo path exists and has a `.git` directory with matching `origin`, reuse it.
8. If the repo path exists but is not a Git repo or has the wrong origin, remove that cache entry and reclone.
9. Do not fetch, pull, or refresh an existing valid clone unless the user explicitly asks to sync, update, refresh, or reclone.
10. Treat all cached reference repos as read-only. Do not edit, execute project code from, or apply patches inside them.

Use this parser snippet when useful:

```bash
ROOT="$(git rev-parse --show-toplevel)"
python3 - "$ROOT/references.toml" "$CACHE_ROOT" <<'PY'
import hashlib
import json
import re
import sys
import tomllib
from pathlib import Path

config_path = Path(sys.argv[1])
cache_root = Path(sys.argv[2])

if not config_path.exists():
    print(json.dumps({"references": []}))
    raise SystemExit

data = tomllib.loads(config_path.read_text())
references = []
for alias, entry in data.get("references", {}).items():
    if not isinstance(entry, dict):
        continue
    repository = entry.get("repository")
    if not isinstance(repository, str) or not repository.strip():
        continue
    safe_alias = re.sub(r"[^A-Za-z0-9_.-]+", "-", alias).strip("-") or "reference"
    digest = hashlib.sha256(repository.encode("utf-8")).hexdigest()[:12]
    references.append({
        "alias": alias,
        "repository": repository,
        "description": entry.get("description"),
        "path": str(cache_root / f"{safe_alias}-{digest}" / "repo"),
    })

print(json.dumps({"references": references}, indent=2))
PY
```

After preparing, inspect references with normal tools:

```bash
rg -n "auth|session|login" "/path/to/cached/reference/repo"
find "/path/to/cached/reference/repo" -maxdepth 3 -type f | sed -n '1,120p'
sed -n '1,180p' "/path/to/cached/reference/repo/path/to/file"
```

## Refresh References

Refresh only when the user explicitly asks. Prefer delete-and-reclone for v1 because clones are shallow and disposable:

```bash
rm -rf "/path/to/cache-entry"
git clone --depth 1 "<repository>" "/path/to/cache-entry/repo"
```

Do not refresh during ordinary "use the reference repos" requests.

## Add A Reference

When the user asks to add a reference:

1. Add or update a table in root `references.toml`.
2. Keep `description` optional but ask for or suggest one when it would help future agents.
3. Clone the new reference into the OS cache using the same cache path rules.

Example:

```toml
[references.opencode]
repository = "https://github.com/sst/opencode.git"
description = "Use for agent UX and reference repository workflow patterns."
```
