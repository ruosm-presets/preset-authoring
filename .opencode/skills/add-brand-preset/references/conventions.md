# literan-moscow conventions — operational digest

Source of truth: `CONTRIBUTING.md` in the
[literan-moscow](https://github.com/ruosm-presets/literan-moscow) repository.
This file is an operational digest for an agent editing `russian_shops.xml`;
if anything conflicts, CONTRIBUTING.md wins.

## Screen-form field order

Order the input fields of a screen form exactly like the built-in JOSM
presets:

`shop`/`amenity` (any top-level tags) → `name` → `name:ru`/`name:en` →
`official_name` → `brand` → `contact:website` → `contact:phone` →
`opening_hours` → `level` → `operator`.

## Internal structure of an `<item>`

1. Start with a blank line element: `<space />`.
2. Input fields the user can edit (`<combo>`, `<reference>`, `<text>`), in the
   screen-form order above.
3. Fixed tags the user cannot change: the `<key ... />` block. When order is
   not important, sort tags alphabetically.
4. `<link ... />` and shared `<reference ref="...">` chunks (which usually
   bundle more screen-form fields) go at the bottom of the form.

## Sorting

Menu groups and items are ordered so that
[JOSM's own sorting](https://josm.openstreetmap.de/wiki/Help/Preferences/Map#Activatepresetsfromavailablepresets)
does not reshuffle them:

1. Items form subgroups: numbers first (`36,6`), then Latin-script (`CMD`,
   `Subway`, `IL Патио`), then Cyrillic.
2. Inside a group, sort alphabetically: `0-9`, `A-z`, `А-я`.
3. If a name exists in both scripts, sort by the Russian one:
   `Domino’s Pizza` / `Домино’c Пицца` sorts by `Домино’c Пицца`.

Quoted examples in this digest (`36,6`, `CMD`, `Subway`, `IL Патио`,
`Domino’s Pizza`, `Л'Этуаль` / `L'etoile`) are verbatim from literan-moscow
CONTRIBUTING.md — illustrative sorting fixtures, not authored precedents.

Groups in the menu follow this macro-order: other groups → `<separator>` →
government / medicine / education → `<separator>` → city entities.

## Tag attribute order

Always write tag attributes in this order: `key`/`ref` → `text`/`name` →
`ru.text`/`ru.name` → `values`/`value` → `ru.display_values` → `default` →
`type` → `preset_name_label` → `name_context` → `icon`. The `icon` attribute
is always last.

## Language

English is the base language of the preset. Put English text into `name`,
`text`, `display_values` and similar attributes; if an English name does not
exist (e.g. a shop name), transliterate or transcribe it. Russian duplicates
go into `ru.*` attributes: `ru.name`, `ru.text`, `ru.display_values`.

## Whitespace

- Indent with spaces, never tabs.
- A tag closed on the same line gets a space before the slash: `<space />`,
  `<key value="Л'Этуаль" />`.
- A tag not closed on the same line gets no trailing space:

  ```xml
  <item name="L'etoile">
  </item>
  ```

## Comments

- `<!-- TODO: ... -->` marks things to be done; `<!-- NB: ... -->` marks
  things to pay attention to.
- Place a comment either above the block it describes (e.g. above a group of
  related chunks), or at the end of the line — for `<item>`s, at the end of
  the line with the closing `</item>` (convenient when tags are folded in the
  editor).
- Do not keep commented-out code around for long: git keeps the history.
  Clean such blocks up once you are sure they are obsolete.

## Chunks (шаблоны)

Reuse chunks as much as possible: renaming a field in one chunk is better than
editing it in 50 items (and missing one). Chunk definitions live in the
`<!-- блок с шаблонами (chunks) -->` block at the top of `russian_shops.xml`;
composite chunks reference other chunks (e.g. `name_website_phone`,
`level_wheelchair_address`, `name_website_phone_level_wheelchair_address`).
Always re-read the current chunk list from the file when composing an item —
never rely on a remembered list.
