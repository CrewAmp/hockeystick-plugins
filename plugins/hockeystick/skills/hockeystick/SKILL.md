---
name: hockeystick
description: Use when the user asks about their HockeyStick clients, partners, partner performance, monthly revenue or briefs (Story, Social, Webinar, Event), or wants to connect to or check their HockeyStick connection. Covers the first call, the connection check and what the HockeyStick tools refuse.
---

# HockeyStick

The HockeyStick connector gives you the `hockeystick` MCP tools. Every call runs as the signed-in HockeyStick user and is recorded against their account.

## First call

Always start with `list_clients`. It names the account and the clients the user can access. Use the returned client ids in every other call. If the user has access to more than one client, ask which one they mean before you read or write.

## Connection check

When the user asks to connect, to check the connection, or when a tool returns 401 or "needs authentication":

1. Tell the user to open the HockeyStick connector and sign in. On claude.ai and Claude Desktop: **Customize > Connectors > HockeyStick > Connect**. In Claude Code: type `/mcp`, select `hockeystick`, then sign in.
2. They must use the same email address as their HockeyStick account, and the email code, not a password or Google sign-in.
3. Call `list_clients`. A successful answer that names their account is the proof. Report the account and the clients.

If `list_clients` still fails after a fresh sign-in, the HockeyStick team must check the account link. Do not retry in a loop.

## Reads

Safe to call directly: `list_partners`, `search_briefs`, `get_brief`, `get_brief_timeline`, `get_partner_performance`, `get_monthly_revenue`, `get_monthly_revenue_context`, `get_activate_readiness`, `get_riley_publish_preflight`.

Example prompts the user may give:

- "Which HockeyStick clients can my account access?"
- "List my partners."
- "Show performance for partner <name> for <month>."
- "Show our September revenue."
- "List my webinar briefs."

## Writes

These change client data: `log_monthly_revenue`, `start_brief`, `update_story_setup`, `update_social_setup`, `update_webinar_setup`, `update_event_setup`, `generate_brief`, `decide_brief`, `record_diagnosis`, `dispatch_brief`, `activate_programme`, `publish_riley_pack`.

Before any write, show the user the exact values you will send and act only on their explicit confirmation. A brief starts as a draft (`start_brief`) that returns the missing questions; answer them only with facts the user gives you. Never invent transcripts, source material, speaker facts, strategy consent or an off-anchor reason. Generation (`generate_brief`) needs the user's explicit request and the version they reviewed.

## Refusals

A refused call returns a specific code and an explanation, for example `setup_validation_failed`, `story_branding_required`, `asset_stale` or `BRIEF_INCOMPLETE`. Tell the user what it says and what they can do. A refusal never authorizes a retry or renewed consent. After a timeout or a lost response, do not repeat a write; give the user the brief id and request id so the HockeyStick team can reconcile it.
