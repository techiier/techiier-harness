# techiier-harness

Shared Claude Code setup for Linux (tested target: Arch Linux), packaged as a plugin marketplace.

## Requirements (Arch Linux)

```bash
sudo pacman -S --needed git
curl -fsSL https://claude.ai/install.sh | bash
```

The `/tag` and `/commit` commands only need `git` and coreutils (`date`).

## Install

Inside Claude Code:

```
/plugin marketplace add techiier/techiier-harness
/plugin install team-tools@techiier-harness
/reload-plugins
```

## What is included

`team-tools` plugin:

- Commands: `/tag`, `/commit`
- Skills: 39 shared skills (SDD workflow, .NET, Docker, SQL, PWA, review and commit helpers).

Skills from a plugin are namespaced, e.g. `team-tools:dotnet`.

## Optional: third-party plugins we use

```
/plugin marketplace add midudev/better-claude
/plugin install image-preview@better-claude
/plugin install sub-meter@better-claude
```

## Update

```
/plugin marketplace update techiier-harness
/reload-plugins
```

## Optional: prompt automatically on startup

Add to `~/.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "techiier-harness": {
      "source": { "source": "github", "repo": "techiier/techiier-harness" }
    }
  },
  "enabledPlugins": { "team-tools@techiier-harness": true }
}
```

## Not included

`CLAUDE.md` and `rules/*.md` are personal and cannot be shipped by a plugin.
