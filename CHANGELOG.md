# Changelog

All notable changes to MacParakeet are documented here.

MacParakeet is in active development. This changelog tracks what's been built and what's coming next.

---

## v0.4 — Streamlining and Polish

Cut everything slow, ship everything useful, polish what stays.

### Inline Model Selector (0.4.8)

Switch LLM models directly from the chat input bar or summary pane — no need to navigate to Settings. The selector shows suggested models for your active provider with a compact dropdown. Disabled during streaming to prevent mid-response confusion.

### Delete Transcriptions (0.4.7)

Right-click any transcription in the sidebar to delete it. Cleans up both the database record and any downloaded audio files.

### Transcription Progress in Menu Bar (0.4.9)

The menu bar icon now shows a yellow processing dot while file or YouTube transcription is running. Useful when you've navigated away from the app — you can tell at a glance that your transcription is still in progress. Dictation recording (red dot) takes priority if both are active.

### Menu Bar Icon Polish (0.4.8)

Fixed the menu bar icon glyph changing color when showing recording (red) or processing (yellow) status indicators. The parakeet icon now stays consistent across all states.

### Smarter Filler Removal (0.4.7)

The clean text pipeline now only removes actual hesitation sounds (um, uh, umm, uhh) — previously it was too aggressive and could strip intentional words.

### Cancel Transcription Fix (0.4.7)

Cancelling a transcription now properly terminates all subprocesses (FFmpeg, yt-dlp) with a confirmation dialog, instead of leaving orphaned processes.

### LLM Provider Integration

Bring your own API key and unlock AI-powered transcript features. Connect any OpenAI-compatible provider — or Anthropic and Ollama natively — and get instant summaries and conversational Q&A for any transcription.

- **Summary tab** — One-click transcript summary, persisted per transcription
- **Chat tab** — Ask questions about the transcript with full conversation history
- **Providers** — OpenAI, Anthropic, Ollama (local), OpenRouter, or any OpenAI-compatible endpoint
- **Dynamic model lists** — Models fetched from your provider automatically
- **Settings card** — Configure provider, API key, and model from Transcription settings
- Streaming responses with markdown rendering

100% optional — MacParakeet works fully offline without any LLM provider configured.

### Private Dictation Mode

New toggle in Settings: turn off dictation history saving entirely. When enabled, dictated text is pasted as usual but nothing is stored in the database — no transcript, no audio, no history entry. For users who want dictation without a paper trail.

Shipped in response to community feedback ([#14](https://github.com/moona3k/macparakeet-community/issues/14)).

### Newline Support in Text Snippets

Text snippets now support `\n` as a newline escape. A snippet with the replacement `Line one\nLine two` expands to actual multi-line text. Useful for signatures, email templates, and formatted blocks.

Shipped in response to community feedback ([#16](https://github.com/moona3k/macparakeet-community/issues/16)).

### Auto-Updates

MacParakeet now updates itself. When a new version is available, you'll see a native update dialog — click "Install Update" and the app downloads, installs, and relaunches automatically. No more manual DMG re-downloads.

- Automatic background update checks (configurable in Settings > Updates)
- Manual "Check for Updates..." in the app menu and menu bar dropdown
- Secure EdDSA-signed updates via [Sparkle](https://sparkle-project.org/)

Shipped in response to community feedback ([#11](https://github.com/moona3k/macparakeet-community/issues/11)).

### Custom Hotkey Support

Set any single key — or modifier+key chord — as your dictation trigger. A key recorder in Settings lets you press the key you want and see it applied immediately. Supports letters, numbers, punctuation, function keys, modifier keys, and modifier+key combos (e.g., Cmd+9). See also [Chord Hotkey Support](#chord-hotkey-support).

### DOCX, PDF, and JSON Export

Three new export formats join TXT, Markdown, SRT, and VTT:

- **DOCX** — Word document with timestamps and formatting
- **PDF** — Print-ready transcript
- **JSON** — Structured data with word-level timestamps and confidence scores

### Menu Bar Drag-and-Drop

Drop audio/video files directly onto the menu bar icon to start transcription. No need to open the main window first.

### Hide Dictation Pill

New toggle: "Show dictation pill at all times." Turn it off and the pill only appears while actively recording — gone when idle.

### Dictation Mode Discoverability

Redesigned how users learn the two dictation modes:

- **Onboarding** — Step 4 replaced with two animated mode cards (Persistent Mode and Push-to-Talk) with gesture illustrations instead of a numbered text list
- **Settings** — New quick-reference strip in the Dictation card showing both modes with key cap visuals; updates reactively when you change your hotkey

Shipped in response to community feedback ([#5](https://github.com/moona3k/macparakeet-community/issues/5), [#6](https://github.com/moona3k/macparakeet-community/issues/6)).

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

### Local LLM Removal → Cloud LLM Integration

Removed all local LLM features (Qwen3-8B / MLX-Swift) due to unacceptable on-device latency, then re-introduced LLM capabilities via cloud API providers. The result: fast, high-quality AI features without the GPU memory overhead.

**Removed:** Local Qwen3-8B inference, `mlx-swift-lm` dependency, AI Text Refinement modes (formal/email/code), Command Mode.

**Replaced with:** Cloud-powered summary and chat via bring-your-own-API-key (see [LLM Provider Integration](#llm-provider-integration) above).

| Metric | Local LLM (before) | Cloud LLM (after) |
|--------|--------|-------|
| Peak memory | ~5.3 GB | ~300 MB |
| Architecture | 3-chip (CPU/GPU/ANE) | 2-chip (CPU/ANE) |
| Response quality | Limited by 8B model | GPT-4o / Claude / any provider |
| Latency | 5-15s local inference | ~1-3s streaming |
| Package deps | 4 | 4 (replaced mlx-swift-lm with Sparkle) |

### Speaker Diarization

Identify who said what. MacParakeet can now detect and label individual speakers in any transcription.

- **GUI** — Speaker summary panel shows each speaker with word count and speaking time. Click any speaker label to rename (e.g., "Speaker 1" → "Alice").
- **CLI** — `macparakeet-cli transcribe recording.mp3 --diarize`
- **Inline labels** — Speaker turns appear in the transcript view and all export formats (TXT, Markdown, SRT, VTT, DOCX, PDF, JSON)
- Powered by FluidAudio's Sortformer diarization model, running locally on the Neural Engine

### Chord Hotkey Support

Set modifier+key combos as your dictation trigger — not just single keys. Cmd+9, Option+Space, Ctrl+Shift+D, or any combination you like. The key recorder in Settings captures the full chord.

### Discover

A curated reading surface in the sidebar. Suppressed patents, buried science, ancient wisdom — things worth knowing that don't make it into the mainstream.

- **Sidebar card** — Rotating preview pinned below the sidebar, cycles through content every 30 seconds
- **Full feed** — Scrollable detail view with editorial card design, watermark icons, and typographic treatments (serif for quotes, rounded for affirmations)
- **Copy any card** — Hover to reveal copy button, includes title + body + attribution
- **Share your thoughts** — Private feedback form at the bottom of the Discover tab (does not go to GitHub)
- Content served from `macparakeet.com/api/discover.json` with bundled fallback for offline use

### Non-blocking Transcription Progress

File transcription now runs in the background. A progress bar in the bottom bar shows status while you browse history, read other transcripts, or keep dictating.

### Bug Fixes

- **Export confirmation blank on first use** — The popover after exporting showed just "Exported" with no format, filename, or "Show in Finder" button on the first export. Fixed by making the popover data-driven ([#9](https://github.com/moona3k/macparakeet-community/issues/9)).
- **PDF export deadlock** — PDF export could freeze the app. Replaced `NSPrintOperation` with a direct `CGContext` renderer.
- **FFmpeg audio conversion failure** — Video file transcription could fail silently on certain containers. Now surfaces clear error messages.
- **FFmpeg discovery** — Extended PATH search for debug builds so FFmpeg is found reliably.
- **YouTube progress bar stuck at 0%** — Fixed download progress parsing.
- **Disk image guard** — App now detects if it's running from a DMG and prompts the user to move it to Applications first.
- **Deprecated mode fallback** — Users with "formal"/"email"/"code" in UserDefaults now correctly fall back to clean mode (was silently falling back to raw).
- **Onboarding offline regression** — Skip network checks when the speech model is already cached locally.
- **Stats word count accuracy** — Fixed counting for non-space whitespace and long space runs; moved word counting from SQL to Swift.
- **Dictation history not refreshing after Clear All** — History list required a restart to reflect cleared entries. Now refreshes immediately ([#12](https://github.com/moona3k/macparakeet-community/issues/12)).
- **Ollama URL builder crash** — Force-unwrap on malformed Ollama base URLs could crash the app. Now uses safe URL construction.
- **Main window closing in menu bar mode** — Toggling menu-bar-only mode could dismiss the main window unexpectedly.
- **Summary persisting to wrong transcription** — Navigating to a different transcript mid-stream could save the summary to the wrong record.
- **Dictation pasting empty text** — When "Save Dictation History" was off, dictation could paste empty text instead of the transcription.
- **Export race condition** — Rapid exports could collide; speaker rename text field lost focus on save.
- **LLM custom model validation** — Custom model names weren't validated before sending to the provider.
- **Chord hotkey edge cases** — Fixed modifier-only chords and key-up detection for chord combos.

### Coming Soon

- Batch file processing

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
