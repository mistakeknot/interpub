---
description: Publish a new plugin version across all version locations (plugin.json, marketplace.json, pyproject.toml, package.json) with validation
arguments:
  - name: version
    description: "Semver version to release (e.g. 1.2.0)"
    required: false
allowed-tools: ["Read", "Write", "Edit", "Grep", "Glob", "Bash", "AskUserQuestion", "Task"]
---

# Plugin Release Workflow

Guide the plugin author through publishing a new version safely. This command prevents the most common plugin publishing failure: **version drift** between `plugin.json` and `marketplace.json`.

When versions drift, the cache directory is named after the marketplace version but runtime loads from plugin.json — commands, skills, and hooks silently don't exist.

**Requested version:** $ARGUMENTS

---

## Phase 1: Discover Version Locations

**Goal**: Find all files that contain version strings for this plugin.

**Actions**:

1. Read `.claude-plugin/plugin.json` in the current working directory to get the plugin **name** and **current version**. If this file doesn't exist, stop and tell the user to `cd` into a plugin repo first.

2. Search for `marketplace.json` files that reference this plugin name. Check these locations using Glob:
   - `../*/.claude-plugin/marketplace.json` (sibling directories)
   - `../*-marketplace/.claude-plugin/marketplace.json` (marketplace-named siblings)
   For each found file, use Grep to check if it contains the plugin name. Only include marketplaces that list this plugin.

3. Check for language-specific version files in the current repo:
   - `pyproject.toml` — look for `version = "X.Y.Z"` at the top level
   - `package.json` — look for `"version": "X.Y.Z"`
   - `Cargo.toml` — look for `version = "X.Y.Z"` under `[package]`

4. Present all discovered locations in a table:

```
Version Locations Found:
────────────────────────────────────────────────────
  File                              Current Version
  .claude-plugin/plugin.json        0.5.0
  pyproject.toml                    0.5.0
  ../my-marketplace/marketplace.json  0.5.0
────────────────────────────────────────────────────
```

**If no version argument was provided** (i.e., `$ARGUMENTS` is empty): Ask the user what the new version should be, showing the current version for reference.

**If the version argument matches the current version**: Tell the user "Already at version X.Y.Z — nothing to do" and stop.

---

## Phase 2: Validate

**Goal**: Ensure the plugin is healthy before publishing.

**Actions**:

1. **Check version sync**: Verify all discovered versions match each other. If they DON'T match, display the mismatches prominently and ask the user:
   - "Existing version drift detected. Continue anyway and set all to `<new-version>`?" (using AskUserQuestion)
   - If user declines, stop.

2. **Run plugin-validator** (optional): Use the Task tool to launch the `plugin-dev:plugin-validator` agent on the current plugin directory. This catches structural issues (missing commands, invalid frontmatter, etc.). If the validator isn't available (plugin-dev not installed), skip this step gracefully and note it was skipped.

3. **Check for tests**: Look for `tests/`, `test/`, `scripts/test.sh`, or a test command in `pyproject.toml`/`package.json`. If found, ask the user if they want to run tests before releasing. Run them if confirmed.

**Output**: Validation summary. If critical issues were found by the validator, present them and ask whether to continue.

---

## Phase 3: Update Versions

**Goal**: Write the new version to all discovered locations and commit.

**Actions**:

1. **Update each version file** using the Edit tool, showing the before/after for each change:

   - For `plugin.json`: Edit the `"version": "old"` to `"version": "new"`
   - For `pyproject.toml`: Edit `version = "old"` to `version = "new"`
   - For `package.json`: Edit `"version": "old"` to `"version": "new"`
   - For `Cargo.toml`: Edit `version = "old"` to `version = "new"`
   - For `marketplace.json`: Find the entry for this plugin by name, then edit its `"version"` field. Be careful — marketplace.json contains multiple plugins, so match by plugin name first.

2. **Commit the plugin repo**: Stage all modified files in the current repo and commit with message:
   ```
   chore: bump version to <version>
   ```

3. **Commit marketplace repos**: For each marketplace repo that was modified:
   - Use AskUserQuestion to confirm: "Push changes to `<marketplace-repo-name>`?"
   - If confirmed, stage and commit with message:
     ```
     chore: bump <plugin-name> to v<version>
     ```

4. **Push** (with confirmation for EACH repo):
   - Ask: "Push `<repo-name>` to origin?" (using AskUserQuestion)
   - Only push if the user confirms
   - **Never auto-push. Always ask.**

**Output**: Summary of what was committed and pushed.

---

## Phase 4: Verify and Remind

**Goal**: Confirm the release succeeded.

**Actions**:

1. **Verify pushes**: For each repo that was pushed, run `git log --oneline -1` to confirm the commit is at HEAD.

2. **Check for CLI tools**: If `pyproject.toml` has `[project.scripts]` or `package.json` has `"bin"`, remind the user to reinstall the CLI tool:
   - Python: `uv tool install --force .`
   - Node: `npm install -g .`

3. **Final reminder**: Display:
   ```
   Release complete: <plugin-name> v<version>

   Remaining steps:
   - Restart Claude Code sessions to pick up the new plugin version
   - Users should run: claude plugin update <plugin-name>@<marketplace-name>
   ```

**Output**: Release summary with version number and any remaining manual steps.

---

## Common Errors

- **"No .claude-plugin/plugin.json found"**: You must run this command from within a plugin repository.
- **"Marketplace not found"**: The command searches sibling directories. If your marketplace repo is elsewhere, you'll need to update it manually.
- **Push rejected**: Usually means you need to pull first. The command will show the git error and suggest `git pull --rebase`.

## Next Step

After releasing, verify the install works: `claude plugin update <plugin-name>@<marketplace-name>` in a fresh session.
