# interpub — Vision and Philosophy

**Version:** 0.1.0
**Last updated:** 2026-02-28

## What interpub Is

interpub is the safe publishing layer for Interverse plugins. It wraps Intercore's Go publish pipeline (`internal/publish/`) behind three entrypoints — the `/interpub:release` skill command, the `ic publish` CLI, and the `scripts/bump-version.sh` shell fallback — plus a Clavain hook (`auto-publish.sh`) that fires on `git push`. Every publish event bumps all version locations (plugin.json, marketplace.json, any derived files), validates sync, rebuilds the marketplace cache, and updates local state before touching the remote. `plugin.json` is the single source of truth; derived files are always patched at publish time, never edited by hand.

The result is a single, durable action that produces a verifiable receipt: version X of plugin Y is in the marketplace, the cache is consistent, and the commit is on `main`. There is no partial publish state that can silently persist.

## Why This Exists

Publishing plugins by hand is a coordination problem. Version drift across `plugin.json`, `marketplace.json`, and `package.json` is silent and common. Pre-interpub, agents and humans both forgot steps, amended commits, and left the marketplace inconsistent with local state. interpub converts a multi-step, error-prone ritual into a single auditable action — so the evidence of a publish is the publish itself, not a developer's mental checklist.

## Design Principles

1. **One source of truth.** `plugin.json` owns the version. All other version references are derived at publish time. Editing derived files directly is always wrong.

2. **Publish as receipt.** A successful publish produces a durable, replayable artifact: a commit on `main`, an updated marketplace entry, and a rebuilt cache. If any step fails, the publish did not happen. Partial state is not a valid outcome.

3. **Defense in depth through separation.** Version bumping, marketplace sync, and cache rebuild are distinct, verifiable steps inside the pipeline. Each can fail and report independently. No single silent failure can leave the system in an inconsistent-but-plausible state.

4. **Strong defaults, overridable gates.** The default flow (`ic publish --patch`) is correct for the overwhelming majority of cases. Dry-run, doctor, and explicit version flags exist for the cases where it isn't. Overrides are explicit and leave a trail.

5. **Self-building.** Demarch publishes its own plugins with interpub. The tool's fitness is continuously validated by its own use. Agent friction with the publish workflow is a direct signal of technical debt in the pipeline.

## Scope

**Does:**
- Bump all version locations atomically from `plugin.json` as source of truth
- Validate marketplace sync and detect drift before committing
- Rebuild the marketplace cache as part of every publish
- Expose `ic publish doctor` and `ic publish status` for health inspection
- Fire automatically on `git push` via Clavain's `auto-publish.sh` hook
- Provide a dry-run mode for safe preview without side effects

**Does not:**
- Manage plugin authorship, ownership, or access control
- Handle npm/PyPI publishing (scope is the Interverse marketplace only)
- Amend commits or force-push
- Make decisions about what version to publish — the caller always decides

## Direction

- Harden the `doctor --fix` auto-repair path so drift is always recoverable without manual intervention
- Surface publish receipts as structured events that downstream tooling (interkasten, interwatch) can consume
- Extend `ic publish status --all` into a live dashboard for the full plugin fleet health
