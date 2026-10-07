# `<item>` skeleton template

Skeleton for assembling a brand `<item>` in `russian_shops.xml`. Fill the
`{{placeholder}}` values from verified brand research; never invent them
(phones, website, wikidata, opening hours — only from sources). The item goes
into group `{{group}}`; its position inside the group follows the sorting
rules in [conventions.md](conventions.md); the icon follows
[icon-pipeline.md](icon-pipeline.md).

## Brand card (вход-опросник)

Start this card BEFORE assembling the `<item>`. Обход формы — строго
сверху вниз, одно поле — один вопрос; ответ «не знаю» оставляет пробел
для авто-исследования.

| # | Field              | Source                                                 | Required |
|---|--------------------|--------------------------------------------------------|----------|
| 1 | name / ru.name     | user → site title/footer                               | yes      |
| 2 | website            | user (starting point of research)                      | yes      |
| 3 | city / context     | user                                                   | no       |
| 4 | type (shop/cuisine)| user → site → ask only if ambiguous; mixed → picker combo, not a fixed `<key>` | yes |
| 5 | locator page       | site navigation (Магазины/Адреса/Карта)                | yes      |
| 6 | email / socials    | site footer                                            | no       |
| 7 | wikidata QID       | search by name; may not exist — then omit              | no       |
| 8 | opening hours      | site; otherwise skip — never invent                    | no       |
| 9 | icon               | agent by meaning: reuse → Twemoji (icon-pipeline.md)   | yes      |

An empty optional field is better than invented data. Показывай заполненную
карточку целиком и получи подтверждение до вставки пункта.

## Skeleton

Section purposes are commented above each block, in the repository's comment
style:

```xml
<item name="{{name}}" ru.name="{{ru_name}}" type="node" preset_name_label="true" icon="{{icon}}">
  <!-- blank line at the top of the screen form -->
  <space />
  <!-- screen-form input fields (<combo>/<reference>/<text>), canonical order -->
  <!-- brand is normally a fixed <key key="brand" ... />; use <combo> only for multi-brand items -->
  <combo key="brand" text="Brand" values="{{brand}}" />
  <combo key="opening_hours" text="Opening Hours" values="{{opening_hours}}" />
  <!-- if no chunk covers a field, add it inline (last resort — see the bad example) -->
  <!-- <combo key="contact:phone" text="Phone number" default="{{phone}}" /> -->
  <!-- fixed tags the user cannot change -->
  <key key="brand:wikidata" value="{{wikidata}}" /> <!-- only when the brand exists in Wikidata -->
  <!-- link to the brand's official place locator -->
  <link href="{{website}}" text="Official website (place locator)" ru.text="Ссылка на сайт (часы работы, координаты и т.п.)" />
  <!-- shared chunks go at the bottom of the form -->
  <reference ref="name_website_phone" />
</item>
```

Attributes of `<item>` follow the attribute order from
[conventions.md](conventions.md) (`name` → `ru.name` → `type` →
`preset_name_label` → `icon`, icon last).

## Good: reuse chunks

```xml
<item name="Acme Coffee" ru.name="Акме Кофе" type="node" preset_name_label="true" icon="https://ruosm-presets.github.io/literan-moscow/pics/icons/coffee.svg">
  <space />
  <combo key="cuisine" text="Cuisine" values="coffee_shop" />
  <key key="amenity" value="cafe" />
  <key key="brand" value="Acme Coffee" />
  <key key="contact:website" value="https://acme.example" />
  <link href="https://acme.example/cafes" text="Official website (place locator)" ru.text="Ссылка на сайт (часы работы, координаты и т.п.)" />
  <reference ref="name_website_phone_oh1_level_wheelchair_address" />
</item>
```

Name, website, phone, opening hours, level, wheelchair and address inputs all
come from existing chunks — one reference instead of six hand-written fields.

## Bad: everything inline

```xml
<item name="Acme Coffee" ru.name="Акме Кофе" type="node" preset_name_label="true" icon="https://ruosm-presets.github.io/literan-moscow/pics/icons/coffee.svg">
  <space />
  <text key="name" text="Name" />
  <combo key="contact:website" text="Website" values="https://acme.example" />
  <combo key="contact:phone" text="Phone number" default="{{phone}}" />
  <combo key="opening_hours" text="Opening Hours" values="{{opening_hours}}" />
  <text key="level" text="Level" ru.text="Этаж" />
  <combo key="wheelchair" text="Wheelchairs" values="yes,limited,no" />
  <preset_link preset_name="Address" />
  <key key="amenity" value="cafe" />
  <key key="brand" value="Acme Coffee" />
</item>
```

Same fields as the chunks already provide, duplicated inline: any future
change to these fields must be repeated in every such item, and some will be
missed. Before writing a field inline, re-read the chunk list at the top of
`russian_shops.xml` and reuse.
