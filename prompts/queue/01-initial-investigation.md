# Initial Investigation

## Objective

Research solutions to the following blocker:

> Claude Code has no built-in mechanism to permanently install a plugin from a local filesystem path. Only `--plugin-dir` (per-session) and marketplace install (requires public GitHub repo) are supported.

## What's Already Been Tried

- Checked settings.json, plugins directory structure, official docs
- Only two install methods confirmed: `--plugin-dir` and marketplace
- Private marketplace considered but raises visibility concerns

## Research Instructions

1. Search for official documentation, GitHub issues, forum threads, and blog posts related to this problem.
2. Identify all known approaches — even unconventional or experimental ones.
3. For each approach, assess: feasibility, complexity, trade-offs, and whether it's a permanent fix or a workaround.
4. Pay special attention to recent developments (last 6 months) — the landscape may have changed.
5. If relevant, check if there are open feature requests or RFCs that would solve this natively.
6. Specifically investigate:
   - Whether Claude Code supports private GitHub repos as marketplace sources
   - Whether the `~/.claude/plugins/` cache directory structure can be manually populated
   - Whether hooks (PreToolUse, PostToolUse, etc.) can be used to load plugins
   - Whether shell wrapper scripts are a viable long-term approach
   - Whether there are any open GitHub issues on anthropics/claude-code requesting this feature

## Output Format

Structure findings as:
- **Approach name**: one-line summary
  - How it works
  - Pros / cons
  - Complexity (low / medium / high)
  - Links / sources
