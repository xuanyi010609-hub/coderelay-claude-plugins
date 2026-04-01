# CodeRelay Claude Plugins

A standalone Claude Code marketplace repo for CodeRelay skills.

This repository intentionally contains **only Claude Code plugin files** for distribution. It does **not** include the CodeRelay application source code.

## Included plugin

- `coderelay`
  - Skill: `/coderelay:onboarding`
  - Purpose: fully set up CodeRelay from install to configuration to startup, with QR-first mobile connection.

## Install

Inside Claude Code:

```text
/plugin marketplace add xuanyi010609-hub/coderelay-claude-plugins
/plugin install coderelay@coderelay-marketplace
/reload-plugins
```

Then use:

```text
/coderelay:onboarding
```

You can also pass a goal:

```text
/coderelay:onboarding 帮我装好 coderelay，配置好并启动，优先扫码连接
```

## What this skill does

The onboarding skill is designed to **do the work proactively** instead of only listing commands.

Default flow:

1. Check whether Node.js, npm, and `coderelay` are available
2. Install or update `@xuanyi0609/coderelay` with `npm install -g` on first use
3. Check existing configuration with `coderelay status`
4. Only ask the user for required missing information, such as email for pairing
5. Run `coderelay pair` only when needed
6. Start the agent with `coderelay start`
7. Prefer QR output for mobile connection, and fall back to manual URL/token only if necessary
8. Tell the user how to use `coderelay` afterwards

## Repository layout

```text
.claude-plugin/marketplace.json
plugins/coderelay/.claude-plugin/plugin.json
plugins/coderelay/skills/onboarding/SKILL.md
```

## Notes

- This repo is distributed through a GitHub-based Claude Code marketplace.
- Users only need Claude Code with plugin support (`/plugin` command available).
- The main CodeRelay app is still distributed separately via npm as `@xuanyi0609/coderelay`.
