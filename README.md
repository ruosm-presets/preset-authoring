# preset-authoring

Скилл для opencode (`add-brand-preset`), который ведёт AI-агента через
добавление `<item>` бренда/сети POI в JOSM-заготовки
[literan-moscow](https://github.com/ruosm-presets/literan-moscow):
исследование бренда, теги, подбор иконки, вставка в нужную позицию и
проверки. Пресет-репозиторий не получает собственной AI-обвязки — все
исполняемые проверки остаются в literan-moscow (`scripts/check_presets.py`);
этот репозиторий состоит только из markdown. Предыстория, рамки задачи и
устройство процесса: [docs/spec.md](docs/spec.md).

## Установка

Склонируйте репозиторий куда угодно, например в `~/Dev/preset-authoring`:

```bash
git clone https://github.com/ruosm-presets/preset-authoring ~/Dev/preset-authoring
```

Зарегистрируйте каталог скиллов в глобальном конфиге opencode,
`~/.config/opencode/opencode.json`, через `skills.paths`:

```json
{
  "skills": {
    "paths": ["~/Dev/preset-authoring/.opencode/skills"]
  }
}
```

Если в конфиге уже есть секция `skills`, допишите путь в существующий
массив `paths`, а не заменяйте его. Путь в примере должен совпадать с
местом, куда вы склонировали репозиторий.

## Установка из удалённого репозитория

Чтобы пользоваться скиллом, не держа development-копию этого репозитория,
установите его из опубликованного git-репозитория: shallow-клон содержит
только файлы скилла и ничего больше, а `git pull` держит их актуальными.

### Вариант 1: shallow-клон + `skills.paths` (рекомендуется)

```bash
git clone --depth 1 https://github.com/ruosm-presets/preset-authoring.git ~/tools/preset-authoring
```

Зарегистрируйте каталог скиллов в глобальном конфиге opencode,
`~/.config/opencode/opencode.json` (или `.jsonc`), через `skills.paths`:

```json
{
  "skills": {
    "paths": ["~/tools/preset-authoring/.opencode/skills"]
  }
}
```

`skills.paths` сканируется рекурсивно по маске `**/SKILL.md`, поэтому
подхватывается каждый скилл из репозитория. После правки конфига
перезапустите opencode (конфиг не перезагружается на лету). Обновление
скилла позже: `git pull` в клоне, затем снова перезапустите opencode.

### Вариант 2: символьная ссылка (без правки конфига)

opencode сам обнаруживает скиллы по пути
`~/.config/opencode/skills/<name>/SKILL.md`, поэтому симлинк в клон
работает без правки конфига:

```bash
git clone --depth 1 https://github.com/ruosm-presets/preset-authoring.git ~/tools/preset-authoring
mkdir -p ~/.config/opencode/skills
ln -s ~/tools/preset-authoring/.opencode/skills/add-brand-preset ~/.config/opencode/skills/add-brand-preset
```

После создания симлинка перезапустите opencode. Обновление то же самое:
`git pull` в клоне, затем перезапуск opencode. Чтобы убрать скилл, удалите
симлинк: `rm ~/.config/opencode/skills/add-brand-preset`.

## Перезапуск opencode

Конфиг не перезагружается на лету: после правки `opencode.json`
перезапустите opencode. Скилл `add-brand-preset` появляется только после
перезапуска.

## Требования

- [opencode](https://opencode.ai) (или любой агент, читающий файлы
  `SKILL.md`).
- Локальный клон [literan-moscow](https://github.com/ruosm-presets/literan-moscow):
  скилл правит в нём `russian_shops.xml` и `pics/icons/` и запускает внутри
  клона `python3 scripts/check_presets.py --no-http russian_shops.xml`.
