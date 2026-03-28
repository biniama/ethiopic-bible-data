# Amharic Bible Data Guide

A guide for developers building APIs, apps, and tools with the Amharic Bible dataset.

## Overview

This dataset contains the complete Bible in three languages — **Amharic (am)**, **English (en)**, and **Ge'ez (gez)** — structured as JSON. It is designed for easy consumption by APIs, mobile apps, web apps, and search tools.

- **66 books** (39 Old Testament + 27 New Testament)
- **1,189 chapters**, **~31,000 verses**
- Per-book JSON files for granular loading, or bundled files for bulk access
- Extensible structure supporting additional languages and audio

---

## Quick Start

### Load a single book

```python
import json

with open("data/old-testament/01-genesis.json", encoding="utf-8") as f:
    genesis = json.load(f)

# Access verse text
verse = genesis["chapters"][0]["verses"][0]
print(verse["text"]["am"])  # በመጀመሪያ እግዚአብሔር ሰማይንና ምድርን ፈጠረ።
print(verse["text"]["en"])  # In the beginning God created the heaven and the earth.
```

### Load the complete Bible (minified bundle)

```python
with open("dist/bible.json", encoding="utf-8") as f:
    bible = json.load(f)

for book in bible["books"]:
    print(f"{book['book_short_name_en']}: {len(book['chapters'])} chapters")
```

### JavaScript / Node.js

```javascript
const bible = require('./dist/bible.json');

const genesis = bible.books[0];
const verse1 = genesis.chapters[0].verses[0];
console.log(verse1.text.am); // በመጀመሪያ እግዚአብሔር ሰማይንና ምድርን ፈጠረ።
```

---

## File Organization

### Per-book files (for selective loading)

```
data/
├── metadata.json
├── old-testament/
│   ├── 01-genesis.json
│   ├── 02-exodus.json
│   ├── ...
│   └── 39-malachi.json
└── new-testament/
    ├── 40-matthew.json
    ├── ...
    └── 66-revelation.json
```

Filename format: `{number:02d}-{english-name-lowercase}.json`

### Bundled files (for bulk loading)

```
dist/
├── bible.json               # Complete Bible, all languages (13 MB)
├── bible.json.gz             # Gzipped (3.5 MB)
├── old-testament.json        # Old Testament only (10 MB)
├── old-testament.json.gz     # Gzipped (2.7 MB)
├── new-testament.json        # New Testament only (2.7 MB)
├── new-testament.json.gz     # Gzipped (741 KB)
├── bible_am.json             # Amharic only (6.5 MB)
├── bible_am.json.gz          # Gzipped (1.4 MB)
├── bible_am_en.json          # Amharic + English (10.8 MB)
└── bible_am_en.json.gz       # Gzipped (2.9 MB)
```

All `dist/` files are minified (no whitespace) for compact transfer.

---

## Data Schema

### Per-Book File

```json
{
  "book_number": 1,
  "book_name_am": "ኦሪት ዘፍጥረት",
  "book_short_name_am": "ዘፍ",
  "book_name_en": "Genesis",
  "book_short_name_en": "Gen",
  "book_name_geez": "ኦሪት ዘፍጥረት",
  "testament": "old",
  "sources": ["ethiopicbible.com"],
  "chapters": [ ... ]
}
```

| Field | Type | Description |
|---|---|---|
| `book_number` | int | Canonical book number (1-66) |
| `book_name_am` | string | Full Amharic book name |
| `book_short_name_am` | string | Amharic abbreviation (e.g. `ዘፍ`) |
| `book_name_en` | string | English book name |
| `book_short_name_en` | string | English abbreviation (e.g. `Gen`) |
| `book_name_geez` | string or null | Ge'ez book name (null if unavailable) |
| `testament` | string | `"old"` or `"new"` |
| `sources` | string[] | Data sources that contributed to this book |
| `chapters` | array | Array of chapter objects |

### Chapter Object

```json
{
  "chapter": 1,
  "sections": [],
  "verses": [ ... ]
}
```

| Field | Type | Description |
|---|---|---|
| `chapter` | int | Chapter number (1-based) |
| `sections` | array | Section/heading groupings (reserved for future use) |
| `verses` | array | Array of verse objects |

The `sections` array is currently empty. It is reserved for adding titled sections (e.g. "The Creation Story", "የፍጥረት ታሪክ") that group verses into thematic blocks.

### Verse Object

```json
{
  "verse": 1,
  "text": {
    "am": "በመጀመሪያ እግዚአብሔር ሰማይንና ምድርን ፈጠረ።",
    "en": "In the beginning God created the heaven and the earth.",
    "gez": "በቀዳሚ ገብረ እግዚአብሔር ሰማየ ወምድረ ።"
  },
  "audio": {}
}
```

| Field | Type | Description |
|---|---|---|
| `verse` | int | Verse number (1-based) |
| `text` | object | Language code to text mapping |
| `audio` | object | Language code to audio path/URL mapping |
| `mergedWith` | object | *(optional)* Per-language merge info (see below) |

### Language Codes

| Code | Language | Coverage |
|---|---|---|
| `am` | Amharic | All 66 books |
| `en` | English (KJV) | All 66 books |
| `gez` | Ge'ez | Partial (null where unavailable) |

Additional languages will use standard ISO 639 codes (e.g. `ti` for Tigrinya, `om` for Oromo).

### Text Values

- A string when the verse text is available
- `null` when the text is not available from the source

Always check for null before displaying:

```python
text = verse["text"].get("gez")
if text:
    display(text)
```

### Audio

The `audio` object maps language codes to audio file paths or URLs. Currently empty — reserved for future audio content.

```json
"audio": {
  "am": "audio/am/01/001/001.mp3"
}
```

---

## Handling Merged Verses

Some verses on the source site are combined into a single cell (e.g. the Amharic translation merges verses 2-3 into one block, while English and Ge'ez have them separate). When this happens:

1. Both verse numbers exist as individual entries
2. Both carry the **same text** for the merged language
3. A `mergedWith` field indicates which verses share text, **per language**

### Example: Genesis 7:2-3

```json
{
  "verse": 2,
  "text": {
    "am": "ከንጹሕ እንስሳ ሁሉ ሰባት ሰባት...",
    "en": "Of every clean beast thou shalt take...",
    "gez": "ወእምነ እንስሳ ዘንጹሕ..."
  },
  "audio": {},
  "mergedWith": { "am": [2, 3] }
},
{
  "verse": 3,
  "text": {
    "am": "ከንጹሕ እንስሳ ሁሉ ሰባት ሰባት...",
    "en": "Of fowls also of the air by sevens...",
    "gez": "ወእምነ አዕዋፈ ሰማይ..."
  },
  "audio": {},
  "mergedWith": { "am": [2, 3] }
}
```

- Amharic text is identical for both verses (merged on the source)
- English and Ge'ez have distinct text per verse
- `mergedWith` only lists languages where merging occurred

### Deduplication Logic

To avoid displaying merged text twice:

```python
def should_display(verse, lang):
    merged = verse.get("mergedWith", {}).get(lang)
    if merged is None:
        return True  # Not merged — always display
    return verse["verse"] == merged[0]  # Only display on the first verse

# Usage
for verse in chapter["verses"]:
    if should_display(verse, "am"):
        print(f'{verse["verse"]}: {verse["text"]["am"]}')
```

```javascript
function shouldDisplay(verse, lang) {
  const merged = verse.mergedWith?.[lang];
  if (!merged) return true;
  return verse.verse === merged[0];
}
```

If you want to show a combined label like "2-3" instead of just "2":

```python
merged = verse.get("mergedWith", {}).get(lang)
if merged and verse["verse"] == merged[0]:
    label = f"{merged[0]}-{merged[-1]}"  # "2-3"
```

---

## Bundle File Structure

The `dist/` bundle files wrap the book array in a top-level object:

```json
{
  "testament": "complete",
  "total_books": 66,
  "books": [ ... ]
}
```

| `testament` value | Contents |
|---|---|
| `"complete"` | All 66 books |
| `"old"` | Books 1-39 (Old Testament) |
| `"new"` | Books 40-66 (New Testament) |

---

## Book Reference

### Old Testament (39 books)

| # | English | Amharic | Short (EN) | Short (AM) | Chapters |
|---|---|---|---|---|---|
| 1 | Genesis | ኦሪት ዘፍጥረት | Gen | ዘፍ | 50 |
| 2 | Exodus | ኦሪት ዘጸአት | Exod | ዘጸ | 40 |
| 3 | Leviticus | ኦሪት ዘሌዋውያን | Lev | ዘሌ | 27 |
| 4 | Numbers | ኦሪት ዘኍልቍ | Num | ዘኍ | 36 |
| 5 | Deuteronomy | ኦሪት ዘዳግም | Deut | ዘዳ | 34 |
| 6 | Joshua | መጽሐፈ ኢያሱ ወልደ ነዌ | Josh | ኢያ | 24 |
| 7 | Judges | መጽሐፈ መሳፍንት | Judg | መሳ | 21 |
| 8 | Ruth | መጽሐፈ ሩት | Ruth | ሩት | 4 |
| 9 | 1 Samuel | መጽሐፈ ሳሙኤል ቀዳማዊ | 1Sam | ፩ሳሙ | 31 |
| 10 | 2 Samuel | መጽሐፈ ሳሙኤል ካልዕ | 2Sam | ፪ሳሙ | 24 |
| 11 | 1 Kings | መጽሐፈ ነገሥት ቀዳማዊ | 1Kgs | ፩ነገ | 22 |
| 12 | 2 Kings | መጽሐፈ ነገሥት ካልዕ | 2Kgs | ፪ነገ | 25 |
| 13 | 1 Chronicles | መጽሐፈ ዜና መዋዕል ቀዳማዊ | 1Chr | ፩ዜና | 29 |
| 14 | 2 Chronicles | መጽሐፈ ዜና መዋዕል ካልዕ | 2Chr | ፪ዜና | 36 |
| 15 | Ezra | መጽሐፈ ዕዝራ | Ezra | ዕዝ | 10 |
| 16 | Nehemiah | መጽሐፈ ነህምያ | Neh | ነህ | 13 |
| 17 | Esther | መጽሐፈ አስቴር | Esth | አስ | 10 |
| 18 | Job | መጽሐፈ ኢዮብ | Job | ኢዮ | 42 |
| 19 | Psalms | መዝሙረ ዳዊት | Ps | መዝ | 150 |
| 20 | Proverbs | መጽሐፈ ምሳሌ | Prov | ምሳ | 31 |
| 21 | Ecclesiastes | መጽሐፈ መክብብ | Eccl | መክ | 12 |
| 22 | Song of Solomon | መኃልየ መኃልይ ዘሰሎሞን | Song | መኃ | 8 |
| 23 | Isaiah | ትንቢተ ኢሳይያስ | Isa | ኢሳ | 66 |
| 24 | Jeremiah | ትንቢተ ኤርምያስ | Jer | ኤር | 52 |
| 25 | Lamentations | ሰቆቃው ኤርምያስ | Lam | ሰቆ | 5 |
| 26 | Ezekiel | ትንቢተ ሕዝቅኤል | Ezek | ሕዝ | 48 |
| 27 | Daniel | ትንቢተ ዳንኤል | Dan | ዳን | 12 |
| 28 | Hosea | ትንቢተ ሆሴዕ | Hos | ሆሴ | 14 |
| 29 | Amos | ትንቢተ አሞጽ | Amos | አሞ | 9 |
| 30 | Micah | ትንቢተ ሚክያስ | Mic | ሚክ | 7 |
| 31 | Joel | ትንቢተ ኢዮኤል | Joel | ኢዮ | 3 |
| 32 | Obadiah | ትንቢተ አብድዩ | Obad | አብድ | 1 |
| 33 | Jonah | ትንቢተ ዮናስ | Jonah | ዮና | 4 |
| 34 | Nahum | ትንቢተ ናሆም | Nah | ናሆ | 3 |
| 35 | Habakkuk | ትንቢተ ዕንባቆም | Hab | ዕን | 3 |
| 36 | Zephaniah | ትንቢተ ሶፎንያስ | Zeph | ሶፎ | 3 |
| 37 | Haggai | ትንቢተ ሐጌ | Hag | ሐጌ | 2 |
| 38 | Zechariah | ትንቢተ ዘካርያስ | Zech | ዘካ | 14 |
| 39 | Malachi | ትንቢተ ሚልክያ | Mal | ሚል | 4 |

> Minor prophets follow the Ethiopian Bible canon order (differs from Protestant ordering).

### New Testament (27 books)

| # | English | Amharic | Short (EN) | Short (AM) | Chapters |
|---|---|---|---|---|---|
| 40 | Matthew | የማቴዎስ ወንጌል | Matt | ማቴ | 28 |
| 41 | Mark | የማርቆስ ወንጌል | Mark | ማር | 16 |
| 42 | Luke | የሉቃስ ወንጌል | Luke | ሉቃ | 24 |
| 43 | John | የዮሐንስ ወንጌል | John | ዮሐ | 21 |
| 44 | Acts | የሐዋርያት ሥራ | Acts | ሐዋ | 28 |
| 45 | Romans | ወደ ሮሜ ሰዎች | Rom | ሮሜ | 16 |
| 46 | 1 Corinthians | ፩ኛ ወደ ቆሮንቶስ ሰዎች | 1Cor | ፩ቆሮ | 16 |
| 47 | 2 Corinthians | ፪ኛ ወደ ቆሮንቶስ ሰዎች | 2Cor | ፪ቆሮ | 13 |
| 48 | Galatians | ወደ ገላትያ ሰዎች | Gal | ገላ | 6 |
| 49 | Ephesians | ወደ ኤፌሶን ሰዎች | Eph | ኤፌ | 6 |
| 50 | Philippians | ወደ ፊልጵስዩስ ሰዎች | Phil | ፊል | 4 |
| 51 | Colossians | ወደ ቆላስይስ ሰዎች | Col | ቆላ | 4 |
| 52 | 1 Thessalonians | ፩ኛ ወደ ተሰሎንቄ ሰዎች | 1Thess | ፩ተሰ | 5 |
| 53 | 2 Thessalonians | ፪ኛ ወደ ተሰሎንቄ ሰዎች | 2Thess | ፪ተሰ | 3 |
| 54 | 1 Timothy | ፩ኛ ወደ ጢሞቴዎስ | 1Tim | ፩ጢሞ | 6 |
| 55 | 2 Timothy | ፪ኛ ወደ ጢሞቴዎስ | 2Tim | ፪ጢሞ | 4 |
| 56 | Titus | ወደ ቲቶ | Titus | ቲቶ | 3 |
| 57 | Philemon | ወደ ፊልሞና | Phlm | ፊልሞ | 1 |
| 58 | Hebrews | ወደ ዕብራውያን | Heb | ዕብ | 13 |
| 59 | James | የያዕቆብ መልእክት | Jas | ያዕ | 5 |
| 60 | 1 Peter | ፩ኛ የጴጥሮስ መልእክት | 1Pet | ፩ጴጥ | 5 |
| 61 | 2 Peter | ፪ኛ የጴጥሮስ መልእክት | 2Pet | ፪ጴጥ | 3 |
| 62 | 1 John | ፩ኛ የዮሐንስ መልእክት | 1John | ፩ዮሐ | 5 |
| 63 | 2 John | ፪ኛ የዮሐንስ መልእክት | 2John | ፪ዮሐ | 1 |
| 64 | 3 John | ፫ኛ የዮሐንስ መልእክት | 3John | ፫ዮሐ | 1 |
| 65 | Jude | የይሁዳ መልእክት | Jude | ይሁ | 1 |
| 66 | Revelation | የዮሐንስ ራእይ | Rev | ራእ | 22 |

---

## Common Patterns

### Look up a specific verse (e.g. John 3:16)

```python
import json

with open("data/new-testament/43-john.json", encoding="utf-8") as f:
    john = json.load(f)

chapter = john["chapters"][2]   # 0-indexed, chapter 3
verse = chapter["verses"][15]   # 0-indexed, verse 16
print(verse["text"]["am"])
```

### Build a verse reference string

```python
book = genesis
ref_en = f"{book['book_short_name_en']} 1:1"   # "Gen 1:1"
ref_am = f"{book['book_short_name_am']} ፩:፩"    # "ዘፍ ፩:፩"
```

### Search across the entire Bible

```python
import json, os

def search_bible(query, lang="am"):
    results = []
    for testament in ["old-testament", "new-testament"]:
        dir_path = f"data/{testament}"
        for filename in sorted(os.listdir(dir_path)):
            with open(f"{dir_path}/{filename}", encoding="utf-8") as f:
                book = json.load(f)
            for ch in book["chapters"]:
                for v in ch["verses"]:
                    text = v["text"].get(lang, "")
                    if text and query in text:
                        results.append({
                            "ref": f"{book['book_short_name_en']} {ch['chapter']}:{v['verse']}",
                            "text": text
                        })
    return results

results = search_bible("እግዚአብሔር")
```

### Serve via a REST API (Flask example)

```python
from flask import Flask, jsonify
import json, os

app = Flask(__name__)

# Load all books at startup
books = {}
for testament in ["old-testament", "new-testament"]:
    for f in os.listdir(f"data/{testament}"):
        with open(f"data/{testament}/{f}", encoding="utf-8") as fh:
            book = json.load(fh)
            books[book["book_number"]] = book

@app.route("/api/books")
def list_books():
    return jsonify([{
        "number": b["book_number"],
        "name_am": b["book_name_am"],
        "name_en": b["book_name_en"],
        "short_en": b["book_short_name_en"],
        "short_am": b["book_short_name_am"],
        "chapters": len(b["chapters"])
    } for b in sorted(books.values(), key=lambda x: x["book_number"])])

@app.route("/api/book/<int:num>/<int:chapter>")
def get_chapter(num, chapter):
    book = books.get(num)
    if not book:
        return jsonify({"error": "Book not found"}), 404
    ch = next((c for c in book["chapters"] if c["chapter"] == chapter), None)
    if not ch:
        return jsonify({"error": "Chapter not found"}), 404
    return jsonify(ch)
```

---

## Extensibility

### Adding a new language

The `text` and `audio` objects are open dictionaries keyed by ISO 639 language codes. To add a new language (e.g. Tigrinya):

```python
for ch in book["chapters"]:
    for verse in ch["verses"]:
        verse["text"]["ti"] = tigrinya_lookup[ch["chapter"]][verse["verse"]]

book["sources"].append("tigrinya-source.com")
```

No schema changes needed — consumers that only read `am` or `en` are unaffected.

### Adding audio

```python
verse["audio"]["am"] = "https://cdn.example.com/audio/am/01/001/001.mp3"
```

### Adding sections

The `sections` array in each chapter is reserved for thematic groupings:

```json
{
  "chapter": 1,
  "sections": [
    {
      "title": "የፍጥረት ታሪክ",
      "title_en": "The Creation Story",
      "start_verse": 1,
      "end_verse": 31
    }
  ],
  "verses": [ ... ]
}
```

---

## Data Source

All text data is sourced from [ethiopicbible.com](https://www.ethiopicbible.com). The `sources` field on each book tracks provenance. English text is the King James Version (KJV).
