---
name: seed-club-content
description: Load a club's courses, recordings, tutorials, guides and AI agents through the Carusela MCP without losing fields or publishing things nobody reviewed. Use when somebody wants to import, migrate, bulk-create or restructure club content, or asks to turn an existing site, syllabus or video library into a club.
---

# Seed club content

## What this skill forbids

**Never invent content that has no source.**

If the syllabus lists a lesson and there is no recording for it, the lesson does not get a made
up video id, and it does not get quietly dropped either. Create it with the sources that exist
and say in the report which items had no video and why. A club full of dead players is worse
than a club with a gap the owner knows about.

**Never publish a batch before the owner has seen it.** Some kinds go live the moment they are
created (see the table below), which means the decision is made for you unless you plan for it.
Create, review, then publish.

**Never write a second copy because the first one refused a field.** A refused field means that
kind has no column for it. Find where the value belongs; do not create a different content type
to hold it.

## Order of work

1. `list_my_clubs`, `get_config`, `get_club_overview`. Know the club before writing to it.
2. Decide the shape on paper first: which courses, which chapters, what belongs in the library
   instead. Show it to the owner. Restructuring 70 lessons afterwards is many calls.
3. Create categories (`manage_category`) before the content that references them.
4. Create content.
5. Publish and gate deliberately.
6. `audit-club-content`.

## Courses carry their lessons

`manage_course` takes `lessons` in the same call. Do not create a course and then add lessons
one by one; it is slower and the ordering is harder to get right.

- `chapter` on a lesson groups it. Lessons sharing a chapter name **consecutively** render as
  that chapter, so order matters more than the string does.
- Lessons are created in array order. `lesson_order` overrides it and **must list every lesson
  id in the course** — a partial list is refused.
- On update, a lesson entry with an `id` updates; one without an `id` is appended.

Per lesson, the fields worth filling: `title`, `description`, `chapter`, `video_url`,
`resource_links`, `duration_label`, `min_tier_level`.

## `resource_links` is not JSON

Newline-separated `Label|https://url` entries. Not an array, not JSON.

```
מדריך ההתקנה|https://example.com/guides/install.html
code.visualstudio.com|https://code.visualstudio.com/
```

It exists on lessons, tutorials, recordings and AI agents. **Guides do not have it** — a guide's
links go inside its `sections` text.

## Video: let the provider fill it in

Pass a Bunny, Vimeo or YouTube URL as `video_url` and leave `thumbnail_url` empty. Lessons,
recordings, tutorials and AI agents all resolve the poster frame from the provider on write, and
lessons and recordings resolve the runtime too. A value you pass is never overwritten by that, so
only pass one if you mean to override.

To upload a file rather than reference a video: `attach_media` with `action: "upload_target"`,
then PUT the bytes to the `upload_url` it returns with the right `Content-Type`, then use the
`public_url`. The target is single-use and expires.

## The four library kinds do not take the same fields

One tool, one shape, four different field sets. An inapplicable field is refused by name rather
than dropped, so a refusal is information, not a failure.

| | recording | tutorial | guide | ai_agent |
|---|---|---|---|---|
| `video_url` | yes | yes | no | yes |
| `resource_links` | yes | yes | **no** | yes |
| `tags` | yes | yes | yes | **no** |
| `category` | **no** | **no** | **no** | **required on create** |
| body | `learning_points` | `steps` | `sections` | — |
| default `is_published` | draft | draft | **live** | **live** |

`steps` and `sections` are **update only**. Create the item, then update it with its body. That
is two calls per guide and there is no way around it today.

**The `is_published` row is a DEFAULT, not a rule.** All four accept the field on create, so a
batch nobody has reviewed is staged by passing `is_published: false` explicitly. Omit it and you
get the row above, which differs by kind: a guide and an AI agent go live the moment they are
created.

## Categories: the trap

Only `courses`, `groups` and `ai_agents` have a real `category` field.

**Recordings, tutorials and guides join a category by carrying its exact name in `tags`.** So:

```
manage_category  scope: "recording"  name: "לייבים מלאים"
manage_library_item  kind: "recording"  tags: ["לייבים מלאים", "יום 1"]
```

The name in `tags` has to match the category name **character for character**. `"לייב מלא"`
against a `"לייבים מלאים"` category attaches to nothing, and nothing tells you. After creating
categories, check they are not empty — `audit-club-content` does this.

`scope: "recording"` is the shared pool for recordings, tutorials, guides and groups.
`scope: "course"` is the separate axis courses use. There is no third pool.

## `difficulty` is a closed set

Exactly four values, on recordings, tutorials and guides:

```
מתחילים · ביניים · מתקדמים · כל הרמות
```

Anything else fails with a raw Postgres constraint name. `"בינוני"` and `"מתקדם"` are both
wrong and both look right.

## Publishing and ordering

`set_content_visibility` handles `is_published`, `min_tier_level`, `is_featured` (recordings
only) and `order`.

`order` is **whole-collection**: it must contain every id in that collection, in the order you
want. A partial list is refused. One call with the full list sets the order; you do not order
items one at a time.

Groups have no published state at all. `is_private` is their only visibility axis.

## Report honestly

At the end, say how many of each type, how many published versus draft, what was gated to which
tier, and **what had no source**. If a category came out empty, say so rather than letting the
owner find it.
