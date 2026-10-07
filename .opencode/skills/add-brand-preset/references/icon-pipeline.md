# Icon pipeline

How to pick or create the icon for a new `<item>` in `russian_shops.xml`.
Follow the steps in order; stop at the first step that produces an icon.

## 1. Reuse an existing icon

Run `ls pics/icons/` in the literan-moscow clone and pick a file by meaning of
the POI. Existing names show the expected style, both `.svg` and `.png`
occur: `burger.svg`, `bubble_tea.svg`, `department_store.svg`,
`clothes_dryer.png`. If nothing fits semantically, continue to step 2.

## 2. Add a new icon from Twemoji

Pick an emoji whose glyph matches the POI's meaning, then download it by
codepoint from the Twemoji CDN:

`https://raw.githubusercontent.com/jdecked/twemoji/v15.1.0/assets/svg/<code>.svg`

`<code>` is the lowercase hex codepoints joined with `-`. Keycap files have
no leading zeros: `35-20e3`, not `0035-20e3`. ZWJ sequences are written the
same way, codepoint by codepoint with `-`, including the ZWJ itself
(`200d`). Save the file into `pics/icons/` in the clone, then check the
download actually succeeded: the file must start with `<svg` (an HTML error
page means the URL is wrong — re-check the codepoint).

## 3. Name the file

Name the file after the glyph's meaning, not after the brand/item — icons are
shared across items: `baby.svg`, `chicken_leg.svg`, not `acme_coffee.svg`.
The name must match `^[a-z0-9_]+\.(svg|png)$` (lowercase snake_case,
`.svg` or `.png`).

## 4. URL in the preset

Use only the GitHub Pages base of the preset repository:
`https://ruosm-presets.github.io/literan-moscow/pics/icons/<file>`. Never
link Wikimedia or any other host — externally hosted icons are rejected.

## 5. Verification

Nothing to check by hand: the literan-moscow checker
(`python3 scripts/check_presets.py --no-http russian_shops.xml`)
automatically verifies that the icon URL is on the Pages base, that the file
exists under `pics/icons/`, and that the file name matches the format above.
Fix every checker failure before committing.
