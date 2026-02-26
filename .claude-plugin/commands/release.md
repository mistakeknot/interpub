---
description: Publish a new plugin version across all version locations (plugin.json, marketplace.json, pyproject.toml, package.json) with validation
arguments:
  - name: version
    description: "Semver version to release (e.g. 1.2.0), or --patch/--minor for auto-increment"
    required: false
allowed-tools: ["Read", "Bash"]
---

# Plugin Release Workflow

Delegates to `ic publish` — the Go-based publish pipeline in Intercore.

**Requested version:** $ARGUMENTS

---

## Instructions

1. Determine the publish command based on `$ARGUMENTS`:
   - If a version number (e.g., `1.2.0`): run `ic publish $ARGUMENTS`
   - If `--patch` or `--minor`: run `ic publish $ARGUMENTS`
   - If empty: run `ic publish --patch` (default to patch bump)

2. Run the command from the current working directory:
   ```bash
   ic publish <args>
   ```

3. If `ic` is not installed, tell the user:
   ```
   ic binary not found. Install from core/intercore:
     cd /home/mk/projects/Demarch/core/intercore && go build -o ~/.local/bin/ic ./cmd/ic/
   ```

4. Report the result. If the publish succeeded, remind:
   ```
   Restart Claude Code sessions to pick up the new plugin version.
   ```

## Dry Run

To preview what would happen without making changes:
```bash
ic publish <args> --dry-run
```

## Troubleshooting

- **Version drift detected**: Run `ic publish doctor` to see all drift, `ic publish doctor --fix` to auto-repair
- **Dirty worktree**: Commit or stash changes before publishing
- **Remote unreachable**: Check network connectivity and git remote configuration
- **Active publish in progress**: A previous publish was interrupted. Re-run to force, or check `ic publish status`
