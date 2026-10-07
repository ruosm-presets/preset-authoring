# Update playbook

How to update an existing `<item>` in `russian_shops.xml`: a moved or dead
link, a rebrand, changed contacts, hours or icon. Typical entry points: a CI
warning about an item link, a known rebrand, or the user asking to update
item X. First read the item's current tags in full — the checker guarantees
name uniqueness inside a group, so group + `name` identifies the item — then
research, then apply a minimal diff. The precedents below (SUBJOY, Связной)
come from a real curation session; trust the methods, re-verify the facts.

## 1. Triage the link: dead, bot-wall, geo-block, moved, soft-404?

Never conclude "dead" from a single status code. curl with a browser
User-Agent, look at the actual response, and match it against this matrix:

| Signal | Verdict | Precedents | What to do |
| --- | --- | --- | --- |
| `401` / `403` / `498` | alive behind a bot-wall | Ozon → 403, Auchan / DNS / Sportmaster → 401, Wildberries → 498 | Do not touch the link. Confirm liveness another way: Wikidata, socials, search. |
| timeout / curl exit `000` | geo-block or actually dead | okmarket.ru, zhivika, teremok.ru unreachable from one network but alive | Retry from another network before judging; check Wikidata/socials meanwhile. |
| `404` on a known page | page moved — or the chain died | subjoy.ru/restaurants | The site may still be alive: find the new page via the site's own navigation («Магазины/Адреса/Карта»). |
| `301`/`302` redirect | page moved | subway.ru/restaurants → subjoy.ru/restaurants | Follow the chain, inspect what the target actually serves, link to the final good page. |
| soft-404: status `404` but full HTML (header, footer, socials) | site alive, page moved | subjoy.ru/restaurants serves full HTML with footer/socials | Treat as "moved", not "dead": walk the live site's navigation — the real locator page was subjoy.ru/map. |

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

For brand history (bankruptcies, rebrands) use ru.wikipedia API extracts,
e.g. the article «Связной (компания)».

**Wiki lags reality.** The absence of `P576` does not mean the brand is alive:

- Q65371 (Связной) has no `P576`, yet ru.wikipedia records the Feb 2024
  bankruptcy of ООО «Сеть Связной» and the customer site is shut down.
- Q244457 (Subway) is stale for the RU chain (rebranded to SUBJOY), and
  SUBJOY itself has no QID yet.

When wiki and the live site/news disagree, trust the site and the news, and
record the discrepancy in the report to the human.

## 3. Rebrand case: Subway → SUBJOY

The evidence chain, in the order it was established:

1. `subway.ru` now shows SUBJOY branding — the RU chain was renamed.
2. `subway.ru/restaurants` 301-redirects to `subjoy.ru/restaurants`.
3. `subjoy.ru/restaurants` is a soft-404 (404 status, full HTML with
   footer/socials) — a moved page, not a dead chain.
4. Navigation on the live `subjoy.ru` leads to the real locator:
   `subjoy.ru/map`.
5. Wikidata Q244457 (Subway) is stale for the RU chain; SUBJOY has no QID
   yet.

The item diff — the whole rebrand, nothing else changes:

- `name` / `ru.name` → the new brand name (language rules in
  [conventions.md](conventions.md)).
- `brand` value → SUBJOY.
- `<link href>` → `https://subjoy.ru/map`; `contact:website`, if the item
  carries one, → `https://subjoy.ru`.
- Icon → change only if the old one is brand-specific; pick the new one via
  [icon-pipeline.md](icon-pipeline.md).
- `brand:wikidata` → drop the stale Q244457; the successor has no QID yet,
  so leave `<!-- TODO: brand:wikidata when SUBJOY gets a QID -->`
  (repository comment style, see conventions.md).
- Position → a rename changes the sort key, so re-place the item per the
  sorting rules in [conventions.md](conventions.md). This is the only case
  in update mode where the item's position changes.

## 4. Closure case: Связной — escalate, never delete

Facts: ru.wikipedia «Связной (компания)» — Feb 2024 bankruptcy of
ООО «Сеть Связной», customer site shut down; wikidata Q65371 still has no
`P576` (wiki lags reality).

Rule: if the chain is dead — a confirmed closure with no successor — item
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
