# TTP Class File Assembly Guide

A comprehensive reference for assembling the weekly class companion page for the TTP Companion App — practices, KK commentary, root text references, and recording links.

This guide assumes you already have a **clean transcript** (see **[TRANSCRIPTION_GUIDE.md](../transcripts/TRANSCRIPTION_GUIDE.md)** for that process). The clean transcript is the primary input; the source texts and previous class entries are the secondary inputs. See **[README.md](../README.md)** for the overall project architecture and directory structure.

---

## Table of Contents

1. [Overview](#overview)
2. [JSON Schema](#json-schema)
3. [Step-by-Step Assembly](#step-by-step-assembly)
   - [1. Metadata](#1-metadata)
   - [2. Practice](#2-practice)
   - [3. Commentary](#3-commentary)
   - [4. Root Text: Meaningful to Behold](#4-root-text-meaningful-to-behold)
   - [5. Root Text: Shantideva](#5-root-text-shantideva)
   - [6. Next Week's Reading (Preview)](#6-next-weeks-reading-preview)
   - [7. Recording Link](#7-recording-link)
4. [Identifying Root Text Sections](#identifying-root-text-sections)
5. [Assembly Checklist](#assembly-checklist)

---

## Overview

Each class in the TTP Companion App is a standalone JSON file in the `classes/` directory (e.g., `classes/2026-01-18.json`), listed in `classes/manifest.json`. The class entry powers everything the student sees on the companion page: the practice assignment, KK's teaching summary, root text references, student practice shares, and links to the recording.

The assembly process draws from multiple sources:

| Source | What it provides |
|--------|-----------------|
| Clean transcript | Practices, commentary content, data shares, KK quotes |
| `../texts/mtb-sections.json` | MTB section slugs, paragraph IDs, representative passages |
| `../texts/shantideva.json` | Shantideva verse text by chapter/verse number |
| `../texts/prayers-for-meditation.json` | Authoritative prayer texts (Liberating Prayer, refuge, Lamrim, dedications) |
| `../texts/mtb-reference-guide.md` | Quick lookup of MTB section slugs and paragraph ID ranges |
| Previous class entry | Verse progression continuity, formatting conventions |
| Audio recording host | Google Drive recording URL |

---

## JSON Schema

The full schema for a class entry (see also `class-TEMPLATE.json` (in this folder)):

```json
{
  "id": "2026-01-18",
  "date": "Jan 18",
  "year": "2026",
  "topic": "Love in Action",
  "recording": "https://drive.google.com/file/d/YOUR_FILE_ID/view?usp=share_link",

  "practice": {
    "title": "Do Everything with Love",
    "steps": [
      "Step 1 description",
      "Step 2 description",
      "Step 3 description"
    ],
    "quote": "Optional KK quote about the practice"
  },

  "commentary": [
    {
      "title": "First Teaching Point",
      "content": [
        "First paragraph of commentary.",
        "Second paragraph of commentary."
      ],
      "quote": "Optional KK quote"
    },
    {
      "title": "Second Teaching Point",
      "content": [
        "Commentary paragraph."
      ]
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

### Key Schema Notes

- **`id`** must be `YYYY-MM-DD` format for app sorting and selection logic
- **`date`** is the short display format (e.g., "Jan 18"), **`year`** is separate
- **`practice.steps`** is a flat array of strings — each step is one practice instruction
- **`practice.quote`** is optional — only include when KK gives a specific, quotable emphasis
- **`commentary`** is an array of teaching point objects, each with a `title`, `content` (array of paragraphs), and optional `quote`
- **`mtb`** references paragraphs by global ID range — the app fetches `../texts/mtb-sections.json` and pulls matching paragraphs at display time. Use `../texts/mtb-reference-guide.md` to look up paragraph IDs.
- **`shantideva`** references verses by chapter and verse range — the app fetches `../texts/shantideva.json` and pulls matching verses. Set to `null` for classes that don't cover specific verses.
- **`preview`** is the "Next Week's Reading" section — same paragraph-range system as `mtb`. Set to `null` if no preview is needed.
- **`recording`** is a Google Drive share link. Set to `null` until the recording is uploaded.

---

## Step-by-Step Assembly

### 1. Metadata

```json
"id": "2026-01-18",
"date": "Jan 18",
"year": "2026",
"topic": "Love in Action",
```

**Where to find it:** The date comes from the filename (`TTP_011826.txt` = January 18, 2026). `id` is the `YYYY-MM-DD` format. `date` is the short display form ("Jan 18"). `year` is separate. The `topic` is a judgment call — listen to KK's framing of the class and pick the dominant theme in 2-4 words.

---

### 2. Practice

This is one of the most important sections. It captures what students should be working on between classes.

```json
"practice": {
  "title": "Do Everything with Love",
  "steps": [
    "Make decisions from a mind of love — stop trying to figure things out from self-grasping.",
    "Do all actions with love, or don't do them at all",
    "Identify your 'target' — overthinking? Laziness? Anxiety-driven motivation?"
  ],
  "quote": "What would it be like to 100% stop trying to figure out what's going on, and instead put all energy into getting a mind of love going?"
}
```

#### Where the practice lives in the transcript

The final 10-15 minutes of class. After the **paired discussion** ends (see Transcription Guide for exclusion rules) and the whole group reconvenes, KK typically asks something like "what do we want to get up to this week?" or "so what do we want to work on?" Students share their commitments, and KK affirms, refines, or synthesizes. This conversation — the **post-discussion resolution** — is where the group-agreed practice comes from.

Note: Sometimes KK has already strongly suggested a practice *during* the teaching (before the paired discussion), and the post-discussion resolution simply confirms it. In that case, draw the practice content from KK's teaching, not just the brief reconvene exchange.

Typical trigger phrases:
- "what do we want to get up to over the next week?"
- "what should we work on?"
- "let's see what we can work on for this class"
- "anything you'd like to declare or swear?"

#### Building the steps array

`practice.steps` is a flat array of strings. List the most important, group-agreed practices first, followed by supplementary contemplations KK suggested during class. Each step should be a clear, actionable instruction.

- **`title`** — A short, evocative name for the practice (2-6 words)
- **`steps`** — Concrete instructions. Aim for 3-6 steps.
- **`quote`** — Optional. Include when KK gives a specific, quotable emphasis about the practice. Should be a direct quote, not a paraphrase.

---

### 3. Commentary

```json
"commentary": [
  {
    "title": "Being Guided by Love",
    "content": [
      "How often do we make a decision with an overt, intense love guiding us?",
      "It's like being possessed by something forcing you to act."
    ],
    "quote": "It's almost a sense that you're being possessed by something that's forcing you to do something."
  },
  {
    "title": "Love Solves Your Problems",
    "content": [
      "If you're an overthinker: Love makes the decision, you act wholeheartedly.",
      "If you're lazy: Love provides almost limitless energy from a non-contaminated source."
    ]
  }
]
```

`commentary` is an array of teaching point objects, each with:

- **`title`** — A short heading that captures the teaching theme
- **`content`** — Array of paragraphs. Each paragraph should capture a distinct idea or argument.
- **`quote`** — Optional. A direct KK quote that encapsulates the point.

**Where to find it:** KK's teaching segments across both sessions. Each teaching point should capture a distinct theme or arc — not just "we discussed X" but the actual insight.

**Tips:**
- Don't just list topics — capture the *insight*. Not "We discussed equanimity" but "Equanimity means abandoning the distinctions of friend, enemy, and stranger."
- Include KK's distinctive examples and metaphors when they're particularly illuminating.
- Student shares that contain strong Dharma insights can be their own commentary entry (e.g., "Student Share: The Traveler Mindset with Bodhicitta").
- If KK reads from Meaningful to Behold and then gives his own commentary, the content should reflect KK's commentary (the "so what"), not just summarize the text passage.
- Aim for 4-8 commentary entries per class.

---

### 4. Root Text: Meaningful to Behold

```json
"mtb": {
  "startParagraph": 184,
  "endParagraph": 188
}
```

This tells the app which paragraphs from `../texts/mtb-sections.json` to display. The app fetches the JSON and pulls all paragraphs with IDs in the given range, automatically inserting section headers when the content crosses section boundaries.

**How to find the paragraph range:**
1. Listen for KK saying page numbers or section names
2. Search `../texts/mtb-reference-guide.md` for the section — it lists every section slug and paragraph ID range with preview text
3. Identify the starting and ending paragraphs covered in class
4. Check that the progression is continuous from the previous class (no gaps or overlaps)

Set to `null` if the class doesn't cover a specific MTB section (rare — nearly every class covers some MTB content).

---

### 5. Root Text: Shantideva

```json
"shantideva": {
  "chapter": 1,
  "startVerse": 15,
  "endVerse": 25
}
```

This tells the app which verses from `../texts/shantideva.json` to display. The app fetches the JSON and pulls verses by chapter and number.

**How to find the verse range:**
- KK or a student reads Shantideva verses aloud near the beginning of class (often Session 1, after prayers)
- Look for verse numbers like "(15)" or "verse 22"
- Check that the progression is continuous from the previous class

Set to `null` for classes that don't cover specific Shantideva verses.

---

### 6. Next Week's Reading (Preview)

```json
"preview": {
  "startParagraph": 189,
  "endParagraph": 217
}
```

This is the "Next Week's Reading" section in the app — it shows the MTB paragraphs students should read before the next class. Same paragraph-range system as `mtb`.

**Where to find it:** KK sometimes assigns reading toward the end of class. Look for phrases like "for next week, read pages..." or "try to get through the next section." If KK doesn't explicitly assign reading, you can infer it — the next class will likely pick up where the `mtb` range of this class left off.

Set to `null` if no preview is needed.

---

### 7. Recording Link

```json
"recording": "https://drive.google.com/file/d/1V_CCJv.../view?usp=share_link"
```

This is a Google Drive share link to the class recording. Set to `null` until the recording is uploaded and a link is available.

---

## Identifying Root Text Sections

### The Relationship Between the Two Texts

Meaningful to Behold (MTB) is a chapter-by-chapter, verse-by-verse commentary on the Guide to the Bodhisattva's Way of Life (the Guide). When KK teaches from MTB, the MTB text itself quotes or references specific Shantideva verses. So tracking what was covered means identifying:

1. **Which MTB paragraphs were covered** — the primary reference (paragraph ID range)
2. **Which Shantideva verses were read or discussed** — the secondary reference (chapter/verse range)

### How to Identify What Was Covered

Look for these indicators in the teaching segments of the clean transcript:

- **Explicit page references:** KK often says things like "page 29" or "turn to page..." referring to MTB
- **Verse numbers:** KK or the text will reference verse numbers like "(15)" or "verse 22" from Shantideva
- **Reading aloud from MTB:** Extended passages of commentary language — these will sound more formal and literary than KK's own commentary
- **Reading aloud from the Guide:** Verse-form text that KK reads, sometimes preceded by "Shantideva says..." or the verse number
- **Where the previous class left off:** Track the progression. If the last class ended at paragraph 183, this class likely picks up around 184.

### Cross-Referencing with Source Files

Use the project source files to verify:

1. Search `../texts/mtb-reference-guide.md` for the section slug and paragraph ID range
2. Search `../texts/mtb-sections.json` for distinctive phrases KK reads to pinpoint the exact paragraph
3. Search `../texts/shantideva.json` for verse content to confirm verse numbers

```bash
# Search MTB reference guide for a section
grep -n "equanimity\|Equanimity" ../texts/mtb-reference-guide.md

# Search the Shantideva JSON for a specific verse
grep -n '"15"' ../texts/shantideva.json
```

### Verse Progression for This Series

The "Transforming Through Patience" series covers:

- **Chapter 1** of the Guide (Benefits of Bodhichitta) — early classes
- Moving into **equanimity** teachings (based on MTB's discussion of recognizing bodhichitta)
- Track which specific verses and MTB paragraph ranges each class covers to maintain continuity

Always check the previous class entry's `mtb` and `shantideva` ranges to ensure there are no gaps or overlaps in the progression.

---

## Assembly Checklist

Before saving a new class file to `classes/` and adding its ID to `manifest.json`:

### Data Completeness

- [ ] All metadata fields populated (`id`, `date`, `year`, `topic`)
- [ ] `practice` has `title` and `steps` (array of strings)
- [ ] `practice.quote` included if KK gave a quotable line about the practice
- [ ] `commentary` array has 4-8 teaching point objects, each with `title` and `content`
- [ ] Commentary entries capture insights, not just topic labels
- [ ] `mtb` paragraph range set (continuous from previous class)
- [ ] `shantideva` chapter/verse range set, or `null`
- [ ] `preview` paragraph range set for next week's reading, or `null`
- [ ] `recording` Google Drive link set, or `null` as placeholder

### Continuity & Consistency

- [ ] Root text progression is continuous from the previous class — no gaps or duplications
- [ ] `mtb.startParagraph` of this class follows `mtb.endParagraph` of the previous class
- [ ] `shantideva.startVerse` of this class follows `shantideva.endVerse` of the previous class
- [ ] JSON is valid (check with a linter)
- [ ] New class ID added to `classes/manifest.json` in chronological order

---

*Last updated: February 2, 2026*
