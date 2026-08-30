---
name: gate-club-access
description: Decide which Carusela content each access tier can reach, and apply it. Use when somebody wants to gate content, build a free tier, put material behind a paid level, or asks why a member cannot see something.
---

# Gate club access

## What this skill forbids

**Never invent a tier level.** `min_tier_level` must be a level the club's tiers table already
defines. A club with levels 0, 1 and 2 will refuse 3, and it is right to. Read the ladder before
you write to it.

**Never gate content you have not looked at.** Moving a lesson to level 2 takes it away from
everyone below. That is a revenue decision wearing the clothes of a config change, and the owner
makes it, not you. Propose the ladder, get agreement, then apply.

**Never claim you set up a subscription.** You did not. Gating content by tier is not the same
as creating a product, a price or a plan, and MCP cannot do the second thing at all.

## Read the ladder first

`get_club_overview` returns it as `access_tiers`, and you are calling that before any write
anyway:

```json
"access_tiers": [
  { "level": 0, "name": "חינם" },
  { "level": 1, "name": "בסיסי" },
  { "level": 2, "name": "פרימיום" }
]
```

`get_member_stats` with `include_tiers: true` also returns it, with a member count per tier.
Use that one only when the counts are the question: it is a tool about people, and gating content
is not.

## Apply it

`set_content_visibility`, one call per item, works across every content type:

```
kind: course | lesson | event | group | recording | tutorial | guide | ai_agent
id: <the item>
min_tier_level: <a level the club defines>
```

The same tool carries `is_published`, `is_featured` (recordings only) and `order`. An axis a
type does not have is refused by name rather than ignored, so a change that cannot happen never
reports success.

**A course and its lessons are gated separately.** Setting the course does not cascade. If the
owner wants a free preview lesson inside a paid course, that is exactly how you build it: course
at the paid level, one lesson at 0.

## A ladder that works

The shape most clubs want, and the reasoning:

- **Level 0 is the proof.** Whatever makes a stranger believe the club is worth paying for.
  Setup and prerequisites are good here: they cost the owner nothing to give away, and someone
  who has installed the tools has already invested.
- **Level 1 is the product.** The thing they paid for.
- **Level 2 is the depth.** Bonuses, archives, Q&A, the agents and automations.

Put the prerequisites free even when the course is paid. A member who cannot complete the setup
cannot use what they bought, and support absorbs the difference.

## When somebody asks for the thing MCP cannot do

"Set up a subscription", "make this cost 200 shekels", "add a coupon", "start a trial" — none of
these are MCP-writable, and no amount of looking will find a tool. Prices, offers, coupons,
trials, instalments, order bumps and the tiers themselves are owner-confirmed actions in the
club's Sales workspace.

Say it in one sentence, say what you *can* do (gate the content once the tiers exist), and do
that.

## Verify

`list_content` with `min_tier_level` returns only items requiring exactly that level. Three calls
give you the whole ladder and are the fastest way to show an owner what each tier actually
reaches.
