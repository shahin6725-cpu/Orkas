# Shahin Assistant — Initial Ecosystem Audit

Date: 2026-11-26

This branch is reserved for a read-only ecosystem audit. No production source files have been changed.

## Current direction
- Keep Orkas as an execution-host candidate, not as the identity or architecture of Shahin Assistant.
- Shahin Core owns identity, SCKG, durable decisions, policy, approvals, and AI routing.
- Keep external engines independent and connect them through version-pinned adapters.
- Do not equate Orkas/DSH session memory with persistent SCKG knowledge.

## Orkas source review — confirmed findings
- The repository README describes VoiceStudio, OfficeWorker, DeepResearcher, StockAnalyser, Commander, Project Memory, To-do, VideoStudio, and ImageStudio.
- `src/main/model/core-agent/office-tools.ts` implements DOCX/XLSX/PPTX creation and review tools. The implementation uses path sandboxing, re-reads local-execution permission, and closes OfficeCLI residents.
- `src/main/features/office/office_engine.ts` is the central OfficeCLI process wrapper.
- `src/main/features/vec_store.ts` implements SQLite/sqlite-vec vector storage. Its comments specify `bge-small-zh-v1.5`; multilingual/Persian retrieval quality must be benchmarked before treating it as the SCKG retrieval backend.
- `.github/workflows/native-dependencies.yml` includes Windows, Linux and macOS runners; `.github/workflows/linux-source-smoke.yml` is Linux-specific. CI configuration is not proof that the current revision passed.

## Persian localization — confirmed gap
- `fa` is absent from the main language union/locale registry.
- `src/main/locales/fa.json` and `src/renderer/locales/fa.json` are absent.
- Renderer RTL is currently Arabic-only; passkey dialog RTL is Arabic-only.
- `shahin/persian-fa-localization` currently has no diff against `main` (0 ahead, 0 behind, 0 changed files).

Required localization work: add independent `fa` / `fa-IR` main and renderer locale entries; provide complete human-reviewed dictionaries; enable RTL for `ar` and `fa` while retaining LTR for code, terminal, URLs and paths; cover dialogs and built-in apps; add locale parity, missing-key and RTL tests. Arabic must remain unchanged.

## Account repositories found
The connected account listing returned these ten repositories: Orkas, opencode, codex, deepseek-harness, gemini-cli, cline, OpenHands, ECC, ccxt, TradingAgents. Preserve each upstream history and license. Do not copy entire engines into Orkas.

## Candidate capabilities for separate evaluation
- Persian meeting assistant: `Dr-Bagheri/neurai-mvp` (Apache-2.0 per repository metadata/README): Persian transcription, diarization, meeting minutes, action items, search and document Q&A. Evaluate on Windows and the HP Victus before adoption.
- Knowledge capture: `jagdeepvirdi/memex` (MIT): ingest notes, links, PDFs, Office documents and voice memos; semantic retrieval and entity graph. Evaluate overlap with Orkas Library and SCKG.
- `khoj-ai/khoj` is AGPL-3.0: consider only as a separately deployed service after license and operations review, not embedded in Shahin Core.
- Obsidian is not an open-source app to fork; Markdown vault interoperability and graph/Canvas ideas can be considered without depending on its proprietary app.
- `ForgeLabsSoft/ai-file-butler` is GPL-3.0 per repository README: separate evaluation only; preview, approval, journaling and undo are mandatory for batch file actions.
- `EwertonDCSilv/files-organizer` declares PolyForm Noncommercial; exclude from a commercial/product integration path.
- `CrispStrobe/CrispSorter` declares AGPL-3.0 for the application; no code integration without a deliberate license review.
- `ard9/Audio8-Persian-TTS` publishes a Persian TTS fine-tune; verify model-artifact terms, voice quality, latency and consent safeguards before selecting it.

## Integration safeguards
1. SCKG is separate from session transcripts and project-local memory.
2. External engines receive only scoped workspace access through adapters.
3. Skills/MCP servers require source review, pinned versions, least privilege and approval.
4. Trading begins with public data, analysis, backtesting and paper trading only; no live-order authority.
5. File organization begins in preview-only mode, preserves originals, records a reversible journal and requires approval before applying moves/renames.
6. Voice must disclose local versus cloud processing; cloud audio processing requires explicit consent.
7. GitHub Actions do not replace Windows/GPU testing on the physical laptop.

## Work packages
1. Finish file-level Orkas audit: voice/STT/TTS, skills, MCP, project Library, file operations, approvals, Windows packaging.
2. Implement Persian localization on a dedicated branch with complete dictionaries and RTL regression tests.
3. Fork/evaluate NeurAI independently for Persian meeting workflows.
4. Define SCKG schema, provenance, decisions and synchronization boundary.
5. Benchmark Persian STT/TTS alternatives on the target laptop.
6. Prototype a safe file-organizer skill with preview/approve/apply/undo.
7. Evaluate coding/trading adapters independently; do not install or auto-enable them.

## Scope and limitations
This is an initial source-level audit, not a full security audit, runtime benchmark, Windows build verification or legal opinion. No tests are claimed as passed unless executed and recorded. No production source code or user data was changed in this commit.


## 8. Detailed voice, transcription and file-workflow findings

### Speech input is not local in the current Orkas voice-chat path

`src/main/features/voice_input.ts` obtains a short-lived ticket from `https://orkas.ai/v1/audio/transcription_sessions` using the user's Orkas API key, then opens a WebSocket at `wss://orkas.ai/api/voice/asr_ws`. The current voice-input session is capped at 120 seconds. The long-lived API key stays in the main process, which is a good credential boundary, but audio transcription in this path is hosted by Orkas rather than a local Whisper runtime.

### Local Whisper exists, but in the media workflow

`scripts/fetch-whisper.cjs` installs a pinned whisper.cpp CLI and multilingual base-q5_1 model for packaged targets. Its own notice explicitly scopes this runtime to local VideoStudio speech transcription. This is useful evidence that Orkas can run local ASR, but it does not establish a general-purpose, always-available local Persian voice-chat input. Persian accuracy is not benchmarked in the checked-in evidence.

### TTS is pluggable but provider-backed, not local by default

`src/main/features/tts.ts` documents OpenAI-compatible, ElevenLabs and Doubao backends and explicitly states that speech generation does not download or run a local renderer implicitly. `tts_auth.ts` lists Orkas, Doubao, MiniMax, Alibaba Cloud, Google, ElevenLabs, OpenAI and custom OpenAI-compatible providers. The capability layer has locale-aware voice selection, but the checked static model catalog does not establish a verified Persian voice for every provider. Persian TTS must be benchmarked by voice/model and provider; local Audio8 Persian TTS is a separate experimental candidate, not an integrated Orkas backend.

### Document management is not the same as automatic file sorting

Orkas has:
- Scoped file reading, search and grep tools with path sandboxing.
- A global and project Library with extraction, embedding, indexing, reconciliation and semantic retrieval.
- DOCX/XLSX/PPTX creation/review tools and bundled OfficeCLI.
- Skills, marketplace and MCP integration.

The inspected file-tool interface is primarily for reading/searching and scoped work. The Library indexes and retrieves documents; it is not itself a general-purpose policy-driven file organizer. I found no confirmed end-to-end batch workflow that proposes a taxonomy, previews every move/rename, applies it after approval, and provides reliable one-click undo. Therefore the file-organizer requirement remains a genuine gap to implement as a guarded Skill or evaluate separately.

## 9. Refined candidate decisions

| Candidate | Verified repository facts | Decision |
|---|---|---|
| [NeurAI](https://github.com/Dr-Bagheri/neurai-mvp) | Apache-2.0; Persian-first meeting assistant; local faster-whisper, diarization, formal Persian minutes, meeting notepad, action tracking and RAG. README describes v0.3 while architecture file labels itself v0.2, so version/status claims need reconciliation. Optional pyannote diarization is pinned below 4 because its own architecture notes a VRAM regression in 4.x. | **Recommend a separate fork for evaluation**, not merge. Its offline meeting path is unusually aligned with the Persian meeting requirement. Benchmark GPU/RAM use on the HP Victus; keep cloud transcription disabled by default. |
| [Memex](https://github.com/jagdeepvirdi/memex) | MIT; local-first second brain; auto-classifies notes, links, documents and voice memos; Whisper ingestion; RAG, entity graph, Obsidian Markdown import/export, provenance and review workflow. Standard deployment uses Docker Desktop, PostgreSQL/pgvector and Ollama. | **Recommend a separate fork for evaluation** as the closest match to the user's "capture anything, organize it, and find it later" requirement. Evaluate whether its data model can interoperate with SCKG; do not let it replace SCKG. |
| [AI File Butler](https://github.com/ForgeLabsSoft/ai-file-butler) | GPL-3.0; Windows/offline; local Ollama classification, content-aware naming, preview, manual mode, undo and learning from corrections. | **Useful separate evaluation fork only.** Do not copy its code into the MIT Orkas/Shahin Core codebase without a deliberate GPL distribution decision. First compare its safe batch workflow against a new Orkas Skill. |
| [Dastyar AI](https://github.com/momoazn/dastyar-ai) | MIT; Persian-first local assistant with RTL UI, Ollama, Whisper speech-to-text, document Q&A and summaries; Bun monorepo and separate Go whisper.cpp service. README screenshots use English audio and do not prove Persian ASR quality. | Keep as a **reference**, not another fork for now: most of its features overlap with Orkas + NeurAI + Memex. |
| [Persian ASR Compare](https://github.com/erfnzdeh/persian-asr-compare) | MIT; compares Persian Whisper fine-tune against general large-v3; the provided runner is Apple Silicon/MLX-specific and the reported WER is model-card sourced, not measured by this repo. | Do not fork for Windows. Reuse the idea of a repeatable fa benchmark; evaluate the Persian Whisper model separately with a Windows/CUDA-compatible runner. |
| [Audio8 Persian TTS](https://github.com/ard9/Audio8-Persian-TTS) | Apache-2.0 repository; Persian fine-tune of a 0.6B model; author reports intelligible but somewhat robotic prosody and MOS 3.32 ± 0.12 in its stated test. | Do not fork as an application. Evaluate the published model weights and license separately as a local TTS backend candidate. |
| [Khoj](https://github.com/khoj-ai/khoj) | AGPL-3.0; mature self-hostable knowledge assistant, agents and document Q&A. | Keep as a separate-service benchmark only. Do not embed its source into Shahin Core without a license review. |
| [Obsidian](https://obsidian.md/) | The application is proprietary, not an open-source GitHub project to fork. Markdown vaults are portable. | Use Markdown vault import/export and link conventions as interoperability, not as a dependency. |

## 10. Recommended fork queue

Only create these additional forks, each independent and on its own upstream history:
1. `Dr-Bagheri/neurai-mvp` — Persian meeting intelligence evaluation.
2. `jagdeepvirdi/memex` — personal knowledge capture and auto-classification evaluation.
3. `ForgeLabsSoft/ai-file-butler` — optional isolated file-sorting evaluation; GPL-3.0 boundary must remain explicit.

Do not fork Dastyar AI, Khoj, Persian ASR Compare or Audio8 right now. Keep them as references/model candidates until the first three evaluations reveal a concrete gap.

## 11. Hardware and privacy implications

The target laptop has 24 GB system RAM and an RTX 3050 Laptop GPU with 6 GB VRAM. NeurAI's stated baseline (16 GB RAM, 8-core CPU, 4 GB GPU) is nominally within the headline numbers, but actual concurrency, diarization and local LLM model residency can exceed the comfortable VRAM budget. Test one heavy pipeline at a time; do not assume all models can remain resident. Memex's Docker/Postgres/Ollama stack adds background memory use. The local Whisper base model is more plausible for the laptop than simultaneously loading a large-v3 ASR, diarization model and chat model.

For all voice/meeting workflows, the default must be local processing. Any cloud path must be opt-in per task/session with an explicit disclosure of provider, data sent and potential charges. Confidential meeting audio must never silently fall back to cloud.
