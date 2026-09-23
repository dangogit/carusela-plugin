---
name: build-sales-funnel
description: Build and publish a Carusela Sales page and its Funnel (Order bumps, Upsells, Downsells and the Thank You page) through the Carusela MCP, with every price and every publication approved by a person first. Use when somebody wants to sell a course or a membership, set up a sales page, a checkout, an order bump, an upsell or downsell, a thank-you page, a coupon for a launch, an A/B test between funnel versions, or to roll a sales page or funnel back.
---

# Build a sales funnel

## What this skill forbids

**Never call a spending tool without a person's explicit yes in the chat.** The spenders are
`apply_commerce_changes`, `publish_sales_page`, `rollback_sales_page`, `publish_sales_funnel`,
`rollback_sales_funnel`, `start_sales_funnel_experiment` and `end_sales_funnel_experiment`, and
the token issuers that lead to them are `preview_commerce_changes`, `preview_sales_page`,
`preview_sales_page_rollback`, `confirm_sales_funnel_publish`, `confirm_sales_funnel_rollback` and
`confirm_sales_funnel_experiment`. Show the person what the preview or the issuer returned, wait
for a yes to that exact thing, then spend. For a Funnel and an A/B test, get that yes before the
confirm call as well: `preview_sales_funnel` and `save_sales_funnel_experiment` both say to confirm
only once the person approves.

**Never treat that yes as the authorization.** The token is. Each one is single-use, short-lived,
minted only by its issuer and bound to this club, this account and the exact state that was
shown. The server refuses a spender without its token, and a yes typed in chat does not change
that. A yes to one preview does not carry over to the next: if anything moved and you preview
again, show the new result and ask again.

**Never retry a refused token.** A spent, expired or stale token is refused on purpose. Read the
current state, preview or confirm again, show it again.

**Never promise a buyer can pay before `get_commerce_readiness` says `ready`.** The club's
CardCom terminal is connected by a person, by hand, at `/admin?tab=payments`. Never ask for
terminal credentials and never send them through MCP. Everything in this skill can be drafted
while the terminal is missing; nobody can pay until it is connected.

**Never author design or topology.** A Sales page is built only from the blocks
`get_landing_page_catalog` returns, with no colours, CSS, class names, raw HTML, scripts or
iframes; the page follows the club's current published brand on every render. A Funnel is a list
of Order bumps, an ordered list of steps and a Thank You; the server derives the graph, the node
ids and whether each step is an Upsell or a Downsell.

**Never send half a document.** Both `edit_sales_page_draft` and `edit_sales_funnel_draft`
replace the whole draft. A step whose `content` you leave out loses its authored copy, and an
empty `steps` list removes every step. Read first, change what you mean to change, send all of it.

## Who can do this

The owner or an admin of the club. Almost every tool here needs the `commerce` or
`design_publish` capability, and only those two roles hold them. A content editor can read a Sales
page, validate a draft and save one (`get_sales_page`, `validate_sales_page_draft`,
`edit_sales_page_draft`), and nothing else here. When a tool answers that the account's role does
not carry a capability, that is the answer: a person with that seat does it.

## The first calls

```
list_my_clubs             -> which club, and the id for every later call
get_commerce_readiness    -> can this club take money yet, and if not, why
get_commerce_catalog      -> courses, member tiers, offers and coupons, drafts included
list_sales_pages          -> what already exists, and every page id
get_landing_page_catalog  -> the closed block vocabulary a page is built from
```

All read-only. `get_commerce_catalog` pages with `offset` and `limit` (up to 100).

## The flow

Hebrew is the club's language. Every word a buyer reads (offer names, headings, button labels,
the Thank You) is written in Hebrew unless the owner says otherwise.

### 1. Readiness

`get_commerce_readiness` returns `status`: `ready`, `not_connected` (connect CardCom in the
payments tab), `not_synchronized` (connected, but the club's selling state is out of step; check
the existing connection in the admin) or `unknown` (the check could not be completed; retry it).
`can_prepare_drafts` is always true. `can_publish_paid_offers` is true only when `ready`.

Publishing a page is not the same as the page taking buyers. `get_sales_page` returns `readiness`
with two parts. `publish` is what a publish needs: a valid, non-empty page document and a primary
offer that exists, is published and fits the page. `activate` is what a buyer reaching it needs
on top of that: the page published, the club's CardCom terminal connected, a Funnel with a live
version (a Thank You at least) and the club launched. Tell the owner which of these are still open.

### 2. The product

A Sales page sells one product: a course or a member tier.

- A course comes from `manage_course`. See `seed-club-content`.
- A member tier is staged with `manage_membership_tier`: `name`, `description`, `rank` (its
  access level) and `isActive`. It is the club's own membership level, not the Carusela package
  the club pays for.

A page that sells a member tier needs an offer for that tier.

### 3. Offers and coupons

Stage each with its own tool. Every call carries a new `request_id` (a UUID you reuse only to
retry the same request), and `values` holds the complete desired state, never a patch. Omit
`resource_id` to create; pass it to change an existing one. To revise a staged draft, pass its
`draft_id` together with its exact `expected_revision`; `get_commerce_changes` lists your drafts.

`manage_offer` values:

| field | rule |
|---|---|
| `name` | 2 to 120 characters |
| `description` | up to 2000, or null |
| `courseId` / `membershipTierId` | exactly one of the two |
| `priceAgorot` | whole agorot, 0 or more (₪490 is `49000`) |
| `interval` | `one_time`, `monthly` or `yearly` |
| `trialDays` | 0 or more, and 0 on a one-time offer |
| `allowsInstalments`, `maxInstalments` | one-time offers only; `maxInstalments` at least 2 and only with instalments on |
| `isPublished` | the state after approval, default true |
| `orderBumpOfferId` | the offer's own single order bump, or null |

`manage_coupon` values: `code` (stored in upper case), `discountType` `percent` or `fixed`,
`discountValue` (a percent up to 100, or fixed agorot), `maxUses` or null, `validFrom` and
`validUntil` as ISO date-times with an offset (`validUntil` may be null), and `isActive`.

Staging changes nothing live: a new offer, tier or coupon stays unpublished or inactive, and an
existing live one is untouched until apply.

Then:

```
preview_commerce_changes  { changes: [{id, revision}], page_ids?, course_ids? }
  -> drafts, pages, readiness, confirmation_token, expires_at (10 minutes)
apply_commerce_changes    { confirmation_token }
```

Show the person every proposed price, interval, trial, instalment count, coupon and access level,
and the price-change effects the preview lists for an existing offer (new buyers pay the new
price at once, active subscribers keep the old price until their next renewal, past orders do
not change). Call `apply_commerce_changes` only after their yes. The batch applies whole or not
at all. It never charges anybody.

`page_ids` and `course_ids` make the same batch publish Sales pages and draft courses too. For a
selected page, apply publishes its current draft and, when the page's Funnel has a saved draft,
publishes that Funnel as well, all in one transaction. That is the shortest path for a new club:
stage the offers, build the page and the Funnel on draft offers, then approve and apply the whole
launch at once. A page in the batch still has to pass its own publish readiness, judged against
the proposed offers.

### 4. The Sales page

```
create_sales_page  { request_id, product_kind, product_id, offer_id, title, slug,
                     entry_mode?, external_url?, seed_fields?, lesson_ids? }
```

- `product_kind` is `course` or `membership_tier`, and `offer_id` is an existing offer for that
  product. Draft offers and draft products are fine at this stage; they block publication, not
  authoring.
- `slug`: lowercase letters and digits in hyphen-separated groups, up to 100 characters. It
  cannot change once the page has been published.
- `title`: 2 to 160 characters.
- `entry_mode` is `hosted` (the default: Carusela serves the page) or `external`.
- **An external page is the club's own site.** Pass `external_url`: a public `https` address
  with no query, fragment, port or credentials. The content stays on that site, and its buy
  button links to the Carusela checkout on the club's own address. The link to paste there is
  copied from the page's screen in the admin once the page is published. An external page takes
  no `seed_fields` and no `lesson_ids`.
- `seed_fields` (`title`, `cover`, `description`, `instructor`) and `lesson_ids` (up to 100,
  published lessons of that course only) copy course material into the first draft. A member
  tier page cannot seed from a course.

It never publishes. It returns the page id, `readiness`, `first_remediation` and the admin link.
Repeating the identical request with the same `request_id` returns the same draft.

Then author it:

```
get_sales_page             -> draft_document (the exact shape the edit takes back),
                              draft_revision, product, primary_offer, brand_kit,
                              authoring_context, versions, readiness
validate_sales_page_draft  -> the first thing wrong, by path; writes nothing
edit_sales_page_draft      { page_id, expected_draft_revision, document }
select_sales_page_primary_offer { page_id, offer_id }
```

The document is `{ schemaVersion: 1, slug, title, seo: { title, description, ogImageUrl },
blocks }`, with up to 100 blocks, each id unique. Validate before saving. Re-read after every
save, because the draft revision moves. Do not copy any `brand_kit` value into a block.

Publishing:

```
preview_sales_page  { page_id }
  -> candidate_document, entry_url, admin_url, confirmation_token, expires_at
publish_sales_page  { page_id, confirmation_token }
```

`preview_sales_page` refuses while the page is not ready to publish and names what is missing.
Show the person the candidate and the admin link, and publish only after their yes. Saving the
draft again invalidates the token. To go back, `preview_sales_page_rollback` with a `version`
from the history, then `rollback_sales_page` with the same version and its token; a rollback
appends the old document as a new version and edits nothing.

### 5. The Funnel

```
get_sales_funnel             { page_id }
list_funnel_followup_offers  { page_id, page?, limit?, query? }
edit_sales_funnel_draft      { page_id, expected_draft_revision, expected_live_version, draft }
preview_sales_funnel         { page_id, expected_draft_revision, expected_live_version, draft }
confirm_sales_funnel_publish { page_id }
publish_sales_funnel         { page_id, confirmation_token }
```

`get_sales_funnel` returns the current draft with its draft revision, the live version (0
when nothing is published), the primary offer, the offer's own single order bump, `repairs` and
every published version. `expected_draft_revision` is null when the page has no Funnel draft yet.

The draft is `{ orderBumps, steps, thankYou }`:

- `orderBumps`: 0 to 3 entries of `{ offerId, headline?, body? }`. A bump is paid inside the
  checkout, in the same charge as the primary offer.
- `steps`: 0 to 12 entries of `{ id, offerId, content?, onAccept, onDecline }`, in the order a
  buyer meets them. `id` matches `^[a-z0-9-]{1,40}$`. `onAccept` and `onDecline` are another
  step's id or `"thank_you"`, never the step's own id, and no arrangement may let a buyer reach
  the same step twice. `content` is `{ headline?, body?, acceptLabel?, declineLabel?,
  timerSeconds? }`, at most 200, 2000, 60 and 60 characters, and a timer of 30 to 3600 seconds.
  A field left out falls back to copy derived from the offer.
- `thankYou`: `{ heading, body?, nextAction? }`. `heading` up to 200 characters, `body` up to
  2000, and `nextAction` is `{ label, href }` where `href` is a path on the club's site or an
  `https` address.

**Upsell or Downsell is derived, not chosen.** A step reached only by declines is a Downsell;
every other step is an Upsell. Arrange `onAccept` and `onDecline` to get the shape the owner
wants, then check the kinds in the preview.

**Which offers the server accepts:**

- A step's offer costs more than 0 and is either a one-time course offer with no trial, or a
  member tier offer billed monthly or yearly. It is not the page's primary offer, not the primary
  offer's own order bump and not one of this Funnel's Order bumps. The same offer on two steps is
  refused. `list_funnel_followup_offers` returns exactly the offers a step may use, with their
  real publication flags; pick from it.
- An Order bump needs a primary offer that is a one-time course offer. The bump itself is a
  one-time course offer that costs more than 0, is not the primary offer, is not any step's offer
  and is not listed twice. When the primary offer allows instalments, the bump must allow
  instalments too, with a maximum at least as high. No tool lists bump candidates, so check this
  against `get_commerce_catalog` yourself; the save refuses a bump that fails it.
- Unpublished offers may sit in a draft. Publication needs them published.

Save with `edit_sales_funnel_draft`, then show the person what a buyer meets:
`preview_sales_funnel` writes nothing and issues no token, and returns what each Order bump state
charges at checkout and, for every step, its derived kind, where accepting and declining lead,
the amount, the access it grants and the consequences. Walk the person through both branches of
every step.

After their yes, `confirm_sales_funnel_publish` mints the token and returns the branches walked
from the **saved** draft. Those are what the token authorizes, so if they differ from what you
showed, show these and ask again. It refuses while `repairs` is not empty, when there is no saved
draft, and when an offer or product in the Funnel is still unpublished. Then
`publish_sales_funnel` with the token. Saving the draft again, or any change to an offer in the
Funnel, invalidates it.

To restore an older version: `confirm_sales_funnel_rollback` with a `target_version` older than
the live one, show its branches, then `rollback_sales_funnel` with the same version and the token
after a yes. It appends that version as a new one. A version whose follow-up offer has since been
unpublished is refused. Buyers already in a session stay on the version they entered on.

A Sales page with no live Funnel cannot take buyers, hosted or external. When the owner wants no bumps and no
steps, publish a Funnel that is only a Thank You.

### 6. A/B tests (optional)

Only once the Funnel has at least two published versions; before that
`get_sales_funnel_experiments` reports `state: "unavailable"`.

```
get_sales_funnel_experiments     { page_id }
save_sales_funnel_experiment     { page_id, experiment_id | null, expected_revision | null, name, variants }
confirm_sales_funnel_experiment  { page_id, experiment_id, action: "start" | "end",
                                   expected_revision, expected_live_version, variants_digest,
                                   winner_version }
start_sales_funnel_experiment    { ..., confirmation_token }
end_sales_funnel_experiment      { ..., winner_version, confirmation_token }
```

`variants` is 2 to 4 entries of `{ version, weight }`: distinct published versions, whole weights
of 1 to 99 that sum to exactly 100. Only a draft test can be edited. One test runs per Funnel at a
time; each new checkout session is pinned to one variant, and a session in progress keeps its
version. Ending with a `winner_version` publishes that version as the new live Funnel; ending with
null keeps the live one. Say which of the two the person is approving before you confirm.

## Refusals and what they mean

| The tool says | It means | Do |
|---|---|---|
| needs the `commerce` (or `design_publish`) capability, or the seat does not hold it | wrong role, or wrong club | `list_my_clubs`; otherwise a person with that seat does it |
| the club is suspended and accepts no changes | the owner has to restore the club from the platform billing page | stop and tell the owner; retrying does not help |
| an existing draft requires its exact revision | `draft_id` sent without `expected_revision`, or the reverse | send both |
| the commercial change was refused, preview again | a draft or a resource moved, or the token expired or was spent | `get_commerce_changes`, preview again, show again |
| the draft or the publication history moved | somebody saved since your read | `get_sales_page` or `get_sales_funnel`, then preview again |
| that confirmation no longer matches this club's Funnel | the token was used, expired, or the draft or an offer behind it moved | `get_sales_funnel`, preview, confirm again |
| `request_id` already belongs to a different draft | the id was reused for a different request | reuse the original request unchanged, or pick a new id |
| slug already belongs to another Sales page | the slug is taken | choose another |
| `followup_offer_unavailable` on `steps` | that offer cannot be a step | pick from `list_funnel_followup_offers` |
| `order_bump_primary_incompatible` | the primary offer is not a one-time course offer | remove the bumps or change the primary offer |
| `order_bump_instalments_incompatible` | the bump allows fewer instalments than the primary offer | stage the bump offer with enough instalments, or pick another |
| `order_bump_offer_unavailable` | the bump offer fails the bump rules | pick another one-time course offer |
| `duplicate_step_offer`, `step_self_target`, `step_target_unknown`, `step_target_cycle` | the step list cannot be walked | fix the ids and targets |
| `max_steps_exceeded`, `max_bumps_exceeded` | more than 12 steps or 3 bumps | trim |
| `funnel_products_unpublished` | an offer or product in the Funnel is unpublished | apply the commerce changes first, or publish through the commerce batch |
| no saved Funnel draft to publish | nothing to bind a token to | `edit_sales_funnel_draft` first |
| `legacy_document_not_representable` | the old two-offer document shape was sent for a Funnel with bumps or more than two steps | send `{ orderBumps, steps, thankYou }` |
| a Sales page readiness code, such as `primary_offer_unpublished` or `landing_page_empty` | the page is not ready to publish | fix what it names; `get_sales_page` lists them all |

Every call, refused or not, lands in `get_audit_log` with `source: "mcp"`.

## When you are done

Read back, do not trust the receipt alone: `get_sales_page` for the published version and
`readiness.activate`, `get_sales_funnel` for the live version. Tell the owner what is live, what
a buyer meets on each branch, and what still stands between the page and its first buyer, most
often the CardCom connection or the club's launch.
