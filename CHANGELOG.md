# Changelog

All notable changes to MacParakeet are documented here.

MacParakeet is in active development. This changelog tracks what's been built and what's coming next.

---

## v0.4 — Streamlining and Polish

Cut everything slow, ship everything useful, polish what stays.

### Custom Hotkey Support

Set any single key as your dictation trigger — not just modifier keys. A key recorder in Settings lets you press the key you want and see it applied immediately. Supports letters, numbers, punctuation, function keys, and all modifier keys.

### DOCX, PDF, and JSON Export

Three new export formats join TXT, Markdown, SRT, and VTT:

- **DOCX** — Word document with timestamps and formatting
- **PDF** — Print-ready transcript
- **JSON** — Structured data with word-level timestamps and confidence scores

### Menu Bar Drag-and-Drop

Drop audio/video files directly onto the menu bar icon to start transcription. No need to open the main window first.

### Hide Dictation Pill

New toggle: "Show dictation pill at all times." Turn it off and the pill only appears while actively recording — gone when idle.

### Voice Stats Dashboard

Dictation history now opens with a stats overview:

- **Total Words** — lifetime count with real-world comparison ("59 emails worth")
- **Time Speaking** — cumulative duration with session count
- **Voice Speed** — average WPM with pace classification
- **Time Saved** — estimated hours saved vs typing at 40 WPM
- **Weekly Streak** — consecutive weeks with at least one dictation

### UI Polish

- Switch toggles in Settings (replacing checkboxes)
- Sidebar reorganized into labeled sections
- Feedback form tightened — email and screenshot side-by-side, drag-and-drop screenshot zone
- ~15 developer jargon strings replaced with user-friendly copy
- New app icon
- Grid card layout fixes across Feedback and Settings

### LLM Removal

Removed all local LLM features (Qwen3-8B / MLX-Swift). Parakeet STT is fast (~155x realtime), but the LLM added unacceptable latency for text refinement, command mode, and chat. Rather than ship slow features, we cut them.

**Removed:** AI Text Refinement (formal/email/code modes), Command Mode, Chat with Transcript, `mlx-swift-lm` dependency, `macparakeet-cli llm`.

**Kept:** Dictation, file transcription, YouTube transcription, all 7 export formats, dictation history, custom words, text snippets, clean text pipeline.

| Metric | Before | After |
|--------|--------|-------|
| Peak memory | ~5.3 GB | ~300 MB |
| Architecture | 3-chip (CPU/GPU/ANE) | 2-chip (CPU/ANE) |
| Processing modes | 5 (raw, clean, formal, email, code) | 2 (raw, clean) |
| Package deps | 4 | 3 (removed mlx-swift-lm) |

We'll bring LLM features back when on-device inference is fast enough to feel instant rather than annoying. Monitoring smaller architectures, speculative decoding, and hardware improvements.

### Bug Fixes

- **FFmpeg audio conversion failure** — Video file transcription could fail silently on certain containers. Now surfaces clear error messages.
- **FFmpeg discovery** — Extended PATH search for debug builds so FFmpeg is found reliably.
- **YouTube progress bar stuck at 0%** — Fixed download progress parsing.
- **Disk image guard** — App now detects if it's running from a DMG and prompts the user to move it to Applications first.
- **Deprecated mode fallback** — Users with "formal"/"email"/"code" in UserDefaults now correctly fall back to clean mode (was silently falling back to raw).
- **Onboarding offline regression** — Skip network checks when the speech model is already cached locally.
- **Stats word count accuracy** — Fixed counting for non-space whitespace and long space runs; moved word counting from SQL to Swift.

### Coming Soon

- Speaker diarization
- Batch file processing
- Whisper mode (optimized for quiet speech)

---

## v0.3 — YouTube, Chat, and Export

### YouTube URL Transcription

Paste a YouTube URL and get a full transcript — no browser extensions, no cloud services. MacParakeet downloads the audio with yt-dlp and transcribes it locally with Parakeet.

- Single-video transcription from any YouTube URL
- Real-time download progress with percentage tracking
- Audio chunking for long videos (1hr+ content works reliably)
- Option to keep or auto-delete downloaded audio after transcription

### ~~Chat with Transcript~~ *(removed in v0.4)*

~~Ask questions about any transcription using the local Qwen3-8B model.~~ Removed due to LLM latency. See [v0.4](#v04--streamlining-and-polish).

### Export Formats

Export transcriptions in the format that fits your workflow.

- **Plain text** — Clean transcript, no formatting
- **Markdown** — Structured with timestamps as headings
- **SRT** — Industry-standard subtitle format
- **VTT** — Web-native subtitle format (WebVTT)
- **DOCX, PDF, JSON** — Added in [v0.4](#v04--streamlining-and-polish)

---

## v0.2 — AI Text Processing

### Clean Text Pipeline

A deterministic four-step pipeline that cleans up raw dictation before you see it. No AI required — just fast, predictable text cleanup.

1. **Filler removal** — Strips "um", "uh", "like", "you know", and similar verbal fillers
2. **Custom words** — Corrects domain-specific terms (e.g., "Kubernotes" → "Kubernetes")
3. **Snippet expansion** — Expands trigger phrases into full text (e.g., "my signature" → your full sign-off)
4. **Whitespace normalization** — Cleans up double spaces, leading/trailing whitespace

### ~~AI Text Refinement~~ *(removed in v0.4)*

~~Qwen3-8B text refinement with formal, email, and code modes.~~ Removed due to LLM latency. See [v0.4](#v04--streamlining-and-polish).

### Custom Words and Snippets

Teach MacParakeet your vocabulary.

- **Custom words** — Correct specific misrecognitions or enforce spelling (case-insensitive matching)
- **Vocabulary anchors** — Words to recognize without replacement (proper nouns, brand names)
- **Text snippets** — Map trigger phrases to expanded text (supports multi-line)
- Full management UI in the Vocabulary panel with enable/disable toggles

### FluidAudio CoreML Migration

Moved the entire speech-to-text pipeline from a Python daemon to native Swift via FluidAudio CoreML.

- Parakeet TDT runs directly on the Neural Engine (ANE) — no Python, no GPU contention
- ~66 MB working memory during inference (down from ~2 GB on GPU)
- ~155x realtime speed on Apple Silicon

### Local Model Management

Lifecycle management for the Parakeet speech model.

- Status panel in Settings showing model state (Ready / Not Loaded / Not Downloaded)
- One-click repair with automatic retry and exponential backoff
- CLI commands: `macparakeet-cli models status`, `warm-up`, `repair`

---

## v0.1 — Core MVP

The foundation. Two co-equal modes — dictation and transcription — built from the ground up for speed and privacy.

### System-Wide Dictation

Press a hotkey anywhere on macOS, speak, and your words appear as text. Works in any app.

- **Double-tap** the trigger key for persistent recording (tap again to stop)
- **Hold** the trigger key for push-to-talk (release to stop)
- Configurable trigger key: Fn, Control, Option, Shift, or Command
- Auto-paste with clipboard save and restore — your clipboard is never overwritten
- Optional silence auto-stop with configurable delay (1–5 seconds)

### Dictation Overlay

A compact dark pill that appears over all windows without stealing focus from your current app.

- Real-time waveform visualization showing audio levels
- Recording timer
- Cancel and stop controls (Escape key also cancels)
- Persistent idle pill — always visible, click to start dictating

### File Transcription

Drag and drop audio or video files for full transcription powered by Parakeet TDT.

- Supports common audio formats (MP3, WAV, M4A, FLAC, OGG) and video formats (MP4, MOV, MKV, WebM)
- FFmpeg-based audio extraction for video files
- Word-level timestamps and confidence scores
- Portal-style animated drop zone

### Dictation History

Everything you've dictated, searchable and playable.

- Flat list with bottom bar audio player
- Full-text substring search with match highlighting
- Copy, re-process, and delete individual entries
- Date-grouped display with duration and word count

### Settings

- Trigger key selection with live preview
- Silence auto-stop toggle and delay picker
- Audio recording retention toggle
- Microphone and Accessibility permission status
- Menu-bar-only mode (hide the Dock icon)
- Launch at login

### First-Run Onboarding

Guided setup that walks new users through permissions, hotkey basics, and local model preparation.

- Step-by-step permission requests (Microphone, Accessibility)
- Model download and warm-up with progress feedback
- Retry with self-healing for failed downloads

### Licensing

- 7-day free trial starting at onboarding completion
- License key activation via LemonSqueezy
- Unlimited grace period — activate once, yours forever
- No accounts, no email required

### Architecture

- **100% local** — audio never leaves your Mac
- **Two-chip architecture**: CPU (app) + ANE (Parakeet via CoreML)
- **Parakeet TDT 0.6B-v3** — ~2.5% word error rate, 155x realtime on Apple Silicon
- **~300 MB peak memory** — lightweight, GPU-free
- **SQLite** (GRDB) for dictation history and transcription records
- macOS 14.2+ on Apple Silicon

---

## Developer Tools

MacParakeet includes a CLI for headless testing and automation.

```
macparakeet-cli transcribe <file-or-url>     Transcribe audio/video or YouTube URL
macparakeet-cli history [dictations|search]   Browse and search dictation history
macparakeet-cli health [--repair-models]      System diagnostics and model repair
macparakeet-cli flow process <text>           Run the clean text pipeline
macparakeet-cli flow words [list|add|delete]  Manage custom words
macparakeet-cli flow snippets [list|add|del]  Manage text snippets
macparakeet-cli models [status|warm-up|repair] Model lifecycle management
```

---

*MacParakeet is built by [moona3k](https://github.com/moona3k). Follow this repo for updates.*
