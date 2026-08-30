# Carusela for Claude Code

Run your club from Claude Code. This plugin connects the Carusela MCP and adds six skills that
keep an agent inside what the surface actually allows, so it stops guessing at fields that are
refused on purpose and stops publishing things nobody reviewed.

## Install

```
/plugin marketplace add dangogit/carusela-plugin
/plugin install carusela@carusela
```

The MCP server comes with it. On the first Carusela tool call Claude Code opens the OAuth flow
against `https://carusela.com/api/mcp`; approve it with the platform account that owns the club.
Nothing else to configure, and no token to paste anywhere.

If your account reaches more than one club, every tool needs a `club_id`. Ask Claude to
"list my clubs" once and it will carry the right one from then on.

## The skills

| skill | when it fires |
|---|---|
| `club-orientation` | before the first write. The tool map, the three rings, and what MCP will never do |
| `seed-club-content` | importing or bulk-creating courses, recordings, tutorials, guides and agents |
| `brand-a-club` | colours, logo, favicon, social card, through the gated design flow |
| `gate-club-access` | deciding what each access tier reaches |
| `audit-club-content` | after an import or before a launch: what is invisible and why |
| `tune-club-mentor` | making the club's AI assistant answer from the club's own material |

They fire from their descriptions, so in practice you say what you want and the right one loads.
You can also name one: "use audit-club-content".

## What this cannot do, by design

Prices, offers, coupons, trials, instalments, order bumps and access tiers are **not** writable
over MCP. They decide money and access, so they stay owner-confirmed actions in the Sales
workspace. You can gate content against tiers a club already has; you cannot create one.

Feature flags, the navigation rail and the home page's block layout are operator-controlled.

There are no repository, deploy, DNS or domain tools here, and the platform stores no
corresponding credentials. Claude Code may already hold your own GitHub and Vercel sessions
locally; those never enter Carusela MCP.

## Rough edges the skills work around

Real behaviours of the surface today. Each one is tracked, and the skills tell Claude how to
avoid it meanwhile, so you should not have to think about any of them:

- Recordings, tutorials and guides join a category by carrying its **exact name in `tags`**, not
  through a `category` field. A near miss attaches nothing and looks fine from both sides.
- `difficulty` accepts exactly four values on recordings, tutorials and guides:
  `מתחילים`, `ביניים`, `מתקדמים`, `כל הרמות`. Anything else is refused by the database.
- Guides and AI agents are created **published**. Recordings and tutorials are created as drafts.
- Tutorials do not resolve a poster frame from `video_url` the way lessons, recordings and AI
  agents do.
- The access-tier ladder is readable only from `get_member_stats`.
- `manage_group` succeeds in a club whose `groups` feature is switched off, and the group is then
  invisible to everyone.

## Safety

Every call is written to the club's audit log with `source: "mcp"`, including failures and their
arguments, with emails redacted. Ask Claude to "show the audit log" to see what any session did.

Design changes cannot be published without a preview: `preview_design_change` returns a link a
human opens and a single-use token that `publish_design` spends. The skills require showing you
that link first.

Member names, emails and phone numbers are never returned by any tool. `get_member_stats` gives
counts only.
