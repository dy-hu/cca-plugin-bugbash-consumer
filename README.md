# cca-plugin-bugbash-consumer

Consumer repo for the [CCA plugins bug bash](https://github.com/github/sweagentd/issues/11889).

The plugin under test lives at [`dy-hu/cca-plugin-bugbash`](https://github.com/dy-hu/cca-plugin-bugbash) and is enabled via `.github/copilot/settings.json`. It bundles one custom agent (Rapper), one slash command (poetry-command), one skill (rapping-skill), and a second agent with MCP (FigmaGetter).

Each surface prints `DISPATCH:<MARKER>` once and tags every line with the same marker, so the agent output alone tells you which surface fired:

| Surface | Trigger phrase | Marker |
|---|---|---|
| Rapper agent | "write a rap song about X" | `RAPPER-AGENT` |
| poetry-command | "write a poem about X" | `POETRY-COMMAND` |
| rapping-skill | "write a limerick about X" | `RAPPING-SKILL` |
| FigmaGetter agent | Figma-related ask | `FIGMA-AGENT` |

## Running a case

Edit `.github/copilot/settings.json` for the setup the row needs, open an issue with the trigger phrase, and assign Copilot. The session log shows the plugin clone in setup and the dispatch line in the agent loop. If a row uncovers a bug, file a one-liner sub-issue under [#11889](https://github.com/github/sweagentd/issues/11889) and self-assign.

## Test matrix

### Settings loading
- [x] Repo-level `.github/copilot/settings.json` loads (baseline)
- [ ] Org-level `copilot/settings.json` in `dy-hu/.github` loads when repo settings are absent
- [ ] Repo entry wins over org entry for the same spec
- [ ] Malformed `settings.json` does not crash the job; a warning surfaces
- [ ] `{"enabledPlugins": {"<spec>": false}}` skips the plugin (no clone in setup logs)

### Spec parsing and resolution
- [x] `dy-hu/cca-plugin-bugbash` (owner/repo)
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
- [x] `plugin.json` at repo root is picked up
- [ ] `plugin.json` in a subpath is picked up (needs a second fixture repo)
- [ ] Plugin's `.mcp.json` merges into session MCP config
- [ ] Plugin's `.lsp.json` registers LSP servers
- [ ] Rapper appears in the agent's `task` tool agent list

### Dispatch (the markers do the verification)
- [x] "Write a rap about CI and save to OUT.md" → `DISPATCH:RAPPER-AGENT` + every line ends with `RAPPER-AGENT`
- [x] "Write a poem about CI and save to OUT.md" → `DISPATCH:POETRY-COMMAND` + every line ends with `POETRY-COMMAND`
- [ ] "Write a limerick about CI and save to OUT.md" → `DISPATCH:RAPPING-SKILL` + every line ends with `RAPPING-SKILL`
- [ ] A Figma-related ask → `DISPATCH:FIGMA-AGENT` + sentences end with `FIGMA-AGENT`
- [ ] Rap prompt does not produce skill markers, and vice versa (no cross-surface bleed)

### Safety
- [ ] Private plugin repo without CCA access → resolution_failure surfaced, job continues
- [ ] Subpath traversal (`owner/repo:../outside`) rejected
- [ ] Non-GitHub host is not silently fetched

### Edge cases
- [ ] Duplicate spec entries with conflicting bools resolve deterministically
- [ ] Two plugins with the same clone URL but different subpaths clone the repo once
- [ ] A large `settings.json` loads within the setup budget
