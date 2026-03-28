# Ethiopic Bible Data

Structured Bible text in Amharic, English, and Ge'ez (Ethiopic), and stored as clean, machine-readable JSON.

## What's included

- **66 books** — 39 Old Testament + 27 New Testament
- **3 languages** — Amharic (`am`), English (`en`), Ge'ez (`gez`)
- **Per-book JSON files** in `data/old-testament/` and `data/new-testament/`
- **Minified distribution bundles** in `minified/` (complete, OT-only, NT-only, with optional gzip and language filtering)
- Merged verse metadata for consumer deduplication

## Quick start

### Use the data directly

```python
import json

with open("data/old-testament/01-genesis.json", encoding="utf-8") as f:
    genesis = json.load(f)

verse = genesis["chapters"][0]["verses"][0]
print(verse["text"]["am"])  # Amharic text
print(verse["text"]["en"])  # English text
```

## Project structure

```
data/
  old-testament/    39 per-book JSON files
  new-testament/    27 per-book JSON files
minified/           Minified bundles (.json and .json.gz)
docs/               Documentation
```

## Documentation

- **[Data Guide](docs/DATA_GUIDE.md)** — Schema reference, code examples, merged verse handling, and book reference tables. Start here if you want to build an app or API with this data.

## License

The Bible text is sourced from [ethiopicbible.com](https://www.ethiopicbible.com). Please respect the source site's terms of use.
