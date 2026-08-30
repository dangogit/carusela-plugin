---
name: tune-club-mentor
description: Configure a Carusela club's AI mentor - its name, instructions, vocabulary and starter prompts - so it answers from the club's own material. Use when somebody wants to set up or improve the AI assistant in their club, or says it gives generic or wrong answers.
---

# Tune the club mentor

## What this skill forbids

**Never put a secret in `mentor_instructions`.**

It is published with the club's configuration and it is **public**. No API key, no token, no
password, no internal URL, no member's details. This is not a style rule; the field ships to
anyone who can read the config.

**Never write instructions that let it invent lessons.** A mentor that confidently names a
lesson which does not exist is worse than one that says it does not know, because the member
goes looking. Make "say you do not know and offer the nearest real thing" an explicit rule in
every mentor you write.

## The four fields

All through `update_club_copy`, all Ring A, all published immediately.

| field | what it does | replaces or merges |
|---|---|---|
| `mentor_name` | what members call it | replaces |
| `mentor_instructions` | its rules, max 8000 chars | **replaces the whole block** |
| `mentor_content_terms` | nouns that mean "our own material" | **replaces the list** |
| `suggested_prompts` | the starter chips | **replaces the list** |

The three that replace do not append. Read the current value with `get_config` first, or you
will silently drop what is there.

The 8000 character cap exists because this is sent to the model on **every answer**. Long
instructions are not more control, they are a bill.

## What actually makes it good

Not tone. Structure. A mentor is useful when it can route a question to the exact place the
answer lives, so the instructions should be mostly a map.

Four blocks, in this order:

**1. Who and what.** One paragraph: what the club is, who teaches, who the members are.

**2. The map.** A line per course or section saying what is in it. This is the block that does
the work. Without it the mentor answers from general knowledge and never cites the club.

**3. How to answer.** Language and register (for Hebrew: spoken Israeli, not translated, with
tool names left in English). Short answer first, then where to find it. Who the member is, so
the level is right. And the two prohibitions: do not invent a lesson, and say when something is
not covered.

**4. The recurring questions.** The five or ten things members actually ask, each with the
lesson that answers it. Pull these from real sources: a Q&A recording, a support inbox, the
questions in a community feed. This is the highest-value block and the one most people skip.

Add a **sensitive topics** block when the club teaches anything with a footgun — credentials,
payments, anything that can lose data. Say what the mentor must never advise and which lesson it
must point at first.

## `mentor_content_terms`

The nouns that make a question count as being about the club's own material rather than the
world. For a workshop: the words for the sessions, the days, the modules, the tools taught. Get
these right and the mentor searches the club before it answers from training data.

Replace the list, do not guess at adding to it: an empty list restores the platform default.

## `suggested_prompts`

Four to six. Make them the questions a **stuck** member has, not the questions a brochure would
ask. "How do I start?" is worth less than "It says it fixed it and it did not, what now?" — the
second proves the mentor knows this specific club.

## What is refused here, and where it lives

`mentor_avatar`, `mentor_route_prefixes` (which areas it may link to), `mentor_internal_hosts`
and `mentor_starter_answers` are all Ring B: the mentor section of the brand editor. The refusal
messages name them. Do not look for another tool.

## Verify

`get_config` and read `mentor` back. Then ask it something only this club could answer. A
generic answer means the map block is too thin, not that the model is bad.
