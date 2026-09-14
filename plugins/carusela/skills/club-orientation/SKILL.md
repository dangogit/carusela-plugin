---
name: club-orientation
description: Read this before the first write to a Carusela club. Explains what the MCP can and cannot do, why a refusal is usually correct, and how to pick the right club. Use whenever a Carusela tool is about to be called for the first time in a session, when a tool refuses a field, or when somebody asks what Claude can change in their club.
---

# Carusela orientation

## What this skill forbids

**Never treat a refusal as a bug to route around.**

Every field the Carusela MCP refuses is refused on purpose, and the refusal message names where
that thing actually lives. `update_club_copy` will not set a colour, a price or a feature flag.
That is not a missing feature you should work around by finding another tool. It is the surface
telling you the change belongs somewhere else, usually because it decides money or access.

If a refusal blocks the user's goal, the answer is to say so and name the right door. Never a
workaround.

**Never write before you have read the club's config.** `get_config` is one call and it tells
you the brand, the tiers, the enabled features, the navigation and the mentor. Half the mistakes
in this surface come from writing into a club whose shape you assumed.

**Never guess `club_id`.** If the account reaches more than one club, every tool requires it and
guessing puts a lesson in a stranger's club. Call `list_my_clubs`, and if more than one comes
back and the user has not said which, ask.

## The one thing to do first

```
list_my_clubs            -> which clubs, and the id for every later call
get_config               -> brand, tokens, features, navigation, mentor
get_club_overview        -> how much of each content type exists, published vs draft
```

Three read-only calls. They cost nothing and they answer most of what you were about to guess.

## The tool map, by job

| The user wants to | Tools |
|---|---|
| See what exists | `get_club_overview`, `list_content`, `get_config`, `get_member_stats` |
| Add or edit a course | `manage_course` (lessons come with it, in the same call) |
| Add a recording, tutorial, guide or AI agent | `manage_library_item` |
| Add an event or a group | `manage_event`, `manage_group` |
| Organise | `manage_category`, `set_content_visibility` |
| Change words | `update_club_copy` |
| Change colours or logos | `preview_design_change` then `publish_design` |
| Bring in a video or upload a file | `attach_media` |
| Bring in members from another system | `import_members` |
| See what has been done | `get_audit_log` |

## The three rings, in one paragraph each

**Ring A is words.** Club name, tagline, page headings, the mentor's text, copy fragments,
outbound links. `update_club_copy` writes them and the platform publishes them on write, with no
second step. That is how the platform is built, and it is not a reason to skip asking: if the
user has not told you what the new wording should be, get it from them before you write it. What
the ring being ungated means is that a wrong word is cheap to correct, not that nobody needs to
see it.

**Ring B is design.** Colours and brand assets. It needs a human to look at it first, so it has
two steps: `preview_design_change` stages a draft and returns a single-use token plus a link a
person can open, and `publish_design` spends that token. There is no way to publish design
without previewing. See `brand-a-club`.

**Ring C is money and access.** Prices, offers, coupons, trials, tiers, the payment terminal.
**MCP does not write any of it, by design.** You can *use* the tiers a club already has to gate
content, but you cannot create a tier, set a price or make an offer. When a user asks for that,
say plainly that it is done in the club's own Sales workspace and move on.

## What MCP will never do, so stop looking

- Create or change prices, offers, coupons, trials, instalments or order bumps
- Create or rename access tiers
- Flip feature flags (`community`, `groups`, and the rest are operator-controlled)
- Reorder the navigation rail or edit the home page's blocks
- Touch a repository, a deployment, DNS or a domain
- Read a member's name, email or phone. `get_member_stats` returns counts and nothing else

The first three have real doors: the Sales workspace, the admin settings, and "ask Carusela".
Tell the user which door, and say plainly that this surface will not do it. Then carry on with
the part you can do. A refusal the user cannot act on is worse than no answer, so the door is
the part that matters.

## Three traps that cost real time

**Features can be off, and the surface will tell you.** A club with `features.groups: false` has
no groups area, and `manage_group` now REFUSES to create one rather than leaving an invisible row
behind. Read that refusal as information: the flag is operator-controlled, so the answer is to
tell the owner it needs Carusela to enable it, not to look for another way in. `get_config` shows
which features are on before you plan around one.

**Before Launch, a created course or library item is published, not a draft.** Nobody can reach a
pre-launch club, so the draft would hide the work from the owner rather than protect a member;
after Launch the old per-kind defaults return. `manage_course` says which happened in
`publish_default`. See `seed-club-content`.

**Categories for recordings, tutorials and guides are not a `category` field.** Those three
join a category by carrying its exact name in `tags`. `manage_library_item` refuses `category`
for them, correctly, because the column does not exist. See `seed-club-content`.

## Everything is logged

Every call lands in `get_audit_log` with `source: "mcp"`, including the ones that failed and the
arguments they carried. Use it at the start of a session to see what a previous session already
did, rather than repeating it. Emails in the arguments are redacted; that is deliberate.

## When you are done

Do not report success from the tool result alone. `get_club_overview` after a batch tells you
how many items are published versus draft, and that is the number the user actually cares about.
`audit-club-content` is the fuller version of that check.
