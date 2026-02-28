---
description: Scan all plugins for version drift between local repos, marketplace, and cache — publish anything stale
arguments: []
allowed-tools: ["Bash", "Read"]
---

# Publish Sweep

Scan every plugin repo, compare versions across local/marketplace/cache, and publish stale ones.

**Announce:** "Running publish sweep across all plugins."

## Step 1: Detect Stale Plugins

Run this scan to find plugins where the local repo has unpushed commits or version drift:

```bash
DEMARCH_ROOT="/home/mk/projects/Demarch"
MARKETPLACE="$DEMARCH_ROOT/core/marketplace/.claude-plugin/marketplace.json"
CACHE_ROOT="$HOME/.claude/plugins/cache/interagency-marketplace"

stale=()
for dir in "$DEMARCH_ROOT"/interverse/*/ "$DEMARCH_ROOT"/os/clavain/; do
    [ -d "$dir/.claude-plugin" ] || continue
    name=$(basename "$dir")
    cd "$dir"

    # Local version from plugin.json
    local_ver=$(python3 -c "import json; print(json.load(open('.claude-plugin/plugin.json'))['version'])" 2>/dev/null) || continue

    # Marketplace version
    market_ver=$(python3 -c "
import json, sys
m = json.load(open('$MARKETPLACE'))
plugins = m if isinstance(m, list) else m.get('plugins', [])
match = [p for p in plugins if p['name'] == '$name']
print(match[0]['version'] if match else 'none')
" 2>/dev/null) || market_ver="none"

    # Cache version (directory name)
    cache_ver_dir=$(ls -d "$CACHE_ROOT/$name"/*/ 2>/dev/null | head -1)
    cache_ver=$(basename "$cache_ver_dir" 2>/dev/null) || cache_ver="none"

    # Unpushed commits
    ahead=$(git log origin/main..HEAD --oneline 2>/dev/null | wc -l)

    # Uncommitted changes (excluding untracked noise)
    dirty=$(git status --porcelain 2>/dev/null | grep -v '^?? \.clavain' | grep -v '^?? \.tldrs' | grep -v '^?? \.gemini' | grep -v '^?? docs/$' | grep -v '^?? \.claude/' | wc -l)

    is_stale=false
    reasons=""
    [ "$local_ver" != "$market_ver" ] && is_stale=true && reasons="$reasons market=$market_ver"
    [ "$local_ver" != "$cache_ver" ] && is_stale=true && reasons="$reasons cache=$cache_ver"
    [ "$ahead" -gt 0 ] && is_stale=true && reasons="$reasons ahead=$ahead"
    [ "$dirty" -gt 0 ] && is_stale=true && reasons="$reasons dirty=$dirty"

    if $is_stale; then
        echo "STALE $name v$local_ver:$reasons"
    fi
done
```

## Step 2: Present Findings

Display a table of stale plugins with their drift reasons:

```
Publish Sweep — N stale plugins found

| Plugin | Local | Market | Cache | Reason |
|--------|-------|--------|-------|--------|
| ...    | ...   | ...    | ...   | ...    |
```

If zero stale: report "All plugins in sync." and stop.

If any have `dirty` changes, warn: "These plugins have uncommitted changes — commit first, or they'll publish without the latest edits."

If any have `ahead` commits, note: "These have unpushed commits that will be pushed during publish."

## Step 3: Confirm

Use **AskUserQuestion**:

> "Found N stale plugins. Publish all?"

Options:
1. **"Publish all (Recommended)"** — run `ic publish --patch` for each stale plugin
2. **"Pick which to publish"** — multi-select from the stale list
3. **"Dry run"** — show what would happen without publishing

## Step 4: Publish

For each plugin to publish, run from its directory:

```bash
cd "$plugin_dir" && ic publish --patch 2>&1
```

Run sequentially (not parallel) — `ic publish` modifies the shared marketplace repo and can conflict.

After each publish, report: `plugin vX.Y.Z → vX.Y.Z+1`

If a publish fails, report the error and continue to the next plugin. Collect all failures for the summary.

## Step 5: Sync Cache

After all publishes, sync the cache for any plugin where the cache version still doesn't match:

```bash
for name in <published_plugins>; do
    local_ver=$(python3 -c "import json; print(json.load(open('$plugin_dir/.claude-plugin/plugin.json'))['version'])" 2>/dev/null)
    cache_dir="$CACHE_ROOT/$name/$local_ver"
    mkdir -p "$cache_dir"
    rsync -a --delete --exclude='.git' --exclude='.clavain' --exclude='.tldrs' --exclude='node_modules' --exclude='__pycache__' --exclude='.venv' "$plugin_dir/" "$cache_dir/"
    # Remove old version dirs
    for old in "$CACHE_ROOT/$name"/*/; do
        [ "$(basename "$old")" != "$local_ver" ] && rm -rf "$old"
    done
done
```

## Step 6: Summary

```
Publish sweep complete!

Published: N plugins
  plugin-a: 0.1.0 → 0.1.1
  plugin-b: 0.2.3 → 0.2.4

Failed: M plugins (if any)
  plugin-c: <error message>

Cache synced: N plugins
Restart Claude Code sessions to pick up new versions.
```
