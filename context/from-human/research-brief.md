# Research Brief

## Blocker

Claude Code has no built-in mechanism to permanently install a plugin from a local filesystem path. The only options are `--plugin-dir` (per-session only) or publishing through a marketplace (requires a public GitHub repo). This makes it impossible to maintain private, local-only plugins that persist across sessions without workarounds.

## Date Captured

2026-04-10

## What Was Tried

- Checked `~/.claude/settings.json` for a plugins configuration key — the `plugins` object exists but is empty and undocumented for local paths
- Checked for `~/.claude/plugins/local/` directory — does not exist
- Read the official Claude Code plugin docs at code.claude.com — confirmed only two installation methods: `--plugin-dir` (per-session) and marketplace install
- Attempted to find a paywalled guide on Substack (Alex McFarland) but the technical details were behind a paywall
- Considered creating a private marketplace repo, but the marketplace itself is public so plugin entries would be visible even if source repos are private

## Potential Directions

- **Shell alias / wrapper script**: Alias `claude` to always include `--plugin-dir` flags for local plugins. Simple but fragile — breaks if Claude Code updates its CLI interface.
- **Private marketplace**: Create a second, private GitHub marketplace repo just for private plugins. Unclear if Claude Code supports multiple marketplaces or private marketplace repos.
- **Startup hook**: Use Claude Code hooks to auto-load plugins on session start. Needs investigation — may not be possible with current hook types.
- **Symlink into marketplace cache**: Symlink local plugin directories into `~/.claude/plugins/marketplaces/` to trick the cache. Hacky but might work.
- **Feature request**: File a GitHub issue or feature request for native local plugin support. Long-term but the right solution.
- **`.claude/settings.json` undocumented fields**: There may be undocumented settings fields for local plugin paths that aren't in the public docs.

## Key Questions

1. Is there any undocumented mechanism in Claude Code for persistent local plugin installation?
2. Can Claude Code be configured to use a private GitHub repository as a marketplace?
3. What is the most robust workaround until native support arrives — alias, symlink, or something else?

## Desired Output

A clear recommendation with supporting evidence — ideally actionable steps to persistently install local/private plugins across Claude Code sessions.
