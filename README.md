# preset-authoring

An opencode skill (`add-brand-preset`) that walks an AI agent through adding a
brand/chain POI `<item>` to the
[literan-moscow](https://github.com/ruosm-presets/literan-moscow) JOSM tagging
preset: brand research, tags, icon selection, insertion at the right position,
and checks. The preset repository gets no AI tooling of its own — all
executable checks stay in literan-moscow (`scripts/check_presets.py`); this
repository is markdown only. Background, scope and process design:
[docs/spec.md](docs/spec.md).

## Installation

Clone this repository anywhere, e.g. `~/Dev/preset-authoring`:

```bash
git clone <url-of-this-repository> ~/Dev/preset-authoring
```

Register the skill directory in the opencode global config,
`~/.config/opencode/opencode.json`, via `skills.paths`:

```json
{
  "skills": {
    "paths": ["~/Dev/preset-authoring/.opencode/skills"]
  }
}
```

If your config already has a `skills` section, append the path to the existing
`paths` array instead of replacing it. The path in the snippet must match the
location you cloned to.

## Restart opencode

The config is not hot-reloaded: after editing `opencode.json`, restart
opencode. The `add-brand-preset` skill appears only after the restart.

## Requirements

- [opencode](https://opencode.ai) (or any agent that reads `SKILL.md` files).
- A local clone of [literan-moscow](https://github.com/ruosm-presets/literan-moscow):
  the skill edits `russian_shops.xml` and `pics/icons/` there and runs
  `python3 scripts/check_presets.py --no-http russian_shops.xml` inside that
  clone.
