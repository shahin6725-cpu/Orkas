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
