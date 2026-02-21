# Changelog

All notable changes to MacParakeet are documented here.

MacParakeet is in active development. This changelog tracks what's been built and what's coming next.

---

## Unreleased

### Command Mode (in progress)

Voice-driven text editing. Select text anywhere on your Mac, speak a command, and the local LLM rewrites it in place.

- Core command services and hotkey arbitration
- Command overlay with selection preview
- Accessibility-based text capture and replacement

### Coming Soon

- Additional export formats (DOCX, PDF, JSON)
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

### Chat with Transcript

Ask questions about any transcription using the local Qwen3-8B model. Great for pulling out key points, summarizing meetings, or finding specific moments.

- Chat panel alongside transcript results
- Full transcript context via 128K token window
- Suggested prompt chips for common questions
- Works entirely offline — your transcripts stay private

### Export Formats

Export transcriptions in the format that fits your workflow.

- **Plain text** — Clean transcript, no formatting
- **Markdown** — Structured with timestamps as headings
- **SRT** — Industry-standard subtitle format
- **VTT** — Web-native subtitle format (WebVTT)

---

## v0.2 — AI Text Processing

### Clean Text Pipeline

A deterministic four-step pipeline that cleans up raw dictation before you see it. No AI required — just fast, predictable text cleanup.

1. **Filler removal** — Strips "um", "uh", "like", "you know", and similar verbal fillers
2. **Custom words** — Corrects domain-specific terms (e.g., "Kubernotes" → "Kubernetes")
3. **Snippet expansion** — Expands trigger phrases into full text (e.g., "my signature" → your full sign-off)
4. **Whitespace normalization** — Cleans up double spaces, leading/trailing whitespace

### AI Text Refinement

When the deterministic pipeline isn't enough, Qwen3-8B refines your text locally on the GPU. Three modes:

- **Formal** — Professional tone, complete sentences, proper grammar
- **Email** — Ready-to-send email format with greeting and sign-off
- **Code** — Technical documentation style, precise terminology

Falls back gracefully to the deterministic pipeline if the LLM is unavailable.

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
- Freed the GPU entirely for Qwen3-8B LLM tasks

### Local Model Management

Full lifecycle management for both local models (Parakeet STT + Qwen3-8B LLM).

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
- **Three-chip architecture**: CPU (app), GPU (Qwen3-8B via Metal), ANE (Parakeet via CoreML)
- **Parakeet TDT 0.6B-v3** — ~2.5% word error rate, 155x realtime on Apple Silicon
- **Qwen3-8B** — 128K context, 4-bit quantization, ~5 GB GPU RAM
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
macparakeet-cli llm generate <prompt>         Generate text with Qwen3-8B
macparakeet-cli llm refine <mode> <text>      Refine text (formal/email/code)
macparakeet-cli llm command <cmd> <text>      Voice command simulation
macparakeet-cli llm chat <question>           Chat with optional transcript context
macparakeet-cli llm smoke-test                Quick model readiness check
```

---

*MacParakeet is built by [moona3k](https://github.com/moona3k). Follow this repo for updates.*
