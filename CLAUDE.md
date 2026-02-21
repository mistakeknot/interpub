# interpub

Safe plugin publishing — bumps all version locations, validates sync, commits and pushes. See `AGENTS.md` for philosophy alignment protocol.

## Quick Commands

```bash
# Publish a new version (from inside any plugin repo)
/interpub:release 1.2.0

# Bump versions without publishing (standalone script)
bash scripts/bump-version.sh 1.2.0
```

## Design Decisions (Do Not Re-Ask)

- Discovers version locations automatically: plugin.json, marketplace.json, pyproject.toml, package.json, Cargo.toml
- Validates all versions are in sync before bumping
- Commits each repo separately, asks before every push
- Optionally runs plugin-validator agent before publishing
- Reminds about session restarts after publish (hooks/skills load at session start)
