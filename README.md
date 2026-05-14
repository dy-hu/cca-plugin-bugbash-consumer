# cca-plugin-bugbash-consumer

Consumer repo for the [CCA plugins bug bash](https://github.com/github/sweagentd/issues/11889).

Plugin fixture: [`dy-hu/cca-plugin-bugbash`](https://github.com/dy-hu/cca-plugin-bugbash) (mirror of belaltaher's `test-plugin-with-agent`).
Plugin contents:
- `agents/Rapper.md` — custom agent, signal `FO-SHIZZLE`
- `agents/FigmaGetter.md` — custom agent with MCP, signal `SHAZAM`
- `commands/poetry-command.md` — slash command, signal `POEM`
- `skills/rapping-skill/SKILL.md` — skill, signal `FO-SHIZZLE`
- `.mcp.json`, `.lsp.json` — plugin-scoped MCP / LSP

## How to run a test case

1. Edit `.github/copilot/settings.json` on `main` (PR or direct push) to set up the case.
2. Open a new issue with a prompt that should exercise the loaded surface.
3. Assign Copilot as the issue's assignee → CCA job kicks off.
4. Watch the session: setup phase shows plugin clone, agent loop shows tool calls.
5. Verify the expected outcome below. File any unexpected behavior as a sub-issue under [#11889](https://github.com/github/sweagentd/issues/11889).

## Test matrix

### Settings loading
- [ ] Repo-level `.github/copilot/settings.json` loads (default case)
- [ ] Org-level `copilot/settings.json` in `dy-hu/.github` loads when repo settings are absent
- [ ] Merged precedence: repo entry wins over org entry for the same spec
- [ ] Malformed settings.json (invalid JSON) → job starts, warning surfaced
- [ ] `enabledPlugins: { "<spec>": false }` skips the plugin (no clone)

### Spec parsing and resolution
- [ ] `dy-hu/cca-plugin-bugbash` — happy path, owner/repo
- [ ] `dy-hu/cca-plugin-bugbash@<sha>` — pinned commit
- [ ] `dy-hu/cca-plugin-bugbash@some-branch` — branch ref
- [ ] `https://github.com/dy-hu/cca-plugin-bugbash.git` — full HTTPS URL with `.git`
- [ ] `git@github.com:dy-hu/cca-plugin-bugbash.git` — SSH URL
- [ ] `https://gitlab.com/foo/bar` — non-GitHub URL rejected cleanly
- [ ] `./local-plugin` — local path accepted (plugin checked in to consumer repo)
- [ ] `/abs/path` — absolute path normalized
- [ ] `../escape-attempt` — traversal rejected
- [ ] `""`, `"   "`, `"owner/repo/"` — weird input handled gracefully

### Marketplace
- [ ] `plugin@marketplace` spec resolves via `extraKnownMarketplaces`
- [ ] Bare plugin name resolves via default marketplace
- [ ] Missing plugin / missing repo / missing ref / malformed `marketplace.json` all fail cleanly
- [ ] Marketplace not in allowlist blocked under `strictKnownMarketplaces`

### Plugin loading (use the dy-hu/cca-plugin-bugbash fixture)
- [ ] `plugin.json` loads from repo root
- [ ] `plugin.json` loads from subpath (use a fixture with plugin in subdir)
- [ ] Plugin's `.mcp.json` merges into session MCP config
- [ ] Plugin's `.lsp.json` registers LSP servers
- [ ] Custom agent is discoverable in the agent's `task` tool

### Runtime execution (signal-based verification)
- [ ] **Rap path:** issue "Write a rap about CI and save to RAP.md" → output contains `FO-SHIZZLE` on every line → agent dispatch worked
- [ ] **Poem/skill path:** issue "Write a poem about CI and save to POEM.md" → output contains `FO-SHIZZLE` per line → skill loaded
- [ ] **Command path:** issue "Run `/poetry-command` about CI" → output contains `POEM` per line → command surface invoked
- [ ] **MCP-agent path:** issue exercising `FigmaGetter` → output contains `SHAZAM` → plugin-defined MCP server attached
- [ ] `copilot_swe_agent_runtime_entry_point_cca_v3_app` feature flag actually gates plugin loading

### Safety
- [ ] Private plugin repo without CCA's access → resolution_failure surfaced, job continues
- [ ] Subpath traversal (`owner/repo:../outside`) rejected
- [ ] Plugin source on a non-GitHub host not silently fetched

### Edge cases
- [ ] Duplicate plugin spec entries (same spec twice with different bool) → deterministic resolution
- [ ] Two plugins with same `CloneURL` but different subpaths → repo cloned once, both subpaths resolved
- [ ] Very large `settings.json` (many enabledPlugins entries) loads without timeout
