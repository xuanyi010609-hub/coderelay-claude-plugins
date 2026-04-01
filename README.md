# CodeRelay Claude Plugins

A standalone Claude Code marketplace repo for CodeRelay skills.

This repository intentionally contains **only Claude Code plugin files** for distribution. It does **not** include the CodeRelay application source code.

## Included plugin

- `coderelay`
  - Skill: `/coderelay:onboarding`
  - Purpose: fully set up CodeRelay from install to configuration to startup, with QR-first mobile connection.
  - Skill: `/coderelay:xiaban`
  - Purpose: quickly bring the current machine online, verify the agent is truly connected, and surface the mobile QR code directly in the visible chat reply.

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
/coderelay:xiaban
```

You can also pass a goal:

```text
/coderelay:onboarding 帮我装好 coderelay，配置好并启动，优先扫码连接
/coderelay:xiaban 帮我把这台机器挂在线，确认连上后把二维码直接发出来
```

## What these skills do

### `/coderelay:onboarding`

The onboarding skill is designed to **do the work proactively** instead of only listing commands.

Default flow:

1. Check whether Node.js, npm, and `coderelay` are available
2. Install or update `@xuanyi0609/coderelay` with `npm install -g` on first use
3. Check existing configuration with `coderelay status`
4. Only ask the user for required missing information, such as email for pairing
5. Run `coderelay pair` only when needed
6. Start the agent with `coderelay start`
7. Immediately output the QR code after successful startup whenever the environment can display it
8. Re-send the ASCII QR into the normal chat reply body in a fenced code block instead of relying only on collapsible tool output
9. Fall back to manual URL/token only if QR truly cannot be shown
10. Tell the user how to use `coderelay` afterwards

### `/coderelay:xiaban`

The xiaban skill is the shorter “go online now” path.

Default flow:

1. Check whether `coderelay` is already available and locally ready to go online
2. Start `coderelay start` in the background and only report success after the agent has actually connected and received hello ack
3. Run `coderelay connect-qr`
4. Re-send the ASCII QR into the visible chat reply body
5. Fall back to manual URL/token only if QR truly cannot be shown

## Repository layout

```text
.claude-plugin/marketplace.json
plugins/coderelay/.claude-plugin/plugin.json
plugins/coderelay/skills/onboarding/SKILL.md
plugins/coderelay/skills/xiaban/SKILL.md
```

## Notes

- This repo is distributed through a GitHub-based Claude Code marketplace.
- Users only need Claude Code with plugin support (`/plugin` command available).
- The main CodeRelay app is still distributed separately via npm as `@xuanyi0609/coderelay`.
