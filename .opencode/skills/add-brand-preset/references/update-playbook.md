# Update playbook

How to update an existing `<item>` in `russian_shops.xml`: a moved or dead
link, a rebrand, changed contacts, hours or icon. Typical entry points: a CI
warning about an item link, a known rebrand, or the user asking to update
item X. First read the item's current tags in full — the checker guarantees
name uniqueness inside a group, so group + `name` identifies the item — then
research, then apply a minimal diff. The scenarios below are generic; trust
the methods, re-verify every fact for the concrete item at hand.

## 1. Triage the link: dead, bot-wall, geo-block, moved, soft-404?

Never conclude "dead" from a single status code. curl with a browser
User-Agent, look at the actual response, and match it against this matrix:

| Signal | Verdict | What it means | What to do |
| --- | --- | --- | --- |
| `401` / `403` / `498` | alive behind a bot-wall | large retail / e-commerce sites commonly answer these codes to non-browser clients | Do not touch the link. Confirm liveness another way: Wikidata, socials, search. |
| timeout / curl exit `000` | geo-block or actually dead | an alive site can be unreachable from one network | Retry from another network before judging; check Wikidata/socials meanwhile. |
| `404` on a known page | page moved — or the chain died | the site may still be alive | Find the new page via the site's own navigation («Магазины/Адреса/Карта»). |
| `301`/`302` redirect | page moved | the target must be inspected, not assumed | Follow the chain, inspect what the target actually serves, link to the final good page. |
| soft-404: status `404` but full HTML (header, footer, socials) | site alive, page moved | the server serves a full page with a 404 status | Treat as "moved", not "dead": walk the live site's navigation to find the real page. |

A `404` on the site's front page with no working alternative page is the only
signal that means "likely dead chain" — and even then the decision belongs to
the human (section 4).

## 2. Wikidata and Wikipedia check

Query the Wikidata API with an explicit `User-Agent` header — the default
python UA gets a plain 403. Sleep 1-2 s between wiki requests: the API
rate-limits with 429.

Properties to read on the brand's entity:

- `P856` — official site(s): compare with the item's `contact:website` and
  `<link>`; a changed or extra value hints at a move or rebrand.
- `P576` — dissolution date: the brand closed.
- `P1366` — replaced by: the successor entity.

For brand history (bankruptcies, rebrands) use ru.wikipedia API extracts on
the company's article.

**Wiki lags reality.** The absence of `P576` does not mean the brand is
alive: a chain can be long bankrupt in the news with its customer site shut
down, while its Wikidata entity still carries no dissolution date. Likewise
an entity can be stale after a rebrand, with the new brand having no QID at
all yet. Wiki can lag reality by months, so corroborate a closure with news
and the dead customer site before concluding. When wiki and the live
site/news disagree, trust the site and the news, and record the discrepancy
in the report to the human.

## 3. Rebrand scenario: a chain rebrands (old-brand.example → new-brand.example)

The evidence chain, in the order to establish it:

1. `old-brand.example` now carries the new brand's identity on its homepage —
   the chain was renamed.
2. A known `old-brand.example/locator` page 301-redirects to
   `new-brand.example/locator`.
3. `new-brand.example/locator` is a soft-404 (404 status, full HTML with
   footer/socials) — a moved page, not a dead chain.
4. Navigation on the live `new-brand.example` leads to the real locator:
   `new-brand.example/map`.
5. Wikidata still describes the old brand; the new brand likely has no QID
   yet.

The item diff — the whole rebrand, nothing else changes:

- `name` / `ru.name` → the new brand name (language rules in
  [conventions.md](conventions.md)).
- `brand` value → the new brand.
- `<link href>` → `https://new-brand.example/map`; `contact:website`, if the
  item carries one, → `https://new-brand.example`.
- Icon → change only if the old one is brand-specific; pick the new one via
  [icon-pipeline.md](icon-pipeline.md).
- `brand:wikidata` → drop the stale old-brand QID rather than reuse the old
  entity for the new brand; the successor has no QID yet, so leave
  `<!-- TODO: brand:wikidata when the new brand gets a QID -->`
  (repository comment style, see conventions.md).
- Position → a rename changes the sort key, so re-place the item per the
  sorting rules in [conventions.md](conventions.md). This is the only case
  in update mode where the item's position changes.

## 4. Closure scenario — escalate, never delete

Rule: if the chain is dead — a closure confirmed by news and the dead
customer site (never by wiki alone, see section 2) with no successor — item
removal is always a human decision. The agent collects the evidence (status
codes, wiki/news extracts, dates), proposes removal, and stops. It never
deletes an item on its own. The same applies to "doubtful": if the triage
above still leaves the picture unclear, stop and ask.

## 5. Minimal-diff rule

- Change only confirmed facts: link, brand/name, contacts, hours, icon.
- Do not restyle: attribute order, whitespace and chunk usage stay as they
  are — fix them only if actually broken.
- Rebrand = name + brand + link (+ icon when needed) + dropping the stale
  `brand:wikidata` when the successor has no QID.
- New contact values must come from verified sources, exactly as when
  creating an item ([item-template.md](item-template.md)): never invent
  phones, hours or a QID.

## 6. Verify and commit

On every update, in the literan-moscow clone:

```bash
python3 scripts/check_presets.py --no-http russian_shops.xml
```

Fix every checker failure before committing; run `xmllint --schema` as well
when xmllint is available. Commit only on the user's explicit word, one-line
message in the repository's style; do not touch the preset version — CI
increments it.
