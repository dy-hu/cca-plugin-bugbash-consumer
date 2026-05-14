# cca-plugin-bugbash-consumer

Consumer repo for the [CCA plugins bug bash](https://github.com/github/sweagentd/issues/11889).

The plugin under test lives at [`dy-hu/cca-plugin-bugbash`](https://github.com/dy-hu/cca-plugin-bugbash) and is enabled via `.github/copilot/settings.json`. It bundles one custom agent (Rapper), one slash command (poetry-command), one skill (rapping-skill), and a second agent with MCP (FigmaGetter).

Each surface prints a unique `DISPATCH:...` line plus a per-line marker, so the agent output alone tells you which surface fired:

| Surface | Trigger phrase | Dispatch line | Per-line marker |
|---|---|---|---|
| Rapper agent | "write a rap song about X" | `DISPATCH:agent:Rapper` | `AGENT-RAPPER-FOSHIZZLE` |
| poetry-command | "write a poem about X" or `/poetry-command` | `DISPATCH:command:poetry` | `COMMAND-POETRY-HAIKU` |
| rapping-skill | "write a limerick about X" | `DISPATCH:skill:rapping` | `SKILL-RAPPER-WORDUP` |
| FigmaGetter agent | Figma-related ask | `DISPATCH:agent:FigmaGetter` | `AGENT-FIGMA-SHAZAM` |

## Running a case

Edit `.github/copilot/settings.json` for the setup the row needs, open an issue with the trigger phrase, and assign Copilot. The session log shows the plugin clone in setup and the dispatch line in the agent loop. If a row uncovers a bug, file a one-liner sub-issue under [#11889](https://github.com/github/sweagentd/issues/11889) and self-assign.

## Test matrix

### Settings loading
- [ ] Repo-level `.github/copilot/settings.json` loads (baseline)
- [ ] Org-level `copilot/settings.json` in `dy-hu/.github` loads when repo settings are absent
- [ ] Repo entry wins over org entry for the same spec
- [ ] Malformed `settings.json` does not crash the job; a warning surfaces
- [ ] `{"enabledPlugins": {"<spec>": false}}` skips the plugin (no clone in setup logs)

### Spec parsing and resolution
- [ ] `dy-hu/cca-plugin-bugbash` (owner/repo)
- [ ] `dy-hu/cca-plugin-bugbash@<sha>` (pinned commit)
- [ ] `dy-hu/cca-plugin-bugbash@some-branch` (branch ref)
- [ ] `https://github.com/dy-hu/cca-plugin-bugbash.git` (HTTPS with .git)
- [ ] `git@github.com:dy-hu/cca-plugin-bugbash.git` (SSH)
- [ ] `https://gitlab.com/foo/bar` rejected cleanly
- [ ] `./local-plugin` (local path checked into consumer repo)
- [ ] `../escape-attempt` traversal rejected
- [ ] Empty string, whitespace, trailing slash handled gracefully

### Marketplace
- [ ] `plugin@marketplace` resolves via `extraKnownMarketplaces`
- [ ] Bare plugin name resolves via the default marketplace
- [ ] Missing plugin, missing ref, malformed `marketplace.json` all fail cleanly
- [ ] Marketplace outside the allowlist is blocked under `strictKnownMarketplaces`

### Plugin loading
- [ ] `plugin.json` at repo root is picked up
- [ ] `plugin.json` in a subpath is picked up (needs a second fixture repo)
- [ ] Plugin's `.mcp.json` merges into session MCP config
- [ ] Plugin's `.lsp.json` registers LSP servers
- [ ] Rapper appears in the agent's `task` tool agent list

### Dispatch (the markers do the verification)
- [ ] "Write a rap about CI and save to OUT.md" → `DISPATCH:agent:Rapper` + every line has `AGENT-RAPPER-FOSHIZZLE`
- [ ] "Write a poem about CI and save to OUT.md" → `DISPATCH:command:poetry` + every line has `COMMAND-POETRY-HAIKU`
- [ ] "Write a limerick about CI and save to OUT.md" → `DISPATCH:skill:rapping` + every line has `SKILL-RAPPER-WORDUP`
- [ ] A Figma-related ask → `DISPATCH:agent:FigmaGetter` + sentences end with `AGENT-FIGMA-SHAZAM`
- [ ] Rap prompt does not produce skill markers, and vice versa (no cross-surface bleed)

### Safety
- [ ] Private plugin repo without CCA access → resolution_failure surfaced, job continues
- [ ] Subpath traversal (`owner/repo:../outside`) rejected
- [ ] Non-GitHub host is not silently fetched

### Edge cases
- [ ] Duplicate spec entries with conflicting bools resolve deterministically
- [ ] Two plugins with the same clone URL but different subpaths clone the repo once
- [ ] A large `settings.json` loads within the setup budget
