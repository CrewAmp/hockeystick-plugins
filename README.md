# HockeyStick plugins for Claude

This directory is a Claude plugin marketplace. Its contents are published, as they
are, to the public marketplace repository that clients install from. Nothing in it
is secret: the client IDs are public native OAuth clients with PKCE and no secret.

| Plugin                    | For                       | Sign-in                                                |
| ------------------------- | ------------------------- | ------------------------------------------------------ |
| `hockeystick`             | claude.ai, Claude Desktop | URL only; Claude identifies itself (CIMD)              |
| `hockeystick-claude-code` | Claude Code               | Static client ID and callback port 6278 in `.mcp.json` |

Two plugins exist because Claude Code cannot use Claude's hosted client metadata
(our sign-in service matches loopback callback ports exactly), and the docs do not say
whether claude.ai reads an `oauth` block from a plugin. If the claude.ai acceptance shows
that it ignores the block, merge the two.

Both plugins share one skill. `hockeystick-claude-code/skills` is a symlink to
`hockeystick/skills`; keep it that way so the text cannot drift.

Install only one of the two in Claude Code. Claude Code keeps the first server it loads
for a URL and drops later ones, so with both installed it can keep the URL-only server,
whose sign-in fails in Claude Code (measured 10 Oct 2026).

## Install

claude.ai or Claude Desktop: **Customize > Plugins > Add > Upload plugin** with a zip of
`plugins/hockeystick`, or add this marketplace from its git URL. Then connect the
HockeyStick connector on the plugin's Connectors tab and sign in with the HockeyStick
email address. On a Team or Enterprise plan an Owner first adds the connector under
**Organization settings > Connectors**.

Claude Code:

```sh
claude plugin marketplace add CrewAmp/hockeystick-plugins
```

```sh
claude plugin install hockeystick-claude-code@hockeystick
```

Then start `claude`, type `/mcp`, select `hockeystick` and sign in.

## Environment

The MCP URL in both `.mcp.json` files and the client ID in
`plugins/hockeystick-claude-code/.mcp.json` are the **Test** values. Production gets
its own values when the production MCP server is live (SLACK-82/83). The test
`mcp/test/claude-plugin.test.ts` pins the files to `mcp/provider/creation-outputs.json`.

## Versions tested

Claude Code 2.1.278 on 10 Oct 2026 (`clientId`, `callbackPort` and `scopes` from a
plugin `.mcp.json` are used at sign-in; `scopes` must be a string). The claude.ai path
is proven by the SLACK-193 acceptance run.
