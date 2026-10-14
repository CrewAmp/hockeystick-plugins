# HockeyStick plugins for Claude and Codex

This directory carries Claude and Codex marketplaces for one shared plugin,
`hockeystick`. Its contents are published, as they are, to the public repository
`CrewAmp/hockeystick-plugins` that clients install from. Nothing in it is secret: the
client ID is a public native OAuth client with PKCE and no secret.

## Claude

One plugin serves claude.ai, Claude Desktop and Claude Code. `plugins/hockeystick/.mcp.json`
carries the MCP URL and an `oauth` block with the Claude Code static client ID, callback
port 6278 and the scopes `hsai.read hsai.write` as a string (Claude Code rejects an
array and drops the server with no visible error). Claude Code needs the block because
our sign-in service matches loopback callback ports exactly, so Claude's hosted client
metadata does not work there. claude.ai either ignores the block (and signs in with
Claude's own identity) or uses the client ID; for the second case the Auth0 application
`IxdjX…` must list `https://claude.ai/api/mcp/auth_callback` as an allowed callback.
It does (checked with the Auth0 CLI on 12 Oct 2026); keep it there.

A plugin added on claude.ai also syncs into Claude Code as `hockeystick@synced`, with the
same `oauth` block, so a Claude Code user needs no separate install. A copy installed
from the marketplace in Claude Code shadows the synced one (same manifest name), and a
server added by hand with `claude mcp add` wins over both.

### Install

claude.ai or Claude Desktop: **Customize > Plugins > Add > Add marketplace**, enter
`CrewAmp/hockeystick-plugins`, add `hockeystick`, then connect the HockeyStick connector
on the plugin's Connectors tab and sign in with the HockeyStick email address. On a
Team or Enterprise plan an Owner first adds the connector under
**Organization settings > Connectors**. A zip of `plugins/hockeystick` also installs
(**Add > Upload plugin**), but a zip never updates; keep it for people who cannot reach
GitHub.

Claude Code without a claude.ai plugin:

```sh
claude plugin marketplace add CrewAmp/hockeystick-plugins
```

```sh
claude plugin install hockeystick@hockeystick
```

Then start `claude`, type `/mcp`, select `hockeystick` and sign in.

### Updates

The manifest carries no `version`, so the version is the commit of the published
marketplace and every push is a new version. Do not add a `version` field: Claude Code
keeps users on the cached copy until that string changes.

- claude.ai and Claude Desktop: a plugin from a marketplace updates from its source with
  nothing to accept. The user can press **Check for updates** on the marketplace, or turn
  on **Sync automatically** for a GitHub marketplace they added themselves.
- Claude Code: a synced plugin follows the claude.ai account. For a marketplace install,
  auto-update is off; the user turns it on once (`/plugin`, **Marketplaces**,
  `hockeystick`, **Enable auto-update**) or runs
  `claude plugin update hockeystick@hockeystick`.

New MCP tools need no plugin update. Every client reads the tool list and the server
`instructions` from the server at each session start, and the skill tells the client to
answer "what can HockeyStick do" from that live list. Change the plugin only when the
skill text, the URL or the OAuth settings change.

### Claude versions tested

Claude Code 2.1.278 on 10 Oct 2026 (`clientId`, `callbackPort` and `scopes` from a
plugin `.mcp.json` are used at sign-in). The claude.ai path is proven by the SLACK-193
acceptance run.

## Codex

The same `plugins/hockeystick` directory serves Codex CLI and the desktop app.
`.codex-plugin/plugin.json` selects `./mcp.codex.json` through **`mcpServers`**
and loads the same `skills/hockeystick/SKILL.md` as Claude. The Codex MCP file
carries the Test URL, static client ID, callback port 6279 and the exact array
`["hsai.read", "hsai.write"]`. Claude's `.mcp.json` remains separate and unchanged.

### Install

Use **Codex ≥ 0.160**. Install with two commands:

```sh
codex plugin marketplace add CrewAmp/hockeystick-plugins
codex plugin add hockeystick@hockeystick
```

Then run `codex mcp login hockeystick`, or open
**Settings > MCP servers > Authenticate** in the desktop app. Sign in with your
HockeyStick email address and email code, not a password or Google sign-in.
The plugin supplies all connection settings. Installation does not start sign-in.
Older binaries can use the manual fallback in
[the client guide](https://github.com/CrewAmp/hockeystickai/blob/main/docs/auth/mcp-client-setup-guide.md#step-1b-codex).

### Updates

```sh
codex plugin marketplace upgrade hockeystick
```

This refreshes the Git marketplace snapshot and its installed plugin cache.
Start a new CLI session or restart the desktop app to load the changed files.
Codex 0.160.1 and desktop 0.162.0-alpha.17.2 both refreshed a changed skill in an
installed, versionless plugin using this command; neither offers `codex plugin update`.
The manifest has no `version`: both binaries accept it and report version `local`.
New MCP tools need no plugin update; the tool list comes from the server.

### Versions tested

On 12 October 2026, Codex 0.160.1 (isolated npm install) and desktop-bundled
0.162.0-alpha.17.2 installed the shared plugin with no version field and used the
client ID, scopes and callback port from `mcp.codex.json` at sign-in.
The manifest key `mcp` was ignored by both binaries, falling back to Claude's
`.mcp.json`; `mcpServers` selected the custom file correctly.
The machine's npm Codex 0.137.0 is below the minimum: it uses the client ID and
scopes but ignores the plugin callback port. Use the manual fallback or upgrade.

## Publish

The public repository holds this directory at its root, as one commit per publish
chained on the previous one. The workflow `.github/workflows/publish-plugins.yml`
does this on every push to `main` that touches this directory (it needs the secret
`HOCKEYSTICK_PLUGINS_DEPLOY_KEY`, a write deploy key on the public repository). To
publish by hand, from the commit on `main` that you want to publish, make a commit of
this directory's tree on top of the published head and push it:

```sh
prev=$(git ls-remote git@github.com:CrewAmp/hockeystick-plugins.git refs/heads/main | cut -f1) && c=$(git commit-tree "$(git rev-parse HEAD:clients/plugins)" -p "$prev" -m "Publish clients/plugins from hockeystickai $(git rev-parse --short HEAD)") && git push git@github.com:CrewAmp/hockeystick-plugins.git "$c:main"
```

This runs in a second. (`git subtree split --prefix=clients/plugins` gives the same tree
with full history, but walks the whole repository and takes minutes.) Clients that turned
on **Sync automatically** receive the new commit by themselves.

## Environment

Both MCP configurations use **Test** values. Production gets its own plugin entry
when its MCP server is live (SLACK-82/83), with a different server name so the two
never collide. `mcp/test/plugin-marketplace.test.ts` pins Claude to
`mcp/provider/creation-outputs.json` and Codex to the admitted-client table in
`docs/auth/cimd-client-onboarding.md`.
