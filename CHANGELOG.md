# Changelog

## Unreleased

- Added an upstream drift check (`scripts/upstream/`, `.github/workflows/upstream-drift.yml`,
  `upstream.json`): a daily job compares the live, published `hitechcloudomnichannel` CLI and the live
  `hitechcloudomnichannel-mcp` default tool set against the docs in `skills/`, and opens/closes a single
  `upstream-drift`-labeled issue when they disagree. Run locally with `npm run check:upstream`.
  This is tooling only — it does not change any skill's published content or `version:`.

## 1.1.1

- `hitechcloudomnichannel` skill: moved the per-command catalog to `skills/hitechcloudomnichannel/references/commands.md` and
  kept a one-line-per-group index in `SKILL.md`. The 1.1.0 note below announced this move, but the
  file did not ship until now.
- Default MCP and CLI skill configuration to `https://app.hitechcloud.vn/api`; SaaS users now provide
  only an API key. Self-hosted instances still set their explicit `/api` URL.

## 1.1.0

- Removed the root `SKILL.md`. Skills now live only under `skills/`, so `npx skills add
  hitechcloud-vietnam/HiTechCloudOmnichannel-agent` lists both `hitechcloudomnichannel` and `hitechcloudomnichannel-mcp`, installs only the skill folder,
  and no longer needs `--full-depth`.
- `hitechcloudomnichannel` skill is CLI-only: the command catalog stays in `SKILL.md` (moved to `references/` in 1.1.1),
  agents discover flags with `--help`, and the skill auto-installs the CLI when it is missing.
- Both skills state when to switch to the other transport (MCP tools connected vs bulk/CLI work).
- Claude Code plugin now starts the hitechcloudomnichanneloudomnichannel MCP server and prompts for the API key and URL via
  `userConfig`, matching the Cursor, Grok, and Gemini manifests.
- README documents installs per agent (Claude Code, Cursor, Codex, Windsurf, Gemini CLI, Grok).
- Ignore local `npx skills add .` artifacts (`.agents/`, `.claude/`, `skills-lock.json`).

## 1.0.0

- Initial hitechcloudomnichanneloudomnichannel agent distribution repository.
- Published public `hitechcloudomnichannel` and `hitechcloudomnichannel-mcp` skills.
- Added a root `SKILL.md` (copy of `skills/hitechcloudomnichannel/SKILL.md`) so `npx skills add hitechcloud-vietnam/HiTechCloudOmnichannel-agent` installs the CLI skill directly.
- Added Cursor, Claude Code, Grok, Gemini, and generic MCP manifests.
