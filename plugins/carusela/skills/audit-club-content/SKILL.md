---
name: audit-club-content
description: Find what is silently wrong in a Carusela club - content nobody can see, categories attached to nothing, missing thumbnails, ungated material and surfaces that are switched off. Use after a bulk import, before a launch, or when somebody says the club looks empty or a member cannot find something.
---

# Audit club content

## What this skill forbids

**Never fix a finding by deleting.** Almost everything this audit turns up is content that
exists and is merely invisible: a draft, an empty category, a wrong tag. The fix is to publish
it, attach it or rename it. Deleting removes the owner's work to make a report look clean.

**Never report a clean audit you did not run.** Each check below is a real call. If one could
not run, say which and why.

## Why this exists

The failures this surface produces are quiet ones. Nothing errors. The tool returns success, the
owner believes they shipped, and the thing is invisible to every member. This is the list of
ways that happens.

## The checks

### 1. Content that exists and nobody can see

```
get_club_overview
```

Compare `total` against `published` for every type. Anything with a gap has drafts.

Then the reason for the gap, which is not the same for every type:

- **recordings and tutorials are created as drafts.** If a batch was imported and never
  published, this is why.
- **guides and AI agents are created live.** A gap here means somebody unpublished on purpose.
- **groups have no published state at all.** `is_private` is their only axis.

### 2. Surfaces that are switched off

```
get_config  ->  published.features
```

If `groups` is false, every group is invisible. If `community` is false, so is the feed and
everything on it. Creating into a switched-off surface succeeds and reports success.

Feature flags are operator-controlled and the owner cannot flip them. If content sits behind a
false flag, the finding is "ask Carusela to enable X", not "enable X".

### 3. Categories attached to nothing

```
manage_category  action: "list"  scope: "recording"
manage_category  action: "list"  scope: "course"
```

For each one, check whether anything actually carries it:

- **courses, groups and AI agents** carry a real `category` field. Compare against
  `list_content`.
- **recordings, tutorials and guides** join by carrying the category's **exact name** in `tags`.
  A near miss attaches nothing and looks fine from both sides.

The cheap way to see the truth: `manage_category action: "delete"` **without** `confirm`. It
refuses and reports the usage count across the whole pool. It is a read that looks like a write,
and it is the only place the counts surface.

Do not pass `confirm: true` while auditing.

### 4. Videos with no poster frame

```
list_content  type: "tutorial"
```

Tutorials do **not** resolve a thumbnail from `video_url` the way lessons, recordings and AI
agents do. Any tutorial with a `video_url` and a null `thumbnail_url` renders blank. Fix with
`attach_media action: "resolve"` and an update.

### 5. Everything at one tier

```
get_member_stats  include_tiers: true
list_content  type: "course"  min_tier_level: 0
```

If a club has paid tiers and every item sits at level 0, the paid tiers buy nothing. That is
usually an oversight after an import rather than a decision. Report it as a question, not a fix.

Check courses and their lessons separately: gating a course does not cascade to its lessons.

### 6. Events in the past

```
list_content  type: "event"
```

A home page with an upcoming-events block and only past events renders an empty block. Worth
telling the owner even though nothing is broken.

### 7. What earlier sessions already did

```
get_audit_log  source: "mcp"  limit: 50
```

Failed calls are logged with their error. A run of failures against one tool usually explains a
gap you are about to re-create. Read this before repeating a batch.

## Report shape

Per finding: what is wrong, how many items, and **who can fix it** — the owner, or Carusela.
Separate those two, because a list that mixes them leaves the owner unsure which half is
waiting on them.

End with the one-line count that matters: how many items are published and reachable by a
member at the lowest paying tier. That is the number that answers "is my club ready".
