# Changelog

All notable changes to MacParakeet are documented here.

MacParakeet is in active development. This changelog tracks what's been built and what's coming next.

---

## v0.4 — LLM Removal and Streamlining

Removed all local LLM features (Qwen3-8B / MLX-Swift). Parakeet STT is fast (~155x realtime), but the LLM added unacceptable latency for text refinement, command mode, and chat. Rather than ship slow features, we cut them to keep MacParakeet focused on what it does best: fast local dictation and transcription.

### What was removed

- **AI Text Refinement** — Formal, email, and code processing modes (powered by Qwen3-8B)
- **Command Mode** — Highlight text + voice command for LLM-powered in-place editing
- **Chat with Transcript** — Ask questions about transcriptions via local LLM
- **`mlx-swift-lm` dependency** — MLX-Swift and all Metal shader compilation
- **`macparakeet-cli llm`** — CLI subcommand for LLM operations

### What stays

Dictation, file transcription, YouTube URL transcription, export (TXT/MD/SRT/VTT), dictation history, custom words, text snippets, and the clean text pipeline — all unchanged.

### Impact

| Metric | Before | After |
|--------|--------|-------|
| Peak memory | ~5.3 GB | ~300 MB |
| Architecture | 3-chip (CPU/GPU/ANE) | 2-chip (CPU/ANE) |
| Processing modes | 5 (raw, clean, formal, email, code) | 2 (raw, clean) |
| Package deps | 4 | 3 (removed mlx-swift-lm) |

### Bug fixes

- **Deprecated mode fallback** — Users who had "formal", "email", or "code" stored in UserDefaults were silently downgraded to raw mode (no processing). Now correctly falls back to clean mode.
- **Onboarding offline regression** — Users who reset onboarding while offline were blocked even if the speech model was already cached. Fixed to skip network checks when the model exists locally.
- **Stale UI strings** — "Local Models" → "Speech Model", "Repair All" → "Repair", "10 GB" → "7 GB"

### A note on local LLM features

We plan to bring these back. The capability isn't the problem — Qwen3-8B is a solid general-purpose model for its size — the problem is speed. On-device inference on Apple Silicon (even with MLX) tops out around 25 tokens/sec for 8B models, and that's on high-end machines. For interactive use cases like command mode and text refinement, that latency turns a feature that should feel instant into one that feels annoying.

For context: cloud models like Claude Opus 4.6 and Gemini 3 Pro operate at a completely different level of both intelligence and throughput. Local models aren't there yet, and we don't want to ship something that feels like a compromise.

We're actively monitoring local model releases — smaller architectures, faster inference engines, speculative decoding, and hardware improvements. When internal testing shows that on-device LLM features are fast enough to be genuinely delightful rather than merely functional, we'll bring them back. Until then, MacParakeet stays focused on what it already does at world-class speed: local speech-to-text.

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

### ~~Chat with Transcript~~ *(removed in v0.4)*

~~Ask questions about any transcription using the local Qwen3-8B model.~~ Removed due to LLM latency. See [v0.4](#v04--llm-removal-and-streamlining).

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

### ~~AI Text Refinement~~ *(removed in v0.4)*

~~Qwen3-8B text refinement with formal, email, and code modes.~~ Removed due to LLM latency. See [v0.4](#v04--llm-removal-and-streamlining).

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
