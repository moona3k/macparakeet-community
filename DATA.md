# MacParakeet Local Data Reference

> For developers and AI coding agents building on top of MacParakeet's local data.

MacParakeet stores all data locally on your Mac. No cloud, no accounts. This document describes the storage format so you can query, export, or build automations on top of your own data.

## Database

**Path:** `~/Library/Application Support/MacParakeet/macparakeet.db`

**Format:** SQLite (WAL mode)

**Access:**

```bash
sqlite3 ~/Library/Application\ Support/MacParakeet/macparakeet.db
```

> **Note:** The app may have the database open. Reads are always safe. Avoid writes while the app is running — use the app's UI or wait until it's closed.

---

## Tables

### `dictations`

System-wide voice dictations (press hotkey, speak, text is pasted).

| Column | Type | Description |
|--------|------|-------------|
| `id` | TEXT (UUID) | Primary key |
| `createdAt` | TEXT (ISO 8601) | When the dictation was recorded |
| `durationMs` | INTEGER | Recording duration in milliseconds |
| `rawTranscript` | TEXT | Original transcription from STT engine |
| `cleanTranscript` | TEXT | Processed transcript (filler removal, custom words, snippets). Null if processing mode is "raw" |
| `audioPath` | TEXT | Absolute path to saved audio file (`.wav`). Null if audio saving is off |
| `pastedToApp` | TEXT | Bundle ID of the app where text was pasted |
| `processingMode` | TEXT | `"raw"` or `"clean"` |
| `status` | TEXT | `"recording"`, `"processing"`, `"completed"`, `"error"` |
| `errorMessage` | TEXT | Error details if status is `"error"` |
| `hidden` | INTEGER (bool) | `1` if created in private dictation mode (excluded from history) |
| `wordCount` | INTEGER | Word count of the final transcript |
| `updatedAt` | TEXT (ISO 8601) | Last modified |

**Full-text search:** The `dictations_fts` virtual table (FTS5) indexes `rawTranscript` and `cleanTranscript` for fast substring search.

```sql
-- Search dictation history
SELECT d.* FROM dictations d
JOIN dictations_fts fts ON d.rowid = fts.rowid
WHERE dictations_fts MATCH 'meeting notes'
ORDER BY d.createdAt DESC;
```

### `transcriptions`

File and YouTube transcriptions.

| Column | Type | Description |
|--------|------|-------------|
| `id` | TEXT (UUID) | Primary key |
| `createdAt` | TEXT (ISO 8601) | When the transcription was created |
| `fileName` | TEXT | Original file name (e.g., `"interview.mp3"`) |
| `filePath` | TEXT | Absolute path to the source file |
| `fileSizeBytes` | INTEGER | Source file size |
| `durationMs` | INTEGER | Audio duration in milliseconds |
| `rawTranscript` | TEXT | Original transcription |
| `cleanTranscript` | TEXT | Processed transcript |
| `wordTimestamps` | TEXT (JSON) | Word-level timing data (see [JSON formats](#word-timestamps) below) |
| `language` | TEXT | Detected language code (e.g., `"en"`) |
| `speakerCount` | INTEGER | Number of detected speakers |
| `speakers` | TEXT (JSON) | Speaker metadata (see [JSON formats](#speakers) below) |
| `diarizationSegments` | TEXT (JSON) | Speaker diarization segments (see [JSON formats](#diarization-segments) below) |
| `summary` | TEXT | LLM-generated summary (if user requested one) |
| `chatMessages` | TEXT (JSON) | Chat conversation history about this transcript (see [JSON formats](#chat-messages) below) |
| `status` | TEXT | `"processing"`, `"completed"`, `"error"`, `"cancelled"` |
| `errorMessage` | TEXT | Error details if status is `"error"` |
| `exportPath` | TEXT | Path where the user last exported this transcription |
| `sourceURL` | TEXT | YouTube URL (if transcribed from YouTube) |
| `updatedAt` | TEXT (ISO 8601) | Last modified |

### `custom_words`

User-defined word corrections applied during clean processing.

| Column | Type | Description |
|--------|------|-------------|
| `id` | TEXT (UUID) | Primary key |
| `word` | TEXT | Word to match (case-insensitive, unique) |
| `replacement` | TEXT | Replacement text. Null means "keep the word as-is" (capitalization fix) |
| `source` | TEXT | `"manual"` (user-created) or `"learned"` |
| `isEnabled` | INTEGER (bool) | Whether this rule is active |
| `createdAt` | TEXT (ISO 8601) | Created |
| `updatedAt` | TEXT (ISO 8601) | Last modified |

### `text_snippets`

Text expansion shortcuts applied during clean processing.

| Column | Type | Description |
|--------|------|-------------|
| `id` | TEXT (UUID) | Primary key |
| `trigger` | TEXT | Trigger phrase (case-insensitive, unique) |
| `expansion` | TEXT | Replacement text (supports `\n` for newlines) |
| `isEnabled` | INTEGER (bool) | Whether this snippet is active |
| `useCount` | INTEGER | Number of times this snippet has been expanded |
| `createdAt` | TEXT (ISO 8601) | Created |
| `updatedAt` | TEXT (ISO 8601) | Last modified |

---

## JSON Column Formats

### Word Timestamps

`transcriptions.wordTimestamps` — array of word-level timing data:

```json
[
  { "word": "Hello", "startMs": 0, "endMs": 350, "confidence": 0.98, "speakerId": "S1" },
  { "word": "world", "startMs": 360, "endMs": 680, "confidence": 0.95, "speakerId": "S1" }
]
```

| Field | Type | Description |
|-------|------|-------------|
| `word` | string | The transcribed word |
| `startMs` | int | Start time in milliseconds |
| `endMs` | int | End time in milliseconds |
| `confidence` | float | Confidence score (0.0–1.0) |
| `speakerId` | string? | Speaker ID (e.g., `"S1"`), present when diarization is enabled |

### Speakers

`transcriptions.speakers` — speaker metadata:

```json
[
  { "id": "S1", "label": "Speaker 1" },
  { "id": "S2", "label": "Speaker 2" }
]
```

### Diarization Segments

`transcriptions.diarizationSegments` — time ranges per speaker:

```json
[
  { "speakerId": "S1", "startMs": 0, "endMs": 5200 },
  { "speakerId": "S2", "startMs": 5300, "endMs": 12400 }
]
```

### Chat Messages

`transcriptions.chatMessages` — conversation history with LLM about a transcript:

```json
[
  { "role": "system", "content": "You are analyzing a transcript..." },
  { "role": "user", "content": "What are the key points?" },
  { "role": "assistant", "content": "The main points discussed were..." }
]
```

| Role | Description |
|------|-------------|
| `system` | System prompt (context for the LLM) |
| `user` | User's question |
| `assistant` | LLM's response |

---

## File Paths

| Item | Path |
|------|------|
| Database | `~/Library/Application Support/MacParakeet/macparakeet.db` |
| Dictation audio | `~/Library/Application Support/MacParakeet/dictations/{uuid}.wav` |
| YouTube downloads | `~/Library/Application Support/MacParakeet/youtube-downloads/` |
| Temp audio (processing) | `$TMPDIR/macparakeet/` |

---

## Example Queries

```sql
-- All completed transcriptions with summaries
SELECT fileName, summary, createdAt
FROM transcriptions
WHERE status = 'completed' AND summary IS NOT NULL
ORDER BY createdAt DESC;

-- Total dictation time (in minutes)
SELECT ROUND(SUM(durationMs) / 60000.0, 1) AS total_minutes
FROM dictations
WHERE status = 'completed';

-- Recent dictations with word count
SELECT rawTranscript, wordCount, createdAt
FROM dictations
WHERE status = 'completed' AND hidden = 0
ORDER BY createdAt DESC
LIMIT 20;

-- YouTube transcriptions
SELECT fileName, sourceURL, durationMs / 60000.0 AS minutes
FROM transcriptions
WHERE sourceURL IS NOT NULL
ORDER BY createdAt DESC;

-- Full-text search across dictations
SELECT d.rawTranscript, d.createdAt
FROM dictations d
JOIN dictations_fts fts ON d.rowid = fts.rowid
WHERE dictations_fts MATCH 'your search term'
ORDER BY d.createdAt DESC;

-- Transcriptions with chat history
SELECT fileName, summary,
       json_array_length(chatMessages) AS message_count
FROM transcriptions
WHERE chatMessages IS NOT NULL;

-- Custom words list
SELECT word, replacement, isEnabled
FROM custom_words
ORDER BY word;

-- Most-used text snippets
SELECT "trigger", expansion, useCount
FROM text_snippets
WHERE isEnabled = 1
ORDER BY useCount DESC;
```
