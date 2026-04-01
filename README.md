# CodeRelay Claude Plugins

A standalone Claude Code marketplace repo for CodeRelay skills.

This repository intentionally contains **only Claude Code plugin files** for distribution. It does **not** include the CodeRelay application source code.

## Included plugin

- `coderelay`
  - Skill: `/coderelay:onboarding`
  - Purpose: guide users through CodeRelay install, pairing, startup, diagnosis, and QR-first mobile connection.

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
/coderelay:onboarding 帮我把电脑接到手机上，优先扫码
```

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
