# Local Claude Code Plugin Installation: Comprehensive Research

**Date:** 2026-04-10
**Researcher:** Claude (Opus 4.6)
**Status:** Complete

## Key Findings

The problem statement -- "Claude Code has no built-in mechanism to permanently install a plugin from a local filesystem path" -- is **partially incorrect as of the current version**. Claude Code **does** support local filesystem-based persistent plugin installation via the `extraKnownMarketplaces` directory source type. However, this mechanism has bugs and rough edges that make it unreliable for some users. Below is a detailed breakdown of every approach found.

---

## Approach 1: Local Directory Marketplace via `extraKnownMarketplaces` (RECOMMENDED)

### How It Works

Claude Code's `extraKnownMarketplaces` setting supports a `"source": "directory"` type that points to a local filesystem path. This is **already configured** on this system:

```json
{
  "extraKnownMarketplaces": {
    "danielrosehill": {
      "source": {
        "source": "directory",
        "path": "/home/daniel/repos/cc-plugins/marketplace"
      },
      "autoUpdate": true
    }
  }
}
```

The marketplace at that path must contain a `.claude-plugin/marketplace.json` file listing plugins. Each plugin entry can use a `"source": "github"` reference (pointing to a GitHub repo) or a relative path reference (`"source": "./plugin-name"`).

### For Fully Local/Private Plugins

To keep plugins entirely local (no GitHub repo needed), structure the marketplace like this:

```
/path/to/local-marketplace/
  .claude-plugin/
    marketplace.json
  my-private-plugin/
    .claude-plugin/
      plugin.json
    skills/
      my-skill/
        SKILL.md
    commands/
    agents/
    hooks/
```

In `marketplace.json`, reference plugins with relative paths:

```json
{
  "name": "my-local-marketplace",
  "plugins": [
    {
      "name": "my-private-plugin",
      "source": "./my-private-plugin",
      "description": "A private plugin",
      "version": "1.0.0"
    }
  ]
}
```

Then install via `/plugin install my-private-plugin@my-local-marketplace`.

### Assessment

| Criterion | Rating |
|-----------|--------|
| **Feasibility** | Works, but has known bugs (see below) |
| **Complexity** | Low -- just a directory with JSON files |
| **Durability** | High -- this is an officially documented feature |
| **Trade-offs** | Plugins are cached to `~/.claude/plugins/cache/`, so edits to the source don't automatically propagate. Must use `/plugin marketplace update` or reinstall. |

### Known Bugs

1. **Issue #12457**: `claude plugin install` succeeds but fails to write to `installed_plugins.json` for local directory marketplaces (reported v2.0.54, workaround: manually edit `installed_plugins.json`)
2. **Issue #23978**: Relative paths in `extraKnownMarketplaces` with `"source": "directory"` don't resolve correctly -- must use absolute paths
3. **Issue #11278**: Plugin path resolution incorrectly uses `marketplace.json` file path instead of marketplace directory

---

## Approach 2: Shell Wrapper / Alias with `--plugin-dir`

### How It Works

Wrap the `claude` command in a shell function or alias that always includes `--plugin-dir` flags:

```bash
# ~/.bashrc or ~/.zshrc
claude() {
    command claude \
        --plugin-dir ~/my-plugins/plugin-one \
        --plugin-dir ~/my-plugins/plugin-two \
        "$@"
}
```

Or a more sophisticated conditional wrapper:

```bash
claude-with-plugins() {
    PLUGIN_ARGS=()
    if [[ "$1" == "--with-skill-creator" ]]; then
        PLUGIN_ARGS+=(--plugin-dir /path/to/skill-creator)
        shift
    fi
    command claude "${PLUGIN_ARGS[@]}" "$@"
}
```

### Assessment

| Criterion | Rating |
|-----------|--------|
| **Feasibility** | Works reliably |
| **Complexity** | Low |
| **Durability** | Medium -- could break if `--plugin-dir` flag changes, but this is unlikely |
| **Trade-offs** | Does NOT work with VS Code extension (only CLI). Plugins loaded via `--plugin-dir` bypass the cache entirely -- reads directly from source, which is actually an advantage for development. However, there's a bug (#37630) where `--plugin-dir` assigns plugins to an `inline` marketplace that can't be registered, causing Component errors in the plugin UI. |

### Key Advantage

`--plugin-dir` reads directly from the filesystem with no caching. Edits to plugin files are immediately available after `/reload-plugins`. This is the only approach that supports true "live editing" of plugins.

---

## Approach 3: Private GitHub Marketplace

### How It Works

Create a private GitHub repository containing the marketplace structure, then add it via `extraKnownMarketplaces`:

```json
{
  "extraKnownMarketplaces": {
    "private-plugins": {
      "source": {
        "source": "github",
        "repo": "your-org/private-claude-plugins"
      }
    }
  }
}
```

### Assessment

| Criterion | Rating |
|-----------|--------|
| **Feasibility** | Partially works -- private repos have Git credential complications |
| **Complexity** | Medium -- requires Git credential manager to be properly configured |
| **Durability** | High -- this is the standard supported path |
| **Trade-offs** | The marketplace catalog (`marketplace.json`) must be in the private repo, but individual plugin entries can reference other private repos. Authentication requires `gh` CLI or Git credential manager to have access. Multiple blog posts (including Dominic Bottger's guide) recommend cloning the private repo locally and using the `"source": "directory"` approach instead to avoid credential headaches. |

### Recommended Alternative

Clone the private repo locally and use Approach 1 instead:

```bash
git clone git@github.com:your-org/private-plugins.git ~/private-plugins
```

Then point `extraKnownMarketplaces` at the local clone with `"source": "directory"`.

---

## Approach 4: Inline Settings Marketplace (`"source": "settings"`)

### How It Works

The `extraKnownMarketplaces` setting supports a `"source": "settings"` type that declares plugins inline directly in `settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "team-tools": {
      "source": {
        "source": "settings",
        "name": "team-tools",
        "plugins": [
          {
            "name": "code-formatter",
            "source": {
              "source": "github",
              "repo": "acme-corp/code-formatter"
            }
          }
        ]
      }
    }
  }
}
```

### Assessment

| Criterion | Rating |
|-----------|--------|
| **Feasibility** | Works for plugins that have a remote source (GitHub/Git) |
| **Complexity** | Low -- just JSON in settings |
| **Durability** | High -- officially supported |
| **Trade-offs** | Each plugin entry still needs a `source` pointing to a GitHub repo, Git URL, or npm package. You cannot point to a local filesystem path as a plugin source within a settings-type marketplace. This approach is useful for curating a set of plugins from various sources without maintaining a separate marketplace repo, but does NOT solve the "fully local plugin" problem. |

---

## Approach 5: Manual Cache Population / Symlinks

### How It Works

Manually populate `~/.claude/plugins/cache/` with plugin directories and edit `installed_plugins.json` to register them:

```json
{
  "version": 2,
  "plugins": {
    "my-plugin@my-marketplace": [{
      "scope": "user",
      "installPath": "/path/to/plugin",
      "version": "1.0.0",
      "installedAt": "2026-04-10T00:00:00.000Z",
      "lastUpdated": "2026-04-10T00:00:00.000Z"
    }]
  }
}
```

Or symlink a local plugin directory into the cache:

```bash
ln -s /home/user/my-plugin ~/.claude/plugins/cache/my-marketplace/my-plugin/1.0.0
```

### Assessment

| Criterion | Rating |
|-----------|--------|
| **Feasibility** | Works as a workaround |
| **Complexity** | Medium -- requires understanding the internal cache format |
| **Durability** | Low -- cache format is undocumented and may change between versions. Plugin updates or reinstalls will overwrite manual entries. |
| **Trade-offs** | This was mentioned as a workaround in GitHub issue #12457. Fragile and not recommended for long-term use. |

---

## Approach 6: Hooks as Plugin Simulation

### How It Works

Claude Code hooks (`PreToolUse`, `PostToolUse`, `PreRequest`, `PostRequest`, `Notification`, etc.) can be configured in `settings.json` to run custom scripts on specific events. In theory, you could replicate some plugin behavior (validation, linting, policy enforcement) through hooks alone.

### Assessment

| Criterion | Rating |
|-----------|--------|
| **Feasibility** | Partially -- hooks can run arbitrary commands but cannot inject skills, agents, or MCP servers |
| **Complexity** | High -- requires scripting each behavior individually |
| **Durability** | High -- hooks are a stable, documented feature |
| **Trade-offs** | Hooks are NOT a replacement for plugins. They cannot provide: slash commands/skills, subagents, MCP server configurations, LSP server configurations, or bin/ PATH additions. Hooks are useful for validation, linting, and automation triggers, but they are a complementary mechanism, not an alternative to plugins. |

---

## Approach 7: Persistent `--plugin-dir` via Settings (FEATURE REQUEST -- NOT YET IMPLEMENTED)

### Status

**Open feature request: [Issue #17663](https://github.com/anthropics/claude-code/issues/17663)** -- "Persistent in-place plugin loading (no cache)"

This is the canonical issue requesting a `pluginDirs` setting in `settings.json` that would work like `--plugin-dir` but persist across sessions. The issue was opened requesting that `--plugin-dir` paths be persistable via something like `claude config add plugin_dirs ~/projects/my-plugin`. Multiple users have upvoted this, including VS Code extension users who cannot use `--plugin-dir` at all.

Issue #27287 (same request, closed as duplicate of #17663) explicitly states: "Could `--plugin-dir` paths be persisted in settings so they survive across sessions?"

### Assessment

| Criterion | Rating |
|-----------|--------|
| **Feasibility** | Not yet available |
| **Complexity** | Would be trivial to use if implemented |
| **Durability** | Would be the ideal solution |
| **Trade-offs** | No ETA from Anthropic. The issue is open and has community support. |

---

## Summary Comparison

| Approach | Fully Local? | Persistent? | Live Editing? | VS Code? | Complexity | Durability |
|----------|:---:|:---:|:---:|:---:|:---:|:---:|
| 1. Directory Marketplace | Yes | Yes | No (cached) | Yes | Low | High |
| 2. Shell Wrapper | Yes | Yes* | Yes | No | Low | Medium |
| 3. Private GitHub | No | Yes | No | Yes | Medium | High |
| 4. Settings Inline | No | Yes | No | Yes | Low | High |
| 5. Manual Cache | Yes | Fragile | Via symlink | Yes | Medium | Low |
| 6. Hooks | N/A | Yes | Yes | Yes | High | High |
| 7. Persistent pluginDirs | Yes | Yes | Yes | Yes | Trivial | -- |

*Shell wrapper persistence depends on always launching via CLI.

---

## Recommended Strategy

### For Your Use Case (Local/Private Plugins, Persistent Across Sessions)

**Primary: Approach 1 (Local Directory Marketplace)** -- This is what you are already partially doing with `extraKnownMarketplaces`. The key refinement is:

1. Ensure the marketplace directory exists at the configured path with a valid `.claude-plugin/marketplace.json`
2. For truly local-only plugins (no GitHub repo), use relative path sources in `marketplace.json` (`"source": "./plugin-name"`)
3. Use absolute paths in `extraKnownMarketplaces` (issue #23978 confirms relative paths don't work)
4. After installation, verify the entry appears in `~/.claude/plugins/installed_plugins.json` -- if not (bug #12457), manually add it

**Secondary: Approach 2 (Shell Wrapper)** -- For plugins you are actively developing and want live editing without the cache, add a shell function that includes `--plugin-dir` for development plugins. This complements Approach 1 (use the marketplace for stable plugins, use `--plugin-dir` for plugins under active development).

**Watch: Approach 7 (Persistent pluginDirs)** -- Upvote [issue #17663](https://github.com/anthropics/claude-code/issues/17663). If implemented, this would be the clean solution that eliminates all workarounds.

---

## Sources

- [Claude Code Plugin Documentation](https://code.claude.com/docs/en/plugins) -- official plugin creation docs
- [Discover and Install Plugins](https://code.claude.com/docs/en/discover-plugins) -- marketplace installation docs
- [Create and Distribute a Plugin Marketplace](https://code.claude.com/docs/en/plugin-marketplaces) -- marketplace creation docs
- [Claude Code Settings Reference](https://code.claude.com/docs/en/settings) -- extraKnownMarketplaces and enabledPlugins docs
- [Issue #17663: Persistent in-place plugin loading](https://github.com/anthropics/claude-code/issues/17663) -- canonical feature request (OPEN)
- [Issue #27287: Persistent --plugin-dir](https://github.com/anthropics/claude-code/issues/27287) -- duplicate of #17663 (CLOSED)
- [Issue #12457: Plugin install fails to persist for local directory marketplaces](https://github.com/anthropics/claude-code/issues/12457) -- known bug (CLOSED)
- [Issue #37630: --plugin-dir inline marketplace bug](https://github.com/anthropics/claude-code/issues/37630) -- inline marketplace resolution bug (CLOSED as duplicate)
- [Issue #23978: Directory source doesn't resolve relative paths](https://github.com/anthropics/claude-code/issues/23978) -- relative path bug
- [Issue #11278: Plugin path resolution bug](https://github.com/anthropics/claude-code/issues/11278) -- path resolution bug
- [Conditionally Loading Plugins Gist](https://gist.github.com/gwpl/776057bec49c47c1327afda07fcc75d2) -- shell wrapper examples
- [Building a Private Claude Code Plugin Marketplace](https://dominic-boettger.com/blog/claude-code-private-plugin-marketplace-guide/) -- private marketplace guide
- [Creating Local Claude Code Plugins](https://somethinghitme.com/2026/01/31/creating-local-claude-code-plugins/) -- step-by-step local plugin tutorial
- [Claude Code Hooks Reference](https://code.claude.com/docs/en/hooks) -- hooks documentation
- [Claude Code Plugins Reference](https://code.claude.com/docs/en/plugins-reference) -- complete technical reference
