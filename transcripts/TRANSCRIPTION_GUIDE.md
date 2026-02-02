# TTP Transcription Guide

A comprehensive reference for processing raw speaker-diarized audio transcripts from the Brooklyn Tharpa Tradition Program (TTP) into clean, structured transcript data.

For assembling the weekly class companion page (practices, commentary, root text, recording links, etc.), see the separate **[CLASS_FILE_GUIDE.md](../classes/CLASS_FILE_GUIDE.md)**.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Source Materials](#source-materials)
3. [Raw Transcript Format](#raw-transcript-format)
4. [Speaker Identification](#speaker-identification)
5. [Class Structure & Two-Session Flow](#class-structure--two-session-flow)
6. [Prayers](#prayers)
7. [Jibber-Jabber: What to Exclude](#jibber-jabber-what-to-exclude)
8. [Special Character & Encoding Cleanup](#special-character--encoding-cleanup)
9. [Transcript Quality Control Checklist](#transcript-quality-control-checklist)
10. [Technical Tools & Commands](#technical-tools--commands)

---

## Project Overview

The TTP (Tharpa Tradition Program) is a Buddhist study group at KMC Brooklyn. The current series, **"Transforming Through Patience,"** runs from November 2025 through January 2026. The teacher is **Kadam Kyle (KK)**.

The class studies two primary texts:

- **Meaningful to Behold** by Geshe Kelsang Gyatso — the primary commentary being taught
- **Guide to the Bodhisattva's Way of Life** by Shantideva — the root verse text that Meaningful to Behold comments on

**This guide covers the first stage of the workflow:** taking a raw speaker-diarized `.txt` file and producing a clean transcript — speakers identified, jibber-jabber removed, encoding fixed, class structure mapped. The output of this stage feeds into the class file assembly process (documented separately in `../classes/CLASS_FILE_GUIDE.md`).

---

## Source Materials

The project repo should contain:

| File | Description |
|------|-------------|
| `../texts/shantideva.json` | Shantideva's root verses, organized by chapter and verse number |
| `../texts/mtb-sections.json` | Geshe Kelsang's commentary, organized by section with numbered paragraphs |
| `../texts/prayers-for-meditation.json` | Authoritative prayer texts used in TTP classes (Liberating Prayer, refuge, Lamrim prayer, dedications, etc.) |
| `TTP_MMDDYY.txt` | Raw speaker-diarized transcript files (one per class) |
| `index.html` | The companion app — loads class files and source texts at runtime |

See the project `../README.md` for the full directory structure and how all the files connect.

---

## Raw Transcript Format

Transcripts come from speaker-diarized audio processing. Each speaker segment looks like this:

```
[Speaker 3] (0:22 - 0:55)
I'm annoyed at myself. I had so many packages to bring today that I left my book bag on.
```

The format is:

```
[Speaker N] (START_TIME - END_TIME)
Content text, which may span multiple paragraphs.

Additional paragraphs are separated by blank lines.

[Speaker M] (START_TIME - END_TIME)
Next speaker's content...
```

Key characteristics of the raw format:

- Speaker numbers are **arbitrary and inconsistent across transcripts.** Speaker 1 in one transcript is not the same as Speaker 1 in another. The diarization software assigns numbers fresh each time.
- Timestamps are in `H:MM:SS` or `M:SS` format.
- A single real person may be assigned **multiple speaker numbers** within the same transcript. The software sometimes splits one person into two or more speaker IDs, especially if there's a pause or change in audio quality.
- Chanted prayers (which are spoken in unison by the group) typically get assigned to a single speaker number, but the content is communal.
- The transcript captures **everything** the microphone picks up — pre-class chatter, break-time socializing, post-class goodbyes, and all sorts of ambient conversation. Much of this needs to be filtered out.

---

## Speaker Identification

### The Teacher: Kadam Kyle (KK)

Kadam Kyle is the primary teacher. Identifying KK's speaker number is the **first and most critical step** for each transcript. KK's segments will be:

- **The longest segments** in the transcript (often 5-15+ minutes of continuous teaching)
- **The most substantive content** — Dharma teachings, text readings, meditation instructions, class logistics
- Present in **both sessions** of the class

**How to identify KK:**

1. Look at the speaker number that appears most frequently with long durations.
2. Check for teaching-style content: reading from Meaningful to Behold, giving meditation instructions, referencing "Geshe-la" or "Shantideva," explaining Buddhist concepts.
3. Look for class management language: "we're going to begin," "let's take a look at the book," "we'll take a break," "let's dedicate here."

**Important:** KK's speaker number **changes from transcript to transcript.** In one file KK might be Speaker 1, in another Speaker 5. Never assume.

### KK Split Across Multiple Speaker Numbers

This is one of the most common issues you'll encounter. The diarization software frequently assigns KK **two or more different speaker numbers** within the same transcript. This happens because:

- KK's voice sounds different when teaching (projecting, formal tone) vs. reading from the text vs. having a casual exchange with a student
- Audio quality shifts (moving closer to or farther from the mic, turning to address different parts of the room)
- Pauses between segments cause the software to "forget" the previous speaker identity
- The software may interpret KK reading a quote in a different cadence as a new speaker

**How to detect this:** After your initial speaker count, if you see two or three speaker numbers that all have high segment counts and long durations, and both contain teaching-style content, they're probably both KK. Verify by checking:

1. **Content continuity** — Does Speaker 1 start explaining a concept and then Speaker 5 continues the exact same thought? That's one person.
2. **Timestamp adjacency** — If Speaker 1 ends at 45:30 and Speaker 5 starts at 45:31 with no natural speaker change, they're the same person.
3. **Teaching voice vs. reading voice** — KK reading aloud from Meaningful to Behold may get a different number than KK giving his own commentary on what he just read.
4. **No student-style content** — If both speaker numbers only contain teaching, commentary, and class management language (never questions or practice shares), they're almost certainly both KK.

**How to handle it:** When building the transcript, combine all KK speaker numbers into a single speaker attribution. The content should flow as one continuous teaching, ordered by timestamp. Don't create separate sections for "KK as Speaker 1" and "KK as Speaker 5" — merge them into the natural chronological flow of the class.

```bash
# Example: Check if two speakers are both KK by looking at their content
# If both contain teaching-style content, they're likely the same person
grep -A3 "\[Speaker 1\]" TTP_MMDDYY.txt | head -20
grep -A3 "\[Speaker 5\]" TTP_MMDDYY.txt | head -20
```

### The Prayer Chanter

Chanted prayers are led by one person but recited by the whole group. In the transcript, this typically shows up as a single speaker number with a long block of prayer text. The prayer chanter:

- Usually gets a **different speaker number from KK** (the diarization software hears the group voice differently than KK's solo teaching voice)
- Has been observed as Speaker 2, Speaker 3, and other numbers across different transcripts
- Can be identified by the distinctive prayer content (see [Prayers](#prayers) section)

### Students

Students appear in practice-sharing / "data share" segments and occasional questions. Some students appear by name in discussions. The transcript may assign several speaker numbers to students — some of whom are actually the same person split by the software.

For the output, student names are identified where possible (from context — KK sometimes addresses them by name, or they introduce themselves). Where names aren't clear, use "Student" or "Class member."

---

## Class Structure & Two-Session Flow

### Standard Two-Session Format

A typical TTP class has **two sessions** separated by a dinner break. This is the norm, not the exception. Here is the full flow:

```
╔══════════════════════════════════════════╗
║          PRE-CLASS (exclude)             ║
║  Casual chatter as people arrive         ║
╠══════════════════════════════════════════╣
║                                          ║
║  SESSION 1 (approx. 4:30 PM - 5:30 PM)  ║
║                                          ║
║  1. [Optional] Pre-Prayers Introduction  ║
║     KK may share a brief reading,        ║
║     thought, or framing before prayers.  ║
║     NOT always present.                  ║
║                                          ║
║  2. Opening Prayers (chanted)            ║
║     - Heart Jewel / Prayers for          ║
║       Meditation                         ║
║     - Includes refuge, mandala offering, ║
║       Lamrim prayers                     ║
║     - Led by prayer chanter, joined      ║
║       by group                           ║
║                                          ║
║  3. Guided Meditation                    ║
║     KK leads a meditation related to     ║
║     the current topic. Ends with:        ║
║     "in our own time, we can gradually   ║
║      arise from our practice"            ║
║                                          ║
║  4. Session 1 Teaching                   ║
║     Could be any combination of:         ║
║     - Text reading & commentary          ║
║     - Practice sharing / Data shares     ║
║     - Discussion of the week's practice  ║
║     - Topical teaching                   ║
║     The content varies by class.         ║
║                                          ║
║  5. Session 1 Dedication                 ║
║     Brief dedication of merit:           ║
║     "through the virtues we've collected ║
║      through practicing, meditating,     ║
║      discussing, may all living beings   ║
║      find lasting peace of mind,         ║
║      enlightenment"                      ║
║                                          ║
╠══════════════════════════════════════════╣
║        DINNER BREAK (exclude)            ║
║  ~15-30 min of socializing, chatter      ║
╠══════════════════════════════════════════╣
║                                          ║
║  SESSION 2 (approx. 6:00 PM - 7:30 PM)  ║
║                                          ║
║  6. Session 2 Opening Prayers            ║
║     Shorter than Session 1. Usually:     ║
║     - Prayers for Meditation             ║
║     - Refuge & bodhisattva vows retaken  ║
║     Often does NOT include the full      ║
║     mandala offering or Lamrim prayers   ║
║                                          ║
║  7. Brief Meditation                     ║
║     Shorter meditation connecting to     ║
║     the vows or topic                    ║
║                                          ║
║  8. Session 2 Teaching                   ║
║     Usually the main text study:         ║
║     - Reading from Meaningful to Behold  ║
║     - Discussion of Shantideva's verses  ║
║     - KK's commentary and examples       ║
║     But could also be data shares or     ║
║     continued discussion from Session 1  ║
║                                          ║
║  9. Agreed Practice for the Week         ║
║     KK and group discuss what to         ║
║     practice between now and next class  ║
║                                          ║
║  10. Closing Dedication                  ║
║      Brief dedication, sometimes with    ║
║      the "nine-line prayer" or similar   ║
║                                          ║
╚══════════════════════════════════════════╝
║         POST-CLASS (exclude)             ║
║  Goodbyes, casual chat                   ║
╚══════════════════════════════════════════╝
```

### Variations to Watch For

- **Pre-Prayers Introduction:** Sometimes KK begins with a short reading or thought *before* the opening prayers. This is class content and should be included. Not every class has this — check whether KK is speaking substantively before the prayers start. Some transcripts begin directly with prayers.
- **Single-Session Classes:** Occasionally a class may only have one session. This is the exception. You can tell because there won't be a mid-class dedication, break chatter, or second set of opening prayers.
- **Session Content Swaps:** The teaching and data shares don't always fall in the same session. Sometimes Session 1 is teaching and Session 2 is data shares; sometimes it's reversed. Follow the actual content, not assumptions about order.
- **Announcements:** KK sometimes makes announcements about upcoming courses or events, usually near the end of Session 1 or beginning of Session 2. These are generally included in the transcript as they're part of the class.

### Key Transition Phrases

These phrases help you identify section boundaries:

| Phrase | Signals |
|--------|---------|
| "we're going to begin" / "let's begin" | Class starting (after optional pre-prayers intro) |
| "O Blessed One Shakyamuni Buddha..." | Opening prayers beginning |
| "in our own time, we can gradually arise from our practice" | End of meditation |
| "Welcome back everyone" / "Okay, welcome back" | Post-meditation transition to teaching |
| "let's take a look at the book" / "let's open to page..." | Text study beginning |
| "let's dedicate here" / "we can think through the virtues..." | End of a session |
| "we'll come back at [time]" / "we'll take a break" | Break between sessions |
| "O Blessed One Shakyamuni Buddha..." (second occurrence) | Session 2 prayers beginning |
| "what should we work on this week" / "agreed practice" | Practice discussion (usually near end) |
| "we can make any personal prayers or dedications" | Final closing — class is over |

---

## Prayers

### Where Prayer Texts Live

The authoritative prayer texts are in **`../texts/prayers-for-meditation.json`** — this is the clean, published version of all prayers used in TTP classes. (A human-readable `.txt` version, `../texts/Prayers_for_Meditation.txt`, is also available for reference.)

The raw transcripts also contain the prayers as captured by the audio diarization, but these transcribed versions are **not reliable.** The software's attempt at transcribing chanted group recitation produces garbled words, mishearings, run-together lines, and mangled Dharma terms. The transcribed prayers should **never** be used as-is.

### Critical Rule: Verbatim Replacement from Source Texts

**Prayers and Shantideva verses must be replaced with the exact published text, not cleaned-up versions of the transcript.**

This applies to two categories:

1. **Prayers** — Replace transcribed prayer content with the exact text from `../texts/prayers-for-meditation.json`. The diarization software cannot accurately capture group chanting, so every prayer section should be swapped wholesale with the published version.

2. **Shantideva verses** — When KK reads Shantideva verses aloud, replace the transcribed version with the exact text from `../texts/shantideva.json`. Match by verse number (format: `(15)`) and verify the content.

**What about Meaningful to Behold?** Leave MTB passages as the transcript has them. KK frequently stops mid-paragraph to insert his own commentary, students ask questions mid-reading, and the boundary between "KK reading the text" and "KK riffing on the text" is often blurry. Attempting verbatim replacement would require surgically splicing published text around these interjections, which isn't worth the effort or risk. Clean up encoding issues and obvious Dharma term misspellings in MTB readings, but don't try to replace them wholesale. (This may be revisited later.)

**Why prayers and Shantideva are different:** Prayers are chanted as complete, uninterrupted blocks — there's a clear start and end with no commentary mixed in. Shantideva verses are similarly discrete units that KK reads as standalone passages. Both have clean boundaries that make wholesale replacement straightforward and reliable.

### Opening Prayers (Session 1) — Full Practice

The Session 1 opening is a longer practice that typically includes all of the following in order:

1. **Prayers for Meditation** — begins with "O Blessed One Shakyamuni Buddha, Precious Treasury of Compassion..."
2. **Refuge and Bodhichitta** — "I and all sentient beings, until we achieve enlightenment, go for refuge to Buddha, Dharma, and Sangha... Through the virtues I collect by giving and other perfections, may I become a Buddha for the benefit of all" (recited 3x)
3. **Four Immeasurables** — "May everyone be happy, may everyone be free from misery, may no one ever be separated from their happiness, may everyone have equanimity, free from hatred and attachment"
4. **Seven-Limb Prayer** — "In the space before me is the living Buddha Shakyamuni, surrounded by all the Buddhas and Bodhisattvas..."
5. **Mandala Offering** — "The ground sprinkled with perfume and spread with flowers..." (short mandala) followed by a longer mandala offering
6. **Lamrim Prayer (Je Tsongkhapa's)** — "The path begins with strong reliance on my kind teacher..." through to "...gain the state of Vajradhara"
7. **Receiving Blessings** — "From the hearts of all the holy beings, streams of light and nectar flow down..."

### Session 2 Opening Prayers — Abbreviated Practice

Session 2 typically uses a shorter version:

1. **Prayers for Meditation** (same as above)
2. **Retaking Bodhisattva Vows** — "Please listen to what I now say. From this time forth, until I attain enlightenment, I go for refuge to the Three Jewels..."
3. **Retaking Tantric Vows** — "I go for refuge to the Guru and Three Jewels. Holding Vajra and Bell, I generate as the deity and make offerings..."
4. Brief meditation connecting to the vows

### Closing Dedications

Dedications are typically brief and spoken (not chanted). The standard formula is:

> "Through the virtues we've collected through practicing, meditating, discussing — may all living beings find lasting peace of mind, enlightenment."

Sometimes followed by "we can make any personal prayers or dedications."

Occasionally the closing uses the **"nine-line Migtsema prayer"** or another short dedication prayer.

### Labeling Prayers in the Transcript

When marking up the transcript, prayers should be labeled consistently:

- Session 1 prayers: label as **"Opening Prayers"** or **"Session 1 Prayers"**
- Session 2 prayers: label as **"Session 2 Prayers"** or **"Session 2 Opening Prayers"**
- Keep the labeling convention consistent across all class files

---

## Jibber-Jabber: What to Exclude

The raw transcripts capture everything the microphone hears. A significant amount of this is **not class content** and must be excluded. Here's where to look and what to cut:

### 1. Before Class Starts

The recording often begins before class formally starts. You'll see casual conversation about:

- People arriving, finding seats, setting up
- Chit-chat about personal lives, the weather, weekend plans
- Comments about the physical space (temperature, seating, candles)
- Fragments of overlapping casual conversations

**How to identify:** This is everything before KK's first substantive remark (either the pre-prayers introduction or the call to begin prayers). The content will feel distinctly casual and unrelated to Dharma.

**Example of pre-class chatter to exclude:**
```
[Speaker 3] (0:22 - 0:55)
I'm annoyed at myself. I had so many packages to bring today
that I left my book bag on...

I had a bag of protein powder for you. Oh, I've been very excited.
```

### 2. Between Sessions (Dinner Break)

After the Session 1 dedication and before Session 2 prayers, there's a break period. This shows up as:

- Casual conversation between students
- Fragments of unclear or garbled speech
- Sometimes literal `...` or ellipses where the software couldn't transcribe
- Short, disconnected utterances
- Talk about food, work, personal matters

**How to identify:** Look for the Session 1 dedication phrase, then skip forward until you find the Session 2 prayers beginning. Everything between those two markers is break chatter.

**Example of break chatter to exclude:**
```
[Speaker 3] (1:59:54 - 2:00:11)
... ... ...

[Speaker 6] (2:00:11 - 2:00:14)
... ... ...

[Speaker 3] (2:00:17 - 2:00:50)
... So I think that, I mean, we can't say that we're tired,
but that's what we would sort of say to our own group...
```

### 3. During Group Discussion / Data Shares

The data share segments contain **mostly good content** but can also include:

- Cross-talk where multiple people speak at once (shows up as garbled fragments)
- Brief social exchanges that aren't about the practice topic
- Laughter or reactions that the software tries to transcribe as words
- Very short interjections that don't carry meaningful content

**Use judgment here.** If a student is sharing a genuine reflection on their practice, keep it. If it's just "yeah, yeah, yeah" or "that's cool," it can be trimmed.

### 4. After Class Ends

After the final dedication ("we can make any personal prayers or dedications, thank you all"), some transcripts continue recording:

- Goodbyes
- People packing up
- Casual post-class conversation
- Plans for the next week unrelated to practice

**Everything after the final dedication is excluded.**

### 5. Garbled / Nonsensical Text

The transcription software sometimes produces nonsensical output, especially when:

- Multiple people speak simultaneously
- Audio quality is poor
- The software mishears Dharma-specific terminology
- Background noise interferes

If a segment is genuinely unintelligible and clearly doesn't represent real class content, exclude it. If it's *mostly* intelligible with some errors, keep it and clean it up.

---

## Special Character & Encoding Cleanup

### Common Encoding Issues

The raw transcripts and HTML files can contain mangled Unicode characters. These are typically UTF-8 sequences that were misinterpreted or double-encoded. Common patterns:

| What You See | What It Should Be | Notes |
|---|---|---|
| `â€™` | `'` (right single quote / apostrophe) | Most common issue |
| `â€œ` | `"` (left double quote) | |
| `â€` or `â€\x9D` | `"` (right double quote) | |
| `â€"` | `—` (em dash) | |
| `â€"` | `–` (en dash) | |
| `Ã©` | `é` | Accented characters (e.g., in "Geshe") |
| `Ã¶` | `ö` | |
| `Ã¼` | `ü` | |
| `Â·` | `·` (middle dot) | Used as a separator |
| `Ã§` | `ç` | |
| `ðŸ"˜` | 📘 (emoji, book) | Emoji encoding issues |
| `ðŸ"œ` | 📜 (emoji, scroll) | |
| `ðŸ™` | 🙏 (emoji, prayer hands) | |
| `ChÃ¶gyam` | `Chögyam` | Specific to Chögyam Trungpa Rinpoche |

### Dharma-Specific Terms to Watch For

The transcription software often mangles Buddhist/Sanskrit/Tibetan terms. Common corrections:

| Transcription Error | Correct Term |
|---|---|
| "Bodhicitta" / "bodhicitta" | "bodhichitta" (preferred Kadampa spelling) |
| "Shantidiva" | "Shantideva" |
| "Geshe-la" | "Geshe-la" (this is correct) |
| "Lamrim" / "Lam Rim" | "Lamrim" (one word, preferred) |
| "Pratimoksha" | "Pratimoksha" (this is correct) |
| "samsara" | "samsara" (this is correct) |
| "nirvana" | "nirvana" (this is correct) |
| Various garbled mantras | Cross-reference with prayer texts |

### Cleaning Process

When processing transcripts:

1. **First pass — Source text replacement:** Identify all prayer sections and Shantideva verse readings. Replace these wholesale with the exact published text from `../texts/prayers-for-meditation.json` and `../texts/shantideva.json`. Do not attempt to "fix" the transcription — just swap it out entirely. (MTB passages are left as-is from the transcript — see note above.)
2. **Second pass — Encoding cleanup:** Search for known mangled patterns (`â€`, `Ã`, `Â`) in KK's teaching content and MTB readings, and replace with correct characters.
3. **Third pass — Dharma terminology:** Read through for Dharma-specific terms in KK's commentary and correct mistranscriptions.
4. **Verification:** Spot-check that replaced prayer text matches the correct prayer sequence for that session (Session 1 = full practice, Session 2 = abbreviated).

---

## Transcript Quality Control Checklist

Before considering a transcript "clean" and ready for class file assembly:

### Speaker Identification

- [ ] KK's speaker number(s) identified — all KK segments attributed correctly
- [ ] If KK was split across multiple speaker numbers, they've been merged
- [ ] Prayer chanter identified and labeled
- [ ] Student speakers identified by name where possible

### Structure Mapping

- [ ] Both sessions accounted for (unless confirmed single-session class)
- [ ] Session 1 and Session 2 boundaries clearly marked
- [ ] Pre-prayers introduction identified (if present)
- [ ] Prayers correctly identified and labeled for both sessions
- [ ] Meditation start/end marked
- [ ] Teaching segments identified
- [ ] Data share / practice sharing segments identified
- [ ] Practice discussion (final 10-15 min) identified

### Content Filtering

- [ ] Pre-class chatter removed
- [ ] Dinner break chatter removed
- [ ] Post-class chatter removed
- [ ] Garbled / nonsensical segments removed or flagged
- [ ] Cross-talk during discussions cleaned up

### Chronological Ordering

- [ ] Sections appear in the order they occurred — verified against timestamps
- [ ] "Class Introduction" content correctly placed (check actual timestamps, not section labels)
- [ ] Timestamps increase monotonically within each session

### Encoding & Terminology

- [ ] No mangled Unicode characters (`â€`, `Ã`, `Â`, `ðŸ` patterns)
- [ ] Dharma terms spelled correctly (especially "bodhichitta")
- [ ] **Prayer sections replaced verbatim** from `../texts/prayers-for-meditation.json` (not cleaned-up transcript versions)
- [ ] **Shantideva verses replaced verbatim** from `../texts/shantideva.json`
- [ ] MTB readings cleaned for encoding and Dharma term errors (but left as transcribed, not replaced verbatim)

---

## Technical Tools & Commands

### Useful grep/sed Patterns

```bash
# Find all unique speaker numbers in a transcript
grep -o "\[Speaker [0-9]*\]" TTP_MMDDYY.txt | sort -u

# Count appearances per speaker (helps identify KK — usually the most frequent)
grep -o "\[Speaker [0-9]*\]" TTP_MMDDYY.txt | sort | uniq -c | sort -rn

# Find where prayers begin (look for the distinctive opening)
grep -n "Shakyamuni Buddha\|treasury of compassion" TTP_MMDDYY.txt

# Find session transitions
grep -n "gradually arise\|let's take a look\|take a break\|come back\|welcome back\|let's dedicate" TTP_MMDDYY.txt

# Find encoding issues
grep -n "â€\|Ã\|Â\|ðŸ" TTP_MMDDYY.txt

# Find likely pre-class or break chatter (look for ... ellipsis patterns)
grep -n "\.\.\." TTP_MMDDYY.txt

# Check which speaker leads prayers
grep -B2 "Shakyamuni Buddha" TTP_MMDDYY.txt | grep "Speaker"

# Search for specific verse references
grep -n "verse\|([0-9]*)" TTP_MMDDYY.txt | head -20

# Find references to Meaningful to Behold page numbers
grep -n "page [0-9]\|page Roman\|turn to page" TTP_MMDDYY.txt
```

### Workflow Summary

1. **Identify speakers** — Run the grep commands above to figure out who's who
2. **Map the structure** — Find the prayer starts, meditation endings, dedications, and break to map out the two-session skeleton
3. **Trim jibber-jabber** — Identify and mark pre-class, break, and post-class content for exclusion
4. **Extract teaching content** — Pull out KK's teaching, text readings, and student shares
5. **Replace source texts verbatim** — Swap all prayers and Shantideva verse readings with exact published text from `../texts/prayers-for-meditation.json` and `../texts/shantideva.json`
6. **Clean encoding** — Fix mangled characters and Dharma term misspellings in KK's commentary and MTB readings
7. **Verify chronology** — Make sure everything is in timestamp order
8. **Quality control** — Run through the checklist above
9. **Hand off to class file assembly** — The clean transcript is now ready for the next stage (see `../classes/CLASS_FILE_GUIDE.md`)

---

## Appendix: Class Dates for This Series

| Class # | Date | File |
|---------|------|------|
| 1 | November 2, 2025 | `TTP_110225.txt` |
| 2 | November 9, 2025 | `TTP_110925.txt` |
| 3 | November 16, 2025 | `TTP_111625.txt` |
| 4 | November 30, 2025 | `TTP_113025.txt` |
| 5 | December 7, 2025 | `TTP_120725.txt` |
| 6 | December 14, 2025 | `TTP_121425.txt` |
| 7 | January 4, 2026 | `TTP_010426.txt` |
| 8 | January 11, 2026 | `TTP_011126.txt` |
| 9 | January 18, 2026 | `TTP_011826.txt` |

---

*Last updated: February 2, 2026*
