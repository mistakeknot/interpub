# interpub

Safe plugin publishing for Claude Code. Bumps all version locations, validates sync, commits and pushes with confirmation.

## The Problem

Plugins distributed through marketplace repositories have version strings in multiple files (`plugin.json`, `marketplace.json`, `pyproject.toml`, etc.) that must stay in sync. When they drift, the plugin appears installed but commands, skills, and hooks silently don't load. See [version drift failure mode](https://github.com/mistakeknot/tldr-swinton/blob/main/docs/solutions/build-errors/plugin-version-drift-breaks-loading.md).

## Install

```bash
claude plugin marketplace add mistakeknot/interagency-marketplace
claude plugin install interpub@interagency-marketplace
```

## Usage

From inside any plugin repository:

```
/interpub:release 1.2.0
```

This guides you through:

1. **Discover** — finds all version locations (plugin.json, marketplace.json, pyproject.toml, package.json, Cargo.toml)
2. **Validate** — checks versions are in sync, optionally runs plugin-validator
3. **Update** — bumps all versions, commits each repo (asks before every push)
4. **Verify** — confirms pushes succeeded, reminds about session restarts

## License

MIT
