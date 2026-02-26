# interpub

Safe plugin publishing — delegates to `ic publish` for version bumping, marketplace sync, cache rebuild, and local state updates.

## Quick Commands

```bash
# From inside any plugin repo:
ic publish --patch              # Auto-increment patch version
ic publish <version>            # Bump to exact version
ic publish --dry-run --patch    # Preview without changes
ic publish doctor               # Detect drift and health issues
ic publish doctor --fix         # Auto-repair everything
ic publish status               # Show publish state for current plugin
ic publish status --all         # Show all plugins' publish health

# Legacy wrappers (both delegate to ic publish):
/interpub:release <version>
scripts/bump-version.sh <version>
```

## Design Decisions (Do Not Re-Ask)

- All publish logic lives in `core/intercore/internal/publish/` (Go)
- `/interpub:release` command is a thin wrapper around `ic publish`
- `auto-publish.sh` hook (in Clavain) calls `ic publish --auto` on git push
- Single version source: `plugin.json` only, derived files patched at publish time
- Never amends commits or force-pushes (unlike the old auto-publish.sh)
