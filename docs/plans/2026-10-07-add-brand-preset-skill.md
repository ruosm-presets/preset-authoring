# add-brand-preset Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Публичный репозиторий `preset-authoring` со скиллом `add-brand-preset`, который ведёт AI-агента через добавление бренда в пресет literan-moscow.

**Architecture:** Инструкционный артефакт (SKILL.md + 3 справочника + README), ноль кода. Весь executable-функционал остаётся в literan-moscow (`scripts/check_presets.py`). Скилл подключается глобально через `skills.paths`.

**Tech Stack:** opencode skill format (SKILL.md frontmatter), markdown, git.

**Spec:** `docs/spec.md` (в этом репозитории)

## Global Constraints

- Имя скилла `add-brand-preset`, папка совпадает с именем, name lowercase-hyphen ≤64.
- `description` обязателен, от третьего лица, содержит триггер-ключи (включая русские: «добавь бренд в пресет»).
- Никакого кода/скриптов в этом репозитории.
- literan-moscow получает ТОЛЬКО абзац в CONTRIBUTING.md.
- Формат имён иконок: `^[a-z0-9_]+\.(svg|png)$`.
- База иконок: `https://ruosm-presets.github.io/literan-moscow/pics/icons/`.
- Источник Twemoji: `https://raw.githubusercontent.com/jdecked/twemoji/v15.1.0/assets/svg/<code>.svg` (keycaps без ведущих нулей: `35-20e3`).
- PII никогда не попадает в артефакты; guardrails — раздел SKILL.md.

## Review Focus

1. Скилл невидим для агента (плохой frontmatter → отфильтрован) — Task 5 проверяет структуру frontmatter явно.
2. Агент выдумывает данные (телефоны, wikidata, часы) — Task 5 требует дословный guardrail «только из источников».
3. Иконка залита на Wikimedia вместо Pages-URL — Task 4 закрепляет домен-правило.
4. Мёртвый бренд добавлен без вопроса человеку — Task 5 требует правило эскалации с примерами SUBJOY/Связной.
5. Пользователь правил конфиг, но скилл «не появился» (нет перезапуска) — Task 1 требует упоминание перезапуска в README.

---

### Task 1: README.md — назначение и установка

**Files:**
- Create: `README.md`

**Interfaces:**
- Produces: канонический JSON-сниппет установки, на который ссылается SKILL-описание и CONTRIBUTING-абзац (Task 6).

- [ ] **Step 1: Написать README.md**

Секции: что это (1 абзац, ссылка на docs/spec.md), установка (git clone → `skills.paths` сниппет для `~/.config/opencode/opencode.json`, точно как в Global Constraints), перезапуск opencode (конфиг не горячо перезагружается), требования (клон literan-moscow для работы скилла).

- [ ] **Step 2: Проверить сниппет**

Run: `python3 -c "import json,re; s=open('README.md').read(); m=re.search(r'\`\`\`jsonc?\n(.*?)\`\`\`', s, re.S); json.loads(m.group(1))" && echo OK`
Expected: `OK` (JSON валиден)

- [ ] **Step 3: Коммит**

```bash
git add README.md && git commit -m "docs: README with installation via skills.paths"
```

### Task 2: reference/conventions.md — выжимка правил literan-moscow

**Files:**
- Create: `reference/conventions.md`

**Interfaces:**
- Consumes: правила из `~/Dev/literan-moscow/CONTRIBUTING.md` (источник истины).
- Produces: операциональная выжимка, на которую ссылается SKILL.md шаг 4.

- [ ] **Step 1: Написать conventions.md**

Разделы: порядок полей экранной формы; сортировка (числа → латиница → кириллица, внутри групп по алфавиту, приоритет ru.name); порядок атрибутов тега (key/ref → text/name → ru.text/ru.name → values → ru.display_values → default → type → preset_name_label → name_context → icon — иконка последняя); язык (en основной, ru.* — дубль); отступы пробелами, самозакрытые теги с пробелом `<space />`, открывающие без; комментарии (TODO/NB, над блоком или в конце строки); чанки — максимально переиспользовать, существующий список чанков снимать с файла, не помнить наизусть.

- [ ] **Step 2: Сверить с оригиналом**

Run: `grep -c 'name_context' reference/conventions.md && grep -c 'name_context' ~/Dev/literan-moscow/CONTRIBUTING.md`
Expected: оба ≥1 (каждое правило порядка атрибутов прослеживается до CONTRIBUTING)

- [ ] **Step 3: Коммит**

```bash
git add reference/conventions.md && git commit -m "docs: conventions digest from literan-moscow CONTRIBUTING"
```

### Task 3: reference/item-template.md — скелет пункта

**Files:**
- Create: `reference/item-template.md`

**Interfaces:**
- Produces: шаблон `{{placeholder}}`, который SKILL.md шаг 2-4 использует для сборки `<item>`.

- [ ] **Step 1: Написать шаблон**

Скелет `<item name="{{name}}" ru.name="{{ru_name}}" ...>` c плейсхолдерами `{{brand}} {{wikidata}} {{website}} {{phone}} {{opening_hours}} {{icon}} {{group}}`; построчные комментарии-назначение секций (`<space />`, комбо-поля ввода, `<key>` блок, `<link>`, `<reference>` внизу); пример «хорошо» (реюз чанков `name_website_phone_level_wheelchair_address` и т.п.) и «плохо» (всё инлайн) — по одному короткому.

- [ ] **Step 2: Проверить плейсхолдеры**

Run: `grep -o '{{[a-z_]*}}' reference/item-template.md | sort -u`
Expected: набор включает brand, wikidata, website, phone, opening_hours, icon, group

- [ ] **Step 3: Коммит**

```bash
git add reference/item-template.md && git commit -m "docs: item skeleton template with placeholders"
```

### Task 4: reference/icon-pipeline.md — конвейер иконок

**Files:**
- Create: `reference/icon-pipeline.md`

**Interfaces:**
- Produces: правило выбора иконки для SKILL.md шаг 3; домен и regex из Global Constraints.

- [ ] **Step 1: Написать pipeline**

Порядок: (1) реюз — `ls pics/icons/` в клоне literan-moscow, подбор по смыслу; (2) новая — Twemoji по кодопоинту, CDN-URL из Global Constraints, ключевые нюансы: keycap-файлы без ведущих нулей, ZWJ-последовательности через дефисы, скачать и проверить что файл начинается с `<svg`; (3) имя `snake_case` по смыслу глифа (не по пункту), regex; (4) URL — только Pages-домен из Global Constraints, никаких Wikimedia; (5) чекер проверит существование файла и формат имени автоматически.

- [ ] **Step 2: Проверить якорные строки**

Run: `grep -c 'ruosm-presets.github.io' reference/icon-pipeline.md && grep -cE '\^\[a-z0-9_\]' reference/icon-pipeline.md`
Expected: обе ≥1

- [ ] **Step 3: Коммит**

```bash
git add reference/icon-pipeline.md && git commit -m "docs: icon pipeline (reuse -> twemoji -> snake_case -> pages url)"
```

### Task 5: SKILL.md — ядро скилла

**Files:**
- Create: `.opencode/skills/add-brand-preset/SKILL.md`

**Interfaces:**
- Consumes: все reference/*.md (относительные пути `../../../reference/...` от SKILL.md), чекер `scripts/check_presets.py` в клоне literan-moscow.
- Produces: скилл, видимый opencode после установки.

- [ ] **Step 1: Написать SKILL.md**

Frontmatter: `name: add-brand-preset`, description из spec (третье лицо, триггеры RU+EN). Тело: 6 шагов процесса из spec (исследование → данные → иконка → вставка → верификация → коммит), каждый шаг ссылается на свой reference-файл; раздел Guardrails дословно по spec (PII; мёртвый/сомнительный бренд → эскалация с прецедентами SUBJOY и «Связной»; не выдумывать данные; только клоны; push по слову); правило «вопросы по одному».

- [ ] **Step 2: Проверить структуру frontmatter**

Run: `head -5 .opencode/skills/add-brand-preset/SKILL.md | grep -E '^name: add-brand-preset$' && grep -c 'добавь бренд' .opencode/skills/add-brand-preset/SKILL.md`
Expected: обе проверки ≥1

- [ ] **Step 3: Коммит**

```bash
git add .opencode/skills/add-brand-preset/SKILL.md && git commit -m "feat: add-brand-preset skill core"
```

### Task 6: Приёмочные тесты в песочнице + CONTRIBUTING-абзац в literan-moscow

**Files:**
- Modify: `~/Dev/literan-moscow/CONTRIBUTING.md` (добавить абзац в конец раздела «Прочее»)

**Interfaces:**
- Consumes: установленный скилл (skills.paths), клон literan-moscow.

- [ ] **Step 1: Установить скилл локально**

Добавить `skills.paths` в `~/.config/opencode/opencode.json` (сохранить существующие поля), перезапустить opencode по указанию пользователя, убедиться что скилл виден.

- [ ] **Step 2: Dry-run на живом бренде**

В ветке-песочнице literan-moscow: прогнать скилл на бренде по выбору пользователя; критерии приёмки из spec (позиция, чекер зелёный, иконка корректна). Ожидаемое вмешательство человека — только ответы на вопросы скилла.

- [ ] **Step 3: Абзац в CONTRIBUTING.md literan-moscow**

Текст: «Есть AI-скилл для добавления брендов: <github-url репозитория после публикации>». Коммит в literan-moscow отдельным сообщением.

- [ ] **Step 4: Финальная проверка плана репозитория**

Run: `find . -name '*.md' | sort`
Expected: `./README.md ./docs/spec.md ./docs/plans/2026-10-07-add-brand-preset-skill.md ./reference/conventions.md ./reference/icon-pipeline.md ./reference/item-template.md ./.opencode/skills/add-brand-preset/SKILL.md`
