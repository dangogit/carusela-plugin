# Carusela for Claude Code

Run your [Carusela](https://carusela.com) club from Claude Code. Build courses, upload a library,
brand the club, set access tiers and configure its AI mentor, by talking to Claude.

```
/plugin marketplace add dangogit/carusela-plugin
/plugin install carusela@carusela
```

Both are typed **inside Claude Code**, not in a terminal. The MCP connection comes with the
plugin, so there is no server address to paste and no token to copy. The first time you ask
Claude to do something in your club, a browser opens and you approve it with the Carusela
account that owns the club. Once.

## Why a plugin and not just the MCP

The connection on its own gives Claude the tools. It does not tell Claude how the surface is
shaped, so it guesses, and several of the natural guesses are quietly wrong: content created live
when you wanted a draft, a category that looks attached and is not, a tier that does not exist.

The six skills are the part that stops that.

| skill | what it is for |
|---|---|
| `club-orientation` | the map, and which refusals are correct so Claude stops trying to route around them |
| `seed-club-content` | importing or bulk-creating content without losing fields or publishing early |
| `brand-a-club` | colours, logo, favicon and social card, through the preview-then-publish gate |
| `gate-club-access` | what each access tier reaches |
| `audit-club-content` | what got created successfully and is still invisible |
| `tune-club-mentor` | making the club's AI assistant answer from your own material |

They load from their descriptions, so in practice you say what you want. You can also name one:
"use audit-club-content".

## What it will not do

Connecting or changing a payment terminal, and charging anybody. Those stay in the Carusela
Sales workspace.

Prices, offers, coupons, trials, instalments, order bumps and the access tiers themselves are
writable, but never in one step: Claude prepares them on a draft that changes nothing a buyer
can see, shows you the exact proposal, and applies it only against a single-use token that the
proposal you approved produced. If you did not read a proposal, nothing went live.

Feature flags, the navigation rail and the home page layout are operator-controlled.

There are no repository, deploy, DNS or domain tools here, and Carusela stores no such
credentials. Claude Code may already hold your own GitHub and Vercel sessions on your machine;
those never enter this connection.

## Safety

Every call is written to your club's audit log with `source: "mcp"`, including the ones that
failed and the arguments they carried, with email addresses redacted. Ask Claude to "show the
audit log".

Design changes cannot be published without a preview: staging returns a link a human opens and a
single-use token that publishing spends, and the skills require showing you that link first.

Claude can read your member directory — names, email addresses, phone numbers, join dates and
subscription status — because it is your club's own data and you are the one asking. The
statistics tool is the exception in the other direction: `get_member_stats` returns counts and
nothing that identifies anyone, so "how many members do I have" never opens the directory at
all. Email addresses in the audit log are redacted either way.

## Requirements

- [Claude Code](https://claude.com/claude-code)
- A Carusela club, and the platform account that owns it

## Licence

MIT. See [LICENSE](LICENSE).
