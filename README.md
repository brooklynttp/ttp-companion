# TTP Companion App

A web-based companion site for the Brooklyn KMC Teacher Training Program (TTP), currently covering the **"Transforming Through Patience"** series (November 2025 – January 2026), taught by **Kadam Kyle**.

Students can review weekly practices, read KK's teaching commentary, access the root texts (Meaningful to Behold and Guide to the Bodhisattva's Way of Life), listen to class recordings, and read full class transcripts.

---

## Directory Structure

```
brooklynttp/
│
├── index.html                          ← The app (single-page, all UI + logic)
├── README.md                           ← This file
│
├── classes/
│   ├── manifest.json                   ← List of class IDs to load
│   ├── 2025-11-02.json                 ← Weekly class file (one per class)
│   ├── 2025-11-09.json
│   ├── ...
│   ├── 2026-01-18.json
│   ├── class-TEMPLATE.json             ← Template for creating new class files
│   └── CLASS_FILE_GUIDE.md             ← How to assemble weekly class JSON files
│
├── texts/
│   ├── mtb-sections.json               ← Full text of Meaningful to Behold (paragraphed, sectioned)
│   ├── shantideva.json                 ← Full text of Guide to the Bodhisattva's Way of Life (by chapter/verse)
│   ├── prayers-for-meditation.json     ← Prayers for Meditation (by prayer section)
│   ├── Prayers_for_Meditation.txt      ← Human-readable prayer reference
│   └── mtb-reference-guide.md          ← Paragraph ID lookup for MTB sections
│
└── transcripts/
    ├── transcript-2025-11-02.json      ← Processed class transcript (one per class)
    ├── transcript-2025-11-09.json
    ├── ...
    └── TRANSCRIPTION_GUIDE.md          ← How to process raw audio transcripts
```

---

## How It All Connects

### Data Flow

```
Raw audio transcript (.txt)
        │
        ▼  [transcripts/TRANSCRIPTION_GUIDE.md]
Processed transcript JSON ──────────────────────► transcripts/transcript-YYYY-MM-DD.json
        │
        ▼  [classes/CLASS_FILE_GUIDE.md]
Weekly class JSON ──────────────────────────────► classes/YYYY-MM-DD.json
                                                         │
                                                         ▼
                                                  classes/manifest.json (add the new ID)
                                                         │
                                                         ▼
                                                    index.html (loads everything at runtime)
```

### What index.html Loads at Runtime

When a user opens the app, `index.html` does the following:

1. **Fetches `classes/manifest.json`** — gets the ordered list of class IDs
2. **Fetches each `classes/{id}.json`** in parallel — loads all weekly class data
3. **Renders the most recent class** — shows practice, commentary, root text sections, recording link, transcript
4. **Lazy-loads text content on demand:**
   - When user opens "Meaningful to Behold" → fetches `texts/mtb-sections.json`, pulls paragraphs by ID range
   - When user opens "Guide to the Bodhisattva's Way of Life" → fetches `texts/shantideva.json`, pulls verses by chapter/number
   - When user opens "Class Transcript" → fetches `transcripts/transcript-{id}.json`

The text files (`mtb-sections.json`, `shantideva.json`) are cached after first load so subsequent classes don't re-fetch them.

---

## Source Text Files (texts/)

These are the authoritative reference texts. They serve two purposes: the app loads them at runtime to display root text content, and the transcription workflow uses them for verbatim replacement of prayers and Shantideva verses.

### mtb-sections.json

The full text of **Meaningful to Behold** by Geshe Kelsang Gyatso, organized into sections with globally numbered paragraphs.

```json
{
  "title": "Meaningful to Behold",
  "sections": {
    "section-slug": {
      "title": "Section Title",
      "paragraphs": [
        { "id": 173, "text": "Paragraph content..." },
        { "id": 174, "text": "Next paragraph..." }
      ]
    }
  }
}
```

Paragraph IDs are globally unique across the entire book (1 through ~1985). Class files reference MTB content by paragraph range — the app pulls the matching paragraphs at display time. See `texts/mtb-reference-guide.md` for a complete lookup of section slugs and paragraph ID ranges.

### shantideva.json

The full text of **Guide to the Bodhisattva's Way of Life** by Shantideva, organized by chapter and verse number.

```json
{
  "chapters": {
    "1": {
      "title": "An Explanation of the Benefits of Bodhichitta",
      "verses": {
        "1": "Verse text with\nnewlines between lines...",
        "2": "Next verse..."
      }
    }
  }
}
```

Class files reference Shantideva content by chapter and verse range — the app pulls the matching verses at display time.

### prayers-for-meditation.json

The full text of **Prayers for Meditation** (Brief Preparatory Prayers for Meditation), from Appendix I of Heart Jewel. Organized by prayer section with slug keys.

```json
{
  "prayers": {
    "liberating-prayer": {
      "title": "Liberating Prayer — Praise to Buddha Shakyamuni",
      "text": "O Blessed One, Shakyamuni Buddha,\nPrecious treasury of compassion..."
    },
    "stages-of-the-path": {
      "title": "Prayer of the Stages of the Path",
      "verses": {
        "1": "The path begins with strong reliance...",
        "2": "This human life with all its freedoms..."
      }
    }
  },
  "session_order": {
    "session1_full": ["liberating-prayer", "going-for-refuge", ...],
    "session2_abbreviated": ["liberating-prayer", "bodhisattva-vows", ...],
    "closing": ["dedication", "prayers-for-the-virtuous-tradition", ...]
  }
}
```

This file is primarily used as the authoritative source for transcript processing — prayer sections in the raw transcript should be replaced verbatim with the text from this file. The `session_order` maps document which prayers are used in Session 1 (full practice), Session 2 (abbreviated), and closing.

**Note:** The app does not currently load this file at runtime (prayers are rendered in the transcript JSON, not as a separate expandable section). This may change in the future if a standalone "Prayers" section is added to the UI.

---

## Weekly Class Files (classes/)

Each class gets its own JSON file in `classes/`. The file is named by date: `YYYY-MM-DD.json`.

### manifest.json

A simple list of class IDs in chronological order. The app uses this to know which class files to fetch.

```json
{
  "classes": [
    "2025-11-02",
    "2025-11-09",
    "2025-11-16",
    "2025-11-30",
    "2025-12-07",
    "2025-12-14",
    "2026-01-04",
    "2026-01-11",
    "2026-01-18"
  ]
}
```

When adding a new class, add its ID to this array.

### Class File Schema

See `classes/class-TEMPLATE.json` for the full template. Key fields:

```json
{
  "id": "2026-01-18",
  "date": "Jan 18",
  "year": "2026",
  "topic": "Love in Action",
  "recording": "https://drive.google.com/file/d/.../view?usp=share_link",

  "practice": {
    "title": "Do Everything with Love",
    "steps": ["Step 1...", "Step 2..."],
    "quote": "Optional KK quote"
  },

  "commentary": [
    {
      "title": "Teaching Point Title",
      "content": ["Paragraph 1...", "Paragraph 2..."],
      "quote": "Optional KK quote"
    }
  ],

  "mtb": {
    "startParagraph": 184,
    "endParagraph": 188
  },

  "shantideva": {
    "chapter": 1,
    "startVerse": 15,
    "endVerse": 25
  },

  "preview": {
    "startParagraph": 189,
    "endParagraph": 217
  }
}
```

**How root text references work:** The `mtb`, `shantideva`, and `preview` fields don't contain the actual text — they contain *pointers* (paragraph ranges, chapter/verse ranges) into the source text files. When the user clicks to expand these sections, the app fetches the source JSON and pulls the matching content. This means:

- You never duplicate text across class files
- If a typo is fixed in `mtb-sections.json`, it's fixed everywhere
- Class files stay small and focused on the curated content (practice, commentary, data shares)

For the full guide to assembling class files, see `classes/CLASS_FILE_GUIDE.md`.

---

## Transcript Files (transcripts/)

Processed class transcripts live in `transcripts/` as `transcript-YYYY-MM-DD.json`. These are structured into sessions with named sections:

```json
{
  "session1": {
    "classIntro": [...],
    "chantedPrayers": { "time": "25:17 - 41:12", "text": "..." },
    "guidedMeditation": [...],
    "postMeditation": [...],
    "shantidevaReading": [...],
    "postReading": [...],
    "practiceReports": [...],
    "mainTeaching": [...]
  },
  "session2": {
    "chantedPrayers": { "time": "2:30:30 - 2:34:01", "text": "..." },
    "guidedMeditation": [...],
    "teaching": [...],
    "conclusion": [...]
  }
}
```

Teaching segments are arrays of speaker objects:

```json
{ "speaker": "Kadam Kyle", "time": "49:22", "text": "Teaching content..." }
```

Prayer sections use the `text` field with the full verbatim prayer text (replaced from `prayers-for-meditation.json` during transcript processing — never the raw transcription software output).

For the full guide to processing raw transcripts, see `transcripts/TRANSCRIPTION_GUIDE.md`.

---

## Workflow: Adding a New Class

1. **Process the raw transcript** → Follow `transcripts/TRANSCRIPTION_GUIDE.md` → Save as `transcripts/transcript-YYYY-MM-DD.json`
2. **Assemble the class file** → Follow `classes/CLASS_FILE_GUIDE.md` → Save as `classes/YYYY-MM-DD.json`
3. **Update the manifest** → Add the new class ID to `classes/manifest.json`
4. **Add the recording link** → Update the `recording` field in the class JSON once the Google Drive link is available
5. **Test** → Open `index.html`, verify the new class appears in the chip selector and all sections render correctly

---

## Documentation

| File | Purpose |
|------|---------|
| `README.md` | This file — project overview and architecture |
| `transcripts/TRANSCRIPTION_GUIDE.md` | How to process raw `.txt` transcripts into clean transcript JSON |
| `classes/CLASS_FILE_GUIDE.md` | How to assemble weekly class companion JSON from a clean transcript |
| `texts/mtb-reference-guide.md` | Paragraph ID and section slug lookup for Meaningful to Behold |
| `classes/class-TEMPLATE.json` | Template for creating new weekly class files |

---

## Series: Transforming Through Patience

| # | Date | Class ID | Topic |
|---|------|----------|-------|
| 1 | Nov 2, 2025 | `2025-11-02` | Introduction |
| 2 | Nov 9, 2025 | `2025-11-09` | The Traveler |
| 3 | Nov 16, 2025 | `2025-11-16` | Precious Human Life |
| 4 | Nov 30, 2025 | `2025-11-30` | Bodhichitta |
| 5 | Dec 7, 2025 | `2025-12-07` | Benefits of Bodhichitta |
| 6 | Dec 14, 2025 | `2025-12-14` | Aspiring & Engaging |
| 7 | Jan 4, 2026 | `2026-01-04` | Equanimity |
| 8 | Jan 11, 2026 | `2026-01-11` | Equanimity Practice |
| 9 | Jan 18, 2026 | `2026-01-18` | Love in Action |

---

*Last updated: February 2, 2026*
