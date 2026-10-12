# HockeyStick plugin for Claude

This directory is a Claude plugin marketplace with one plugin, `hockeystick`. Its
contents are published, as they are, to the public repository
`CrewAmp/hockeystick-plugins` that clients install from. Nothing in it is secret: the
client ID is a public native OAuth client with PKCE and no secret.

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

## Install

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

## Publish

The public repository holds this directory at its root. Push it with a subtree split
from the commit on `main` that you want to publish:

```sh
git subtree split --prefix=clients/claude -b hockeystick-plugins-publish
```

```sh
git push git@github.com:CrewAmp/hockeystick-plugins.git hockeystick-plugins-publish:main
```

Delete the local branch afterwards. Clients that turned on **Sync automatically** receive
the new commit by themselves.

## Updates

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
`instructions` from the server at each session start, and the skill tells Claude to
answer "what can HockeyStick do" from that live list. Change the plugin only when the
skill text, the URL or the OAuth settings change.

## Environment

The MCP URL and the client ID in `plugins/hockeystick/.mcp.json` are the **Test**
values. Production gets its own plugin entry when the production MCP server is live
(SLACK-82/83), with a different server name so the two never collide. The test
`mcp/test/claude-plugin.test.ts` pins the file to `mcp/provider/creation-outputs.json`.

## Versions tested

Claude Code 2.1.278 on 10 Oct 2026 (`clientId`, `callbackPort` and `scopes` from a
plugin `.mcp.json` are used at sign-in). The claude.ai path is proven by the SLACK-193
acceptance run.
