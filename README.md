# interpub

Safe plugin publishing for Claude Code.

## What This Does

Plugins distributed through marketplace repositories have version strings in multiple files — `plugin.json`, `marketplace.json`, `pyproject.toml`, sometimes more. When they drift, the plugin appears installed but commands, skills, and hooks silently don't load. It's the kind of bug where everything looks fine until nothing works, and you spend 20 minutes wondering why your new skill isn't triggering before you realize `marketplace.json` still says `0.1.3`.

interpub bumps all version locations atomically, validates they're in sync, commits each repo, and asks before every push. The automation exists because manually keeping three version strings in sync across two repos is exactly the kind of thing humans mess up.

## Installation

First, add the [interagency marketplace](https://github.com/mistakeknot/interagency-marketplace) (one-time setup):

```bash
/plugin marketplace add mistakeknot/interagency-marketplace
```

Then install the plugin:

```bash
/plugin install interpub
```

## Usage

From inside any plugin repository:

```
/interpub:release 1.2.0
```

This walks through four phases:

1. **Discover** — finds all version locations (plugin.json, marketplace.json, pyproject.toml, package.json, Cargo.toml)
2. **Validate** — checks versions are in sync, optionally runs the plugin-validator agent
3. **Update** — bumps all versions, commits each repo (asks before every push)
4. **Verify** — confirms pushes succeeded, reminds about session restarts

## License

MIT
