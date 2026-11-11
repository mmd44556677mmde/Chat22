# Orion Android Agent — Sequential Build Prompts

## How to use this document

1. Open one chat with the coding AI that will build the project.
2. Paste **Prompt 0** first.
3. Then paste **one numbered step at a time**, in order. Do not combine steps.
4. Let the AI modify the project, run the tests, and report the result before moving on.
5. If a step reports failure, ask it to fix that step and rerun its tests. Do not continue with a broken baseline.
6. The AI must work on the same repository throughout all steps.

---

# Prompt 0 — Master project instructions

You are the senior Android engineer, AI-agent architect, security engineer, QA engineer, and technical writer for a production-grade personal Android agent named **Orion**, created by **Mohammad**.

We will build Orion incrementally in the same repository and conversation. I will give you one step at a time. For every step:

- Work on the actual existing project, not an isolated conceptual example.
- First inspect the current repository and understand what previous steps implemented.
- Preserve all working functionality from earlier steps. Do not replace a working implementation with a simplified mock.
- Implement only the current step, except for small supporting changes required to integrate it correctly. Do not jump ahead.
- Deliver complete, compilable source files and resources. No TODOs, placeholders, pseudo-code, omitted methods, fake integrations, or “implement this later” comments in production code.
- Use Kotlin, Jetpack Compose, coroutines/Flow, Android architecture best practices, dependency injection, Room, DataStore where appropriate, WorkManager for deferrable work, and explicit repository/use-case boundaries. Use stable, mutually compatible dependency versions and a version catalog.
- The app must support a Redmi Note 13 running HyperOS/MIUI while remaining standard Android-compatible. Use Android APIs rather than root, package spoofing, or hidden privileged APIs.
- Keep AI models away from direct Android permissions. Models may propose structured tool calls, but only app-owned orchestration code may validate, authorize, and execute them.
- Never hardcode API keys, model names, base URLs, assistant names, or provider-specific assumptions into business logic.
- Never log secrets, full private conversation contents, screenshots, memory contents, or authorization tokens.
- Network and database operations must be cancellable and lifecycle-aware. Avoid memory leaks and blocking the main thread.
- Every user-visible string must come from resources. Lay out Persian correctly with RTL while keeping the terminal/security visual identity.
- Default identity: assistant name **Orion**, creator **Mohammad**. The assistant name must later be configurable.
- Visual reference: lightweight professional cybersecurity/terminal aesthetic; black/near-black background, phosphor green primary accents, restrained cyan secondary accents, monospaced technical labels, clear Persian UI text, and no visually heavy sci-fi clutter.
- Three key interfaces will eventually exist: a translucent assistant overlay opened by long-pressing power, a full chat screen, and a Live Voice screen with a reactive circular HUD and live transcript.

### Mandatory completion and testing protocol for every step

1. State briefly what you inspected and the exact files/modules you changed.
2. Build the project from a clean state using the relevant Gradle commands.
3. Add meaningful automated tests for the new behavior, including failure and edge cases—not only happy-path tests.
4. Run all existing unit tests and the new unit tests. Run Android lint and relevant static checks. Run instrumentation/Compose tests on an emulator or device if the environment provides one.
5. Use fake providers, fake clocks, in-memory or temporary test databases, and simulated network/device conditions where appropriate. Tests must not require a real paid API key or perform real sensitive phone actions.
6. Actually execute the tests whenever the environment permits. Never claim that a test passed if it was not executed. Clearly distinguish **passed**, **failed**, and **not executed because the environment lacks an emulator/tool**.
7. If a test fails, diagnose and fix the implementation, then rerun the complete affected test suite. Do not hide failures, weaken assertions merely to pass, or delete useful tests.
8. Exercise integration boundaries, not only individual functions. For example, a streaming test must verify UI state and persistence, and a database test must verify actual stored/retrieved values.
9. End with a concise completion report containing:
   - Implemented features
   - Important architecture decisions
   - Files changed
   - Exact commands/tests executed
   - Test results, including counts where available
   - Remaining environment-only verification steps, if any
   - Known limitations, if any
10. Stop after this step and wait for my next prompt. Do not begin the next step.

Acknowledge these rules and inspect/prepare to work in the current repository. Do not start implementing Step 1 until I send it.

---

# Step 1 — Android foundation, architecture, and Orion design system

Continue Orion under the master rules. Implement **Step 1 only**: create or normalize a clean, buildable Android foundation and the reusable visual design system.

## Build requirements

- Configure a standard Android application with Kotlin and Jetpack Compose, using stable compatible versions, Gradle version catalogs, explicit SDK versions, release/debug build types, and a reproducible Gradle wrapper.
- Use a maintainable package/module structure separating `app`, UI, domain, data, AI-provider abstractions, local storage, device tools, and test utilities. Avoid excessive micro-modules; boundaries matter more than module count.
- Add dependency injection, coroutine dispatchers, application-scoped lifecycle handling, structured error types, and safe logging with release-safe redaction.
- Set application identity and default branding to Orion / by Mohammad. Add a proper adaptive launcher icon and splash treatment consistent with the design.
- Create a Compose design system containing:
  - Dark security-terminal color scheme, semantic colors, success/warning/error/offline/connecting states, typography, spacing, shapes, elevation, and motion durations.
  - Phosphor-green primary accent and cyan secondary accent on near-black surfaces.
  - Readable Persian/body typography alongside monospaced technical labels; do not use monospace for long Persian paragraphs.
  - Reusable `OrionTopBar`, status indicator, terminal text, primary/secondary/icon buttons, text input, dialog, snackbar, loading state, empty state, error state, progress card, setting rows, and assistant-state indicator.
  - Light theme support and system dynamic color only if it can be disabled so the Orion identity remains available.
  - RTL-aware layout, content descriptions, touch targets, contrast, reduced-motion handling, and font-scale resilience.
- Create a temporary but polished home/diagnostics screen showing the design components and project health. It is a foundation screen, not the final chat.
- Add a basic navigation host and a settings destination shell so later steps can integrate without rewriting navigation.
- Add baseline unit tests, Compose UI tests, lint configuration, and a debug-only way to inspect build/environment information without exposing sensitive data.

## Required tests

- Verify a clean debug build and test compilation.
- Test application startup and navigation between the temporary home and settings shell.
- Test design-system components in LTR and RTL, including long Persian text and large font scale where the test environment supports it.
- Test semantic status colors/states and accessibility content descriptions.
- Run all tests and lint according to the master protocol. Provide the completion report and stop.

---

# Step 2 — Encrypted local database and user/agent-controlled long-term memory

Implement **Step 2 only**: build Orion’s secure, persistent, inspectable long-term memory foundation. It must be comparable in purpose to an agent’s durable memory, not merely chat history.

## Storage and security requirements

- Store private data in app-private internal Android storage, never public external storage.
- Use Room with a strong maintained database-encryption solution such as SQLCipher for Android. Generate a random database passphrase and protect it with Android Keystore; handle key invalidation without silently deleting data.
- Keep non-secret preferences separately in DataStore. Do not put database keys or API keys in plaintext preferences.
- Define backup behavior deliberately: unencrypted secrets and keys must not be uploaded through Android Auto Backup. Encrypted user-initiated export will be handled later.
- Add database migrations, schema export, transaction handling, foreign keys/indexes, and corruption/key-error reporting that does not leak content.

## Memory model and repository requirements

Create typed long-term memories with at least:

- Stable ID and revision
- Type: profile, preference, personal fact, relationship, goal, project, episode/event, conversation summary, instruction, or custom
- Title/content, normalized searchable text, and optional structured metadata
- Tags, source type, source conversation/message reference when available
- Created/updated/last-accessed timestamps
- Importance, confidence, pinned status, archived status, soft-deletion timestamp
- Access count and optional expiry/review date
- Sensitivity level and origin: user-created, agent-proposed, or agent-created

Implement repository/use-case APIs for create, read, update, delete, restore, archive, pin, search, filter, paginated listing, recent/relevant retrieval, touch/access tracking, bulk delete, and complete memory wipe. Support full-text search with an offline fallback and a clean interface for semantic/vector retrieval later.

## Agent memory access contract

- Define structured tools for `memory_search`, `memory_read`, `memory_add`, `memory_update`, `memory_archive`, `memory_delete`, and `memory_forget_topic`.
- These tools must call app-owned use cases; no model receives raw database access.
- Introduce memory policy controls: read allowed, agent additions allowed, edits require confirmation, deletions require confirmation, and sensitive memories always require confirmation. Sensible secure defaults are mandatory.
- Detect and reject or explicitly warn against storing API keys, passwords, OTPs, private keys, auth tokens, and full payment credentials.
- Add an audit log for memory mutations containing action, memory ID, actor, timestamp, and result without duplicating sensitive content.
- Add an initial Memory management UI where the user can inspect, search, add, edit, pin, archive, restore, and delete memories and view why/when an item was last accessed.

## Required tests

- Real Room integration tests for every CRUD path, migrations, search, pagination, soft delete/restore, filtering, and complete wipe.
- Concurrency tests for simultaneous updates and access-count updates.
- Tests proving policy enforcement for agent reads/writes/deletes and sensitive confirmations.
- Tests for secret-like content rejection/warning and audit behavior.
- Keystore/wrapped-key tests using appropriate fakes/instrumentation; test invalidation/error behavior without pretending real hardware behavior is covered by a fake.
- UI tests for memory CRUD and empty/error states.
- Run the full existing suite, new tests, clean build, and lint. Report and stop.

---

# Step 3 — Multi-provider AI layer with custom Base URL and secure credentials

Implement **Step 3 only**: create the production AI-provider abstraction and its settings UI. Do not build the chat screen yet.

## Provider requirements

- Create a provider-neutral `LlmClient`/gateway supporting non-streaming generation, streaming generation, cancellation, structured tool definitions/calls, usage metadata when supplied, and normalized errors.
- Implement protocol adapters for:
  - OpenAI-compatible chat/completions or the appropriate configurable compatible endpoint
  - Gemini native
  - Anthropic native
- Isolate protocol-specific request/response models. Never leak them into domain code.
- Support independent **Voice** and **Brain** profiles, each with provider, custom Base URL, API key, model name, temperature, max tokens, timeout, retry count, optional custom headers where safe, and optional fallback profile.
- Do not hardcode GLM, Gemini, or any other model. Those are user-selectable examples only.
- Normalize HTTP, TLS, timeout, cancellation, authentication, rate-limit, quota, malformed-response, server, empty-stream, and unsupported-tool/stream errors.
- Use cancellation-aware streaming and never automatically retry a non-idempotent request after partial output unless the design can prevent duplication.
- Store API keys using Android Keystore-backed encryption. Redact authorization headers and keys in logs and error messages.
- Add connection health checks that actually perform a minimal authenticated request and report latency/status without exposing content.
- Support proxy/VPN-aware Android networking without implementing an unsafe global trust manager. Never disable certificate validation.

## Settings UI requirements

- Provider list and add/edit/delete/duplicate profile flows.
- Role assignment for Voice and Brain and optional fallback.
- Base URL, protocol, model, key, generation parameters, and timeout fields with validation.
- Show/hide key input; do not display an existing full key.
- Test Connection with clear progress, success, latency, and actionable error feedback.
- Prevent deleting a profile still in use unless the user selects a replacement.
- A diagnostics view with sanitized recent provider errors and no private prompt/response logging.

## Required tests

- MockWebServer or equivalent contract tests for every protocol: request serialization, headers, streaming chunks, tool calls, malformed JSON, empty deltas, rate limits, timeouts, cancellation, and HTTP errors.
- Verify custom Base URLs are honored and no request goes to a hardcoded vendor URL.
- Verify independent Voice/Brain profile selection and safe fallback selection logic.
- Test secure credential storage with appropriate fakes/instrumentation and redaction unit tests.
- Compose tests for settings validation, connection states, and profile deletion safeguards.
- Run all previous and new tests, clean build, and lint. Report and stop.

---

# Step 4 — Full chat experience, streaming, conversations, and response actions

Implement **Step 4 only**: build the complete text chat and conversation experience using the provider layer and local persistence.

## Chat functionality

- Conversation list plus full chat screen, with new chat, rename, pin, archive, search, delete, and restore behavior where appropriate.
- Persist conversations/messages locally with stable IDs, roles, timestamps, status, model/profile metadata, token/usage metadata when available, and error information safe for display.
- Message states: queued, sending, streaming, complete, cancelled, failed, and retrying.
- Send, stop, retry, regenerate, edit-and-resend, copy, share, and select text. Preserve failed drafts.
- Stream responses incrementally with efficient recomposition and periodic batched persistence so process death does not lose an entire completed response.
- Correctly handle rapid sends, switching conversations during streaming, app backgrounding, rotation/process recreation, cancellation, and provider failure.
- Render Markdown-like content, headings, lists, links, inline code, and scrollable/copyable code blocks safely; do not execute embedded content.
- Include user attachment metadata interfaces but do not implement camera/screen analysis until its later step.
- Empty, loading, offline, error, and no-provider-configured states must be polished and actionable.

## Visual requirements

- Match the reference chat: near-black terminal aesthetic, `operator@phone:~$` and `orion>` accents, secure/link status indicators, restrained green/cyan styling, readable Persian message bubbles or terminal blocks, and a bottom composer.
- Add Live and microphone affordances as disabled/“coming in setup” navigation-safe actions only if their implementation is not yet present; never make a button appear functional when it is not.
- Respect RTL text while keeping code and terminal identifiers LTR.

## Required tests

- Repository tests for all conversation/message operations and process-style restoration.
- Streaming integration tests using fake providers: chunk ordering, duplicate/malformed chunks, cancellation, mid-stream failure, retry, regeneration, and switching screens.
- Verify no duplicate messages after retry or configuration changes.
- Compose tests for send/stop/copy/edit/regenerate states, long messages, code blocks, Persian RTL, and conversation operations.
- Run clean build, entire test suite, and lint. Report and stop.

---

# Step 5 — Default Android assistant and translucent power-button overlay

Implement **Step 5 only**: integrate Orion with Android’s assistant framework and build the reference overlay.

## Assistant integration

- Implement and correctly declare `VoiceInteractionService`, `VoiceInteractionSessionService`, `VoiceInteractionSession`, and any required recognition components using public Android APIs.
- Make Orion appear as a selectable default digital assistant where supported by the OS.
- Add an in-app setup screen that checks whether Orion is the selected assistant, explains the setting, and deep-links the user to the correct system screen.
- Launching via long-press power must open a session overlay rather than forcing the full chat Activity.
- Handle unavailable/unsupported assistant invocation gracefully and provide a normal app entry point.
- Do not use root, Gemini package spoofing, Accessibility interception, or hidden APIs for this feature.

## Overlay UI

- Bottom-sheet style session UI above the current app, with the launcher/current app visible behind it.
- Translucent glass-like dark panel, blur where supported with a performant fallback, subtle green border/glow, Orion orb, waveform, assistant state, and secure-link status.
- Text input, send, microphone/Live entry points, quick actions, and expand-to-full-chat.
- Quick-action examples: summarize this page, analyze this page, and search/research. Unimplemented context actions must clearly request the later capability rather than fake a result.
- Dismiss via swipe/back/outside interaction where appropriate; expand without losing the current draft/session.
- Connect overlay text requests to the existing chat/provider flow and show streaming responses without blocking the UI.
- Include optional Quick Settings Tile as a reliable alternate launcher. Do not add floating bubble yet unless needed for this integration.

## Required tests

- Validate manifest/service declarations and session creation with unit/instrumentation tests possible in the environment.
- Compose tests for overlay states, draft preservation, dismissal, expansion, quick actions, and streaming error states.
- Test default-assistant setup status logic using fakes; do not claim power-button behavior was device-tested unless it actually was.
- Verify normal chat and previous tests still pass.
- Provide exact manual verification instructions for Redmi Note 13 only for behaviors requiring a physical device. Run everything else, report, and stop.

---

# Step 6 — Dual-model Voice/Brain orchestration and durable background jobs

Implement **Step 6 only**: add the real dual-brain agent architecture and background job system.

## Orchestration requirements

- Implement an app-owned Orchestrator separating fast conversational **Voice** work from asynchronous reasoning/research **Brain** work.
- Voice handles short responses, tone, and simple routing. Brain handles research, multi-step reasoning, planning, and long analysis.
- Implement structured Voice tools: `delegate_to_brain`, `check_job_status`, and `cancel_job`, with strict schemas and argument validation.
- Add rule-based routing for explicit research/search requests and ambiguous requests; never rely only on prompt wording for security or correctness.
- Delegation flow: Voice acknowledges briefly, Brain runs independently, the user can continue chatting, and the result returns to Voice for a concise spoken-friendly summary.
- Prevent duplicate delegation, recursive delegation loops, stale job updates, and results being posted into the wrong conversation.

## Job system

- Durable Room-backed jobs with ID, conversation ID, type, input/context reference, status, progress, timestamps, attempt count, cancellation state, sanitized error, result, and source metadata.
- Statuses include queued, running, waiting for network, waiting for confirmation, completed, failed, and cancelled.
- Execute deferrable jobs with WorkManager and immediate active jobs with structured coroutines as appropriate. Restore observable state after process death.
- Expose cancel, retry, details, and progress in chat and a Jobs screen. Brain outputs a typed result containing summary for speech, details, sources, proposed actions, and confidence.
- Treat proposed actions as proposals only. Do not execute device actions in this step.
- Keep Voice and Brain contexts separate and pass only necessary scoped context with explicit size/token budgets.

## Required tests

- Orchestrator tests for direct response, delegation, duplicate suppression, cancellation, stale results, malformed Brain output, provider failure, and conversation isolation.
- WorkManager/Room integration tests for persistence, retry, process-style restoration, and cancellation.
- Compose tests for progress cards, Jobs screen, details, retry/cancel, and continued chat while a job runs.
- Run all suites, clean build, and lint. Report and stop.

---

# Step 7 — Network resilience, offline queue, health recovery, and fallback

Implement **Step 7 only**: make Orion reliable through Wi-Fi/mobile changes, VPN toggling, API outages, and hours of disconnected time without pretending the API is always connected.

## Requirements

- Use `ConnectivityManager` network callbacks plus actual provider health checks. A connected network must not be treated as proof of internet/API reachability.
- Model local assistant availability separately from remote AI connectivity.
- Expose clear states: online, connecting/reconnecting, offline, provider unavailable, and authentication/configuration error.
- Add a durable offline request queue linked to chat/jobs. Preserve user drafts and explicitly label queued work.
- Use exponential backoff with jitter, capped attempts/delays, cancellation, and appropriate retry classification. Do not retry invalid API keys or malformed requests endlessly.
- Re-evaluate requests after network/VPN changes and recreate stale streaming connections safely without duplicated user messages or responses.
- Implement Voice and Brain fallback profiles with cooldown/circuit-breaker behavior, while preventing fallback to an unintended or untrusted provider.
- Add per-provider timeout and retry policies, optional daily request/cost guardrails, and user controls for mobile-data usage and background queued work.
- Local-only commands must be architecturally available without an LLM, but do not implement the complete device-tool suite yet.
- Ensure Orion gives immediate honest feedback rather than silence when offline.

## Required tests

- Simulate network loss before send, during streaming, during Brain work, and before result delivery.
- Simulate VPN endpoint changes, captive/no-internet connectivity, timeouts, 401/403, 429, 5xx, malformed responses, and fallback failure.
- Verify queue durability across repository/job recreation, backoff using a fake clock, retry limits, duplicate prevention, and cancellation.
- Verify UI state and spoken-friendly status messages for every connectivity state.
- Run soak-style repeated reconnect tests with fake networks/providers in addition to the full suite, build, and lint. Report and stop.

---

# Step 8 — Speech input, spoken responses, voices, and conversational controls

Implement **Step 8 only**: add standard push-to-talk voice interaction and high-quality TTS integration, not the full Live Voice experience yet.

## Requirements

- Abstract speech recognition and synthesis behind interfaces, with Android implementations and test fakes.
- Support microphone permission education, request, denial, permanent denial, unavailable recognizer, no speech, partial results, and errors.
- Add push-to-talk in chat and overlay, live partial transcript, edit-before-send option, and direct-send option.
- Support Persian and English recognition/TTS where device engines support them, with language selection and clear unsupported-language feedback.
- Add configurable TTS voice, feminine/masculine presentation where voices are available, speech rate, pitch, and auto-speak setting. Do not claim a voice gender that the engine does not actually provide.
- Keep assistant display name configurable; update branding and conversational references without hardcoded Orion strings.
- Stream response sentences to TTS with a safe sentence segmenter for Persian/English, code/link handling, queue cancellation, and no overlapping playback.
- Add stop speaking, replay, interrupt/barge-in foundation, audio-focus handling, and correct behavior for calls/headphones/other media.
- Persist voice preferences securely but do not retain raw audio by default.

## Required tests

- Fake recognizer/TTS tests for partial/final transcripts, empty speech, permission denial, cancellation, unsupported language, and engine failure.
- Sentence-segmentation tests for Persian punctuation, English punctuation, decimals, ellipses, code, and links.
- Tests for audio-focus loss, interruption, replay, queue cancellation, and switching text-to-speech off mid-response.
- Compose tests for microphone states and voice settings in chat/overlay.
- Run all tests, clean build, and lint; distinguish physical-audio behavior requiring manual device verification. Report and stop.

---

# Step 9 — Live Voice screen, reactive circular HUD, and real-time transcript

Implement **Step 9 only**: complete the Live Voice experience using the speech, chat, orchestration, and connectivity layers.

## Requirements

- Add Live entry points beside chat composer and inside the assistant overlay.
- Build the reference Live screen: black background, secure-link/session timer, large circular HUD with tick marks and green/cyan arcs, central waveform, and visible Listening/Thinking/Speaking/Reconnecting/Error states.
- Animate from real microphone/playback amplitude when available; use subtle idle animation otherwise. Honor reduced-motion settings and avoid high battery usage.
- Maintain a turn-based live transcript that appears during conversation and is saved to the associated chat history with correct ordering and timestamps.
- Implement continuous conversational turns without requiring app reopen, microphone mute/unmute, end session, switch to text chat, and session cleanup.
- Support barge-in: user speech interrupts TTS promptly, stale responses are cancelled or ignored, and the next turn uses correct context.
- Allow Brain research jobs to run during Live without blocking unrelated speech; announce completion briefly and offer details in chat.
- Handle network loss mid-turn with explicit feedback, reconnection, queued/failed transcript state, and optional switch to text.
- Prevent microphone/TTS feedback loops through turn management, echo/headset considerations, and state-machine guards.
- Ensure background/foreground transitions, audio focus, permissions, and process recreation cannot leave phantom recording or duplicate sessions.

## Required tests

- Unit-test the Live state machine exhaustively, including invalid transitions, rapid mute/end/restart, barge-in, stale events, and reconnect.
- Integration-test transcript persistence and Brain-job announcements with fakes.
- Test amplitude-to-HUD state mapping and reduced-motion behavior.
- Compose tests for all controls, timer, status, transcript, errors, and switch-to-chat.
- Run repeated lifecycle tests and all previous suites, build, and lint. List microphone/audio behaviors requiring physical verification honestly. Report and stop.

---

# Step 10 — Screen context, screenshots, camera/gallery, and multimodal requests

Implement **Step 10 only**: add privacy-controlled visual and screen understanding.

## Requirements

- Use `onHandleAssist`/`AssistStructure` where available to obtain visible text and assistant context, with graceful fallback when an app blocks it or content is unavailable.
- Add explicit screenshot capture using supported Android APIs and user consent. Never silently capture persistent screen content.
- Add camera and gallery attachment flows, image preview/removal, size/dimension validation, compression, orientation handling, and safe temporary-file cleanup.
- Build provider-neutral multimodal message parts and implement image support only for compatible configured providers; reject unsupported profiles clearly rather than dropping images.
- Add quick actions: summarize page, translate page, analyze image/page, and draft a reply. Include selected-text/selected-region extensibility without pretending every app supports it.
- Show exactly what will be sent before sensitive screenshot/context submission, with per-request consent and a privacy-mode control.
- Redact or exclude passwords and sensitive fields when detectable. Do not bypass secure windows or app screenshot protections.
- Keep captured data local by default, restrict retention, and delete temporary captures after use according to settings.

## Required tests

- Test AssistStructure parsing, empty/blocked context, sensitive-node exclusion, and context-size limits.
- Test image rotation/compression/validation and temporary-file cleanup using sample assets.
- Contract-test multimodal serialization for each supported protocol and unsupported-provider errors.
- Test consent flow, privacy mode, cancellation, upload failure, and no-capture-without-permission behavior.
- Compose-test attachments and quick actions in chat and overlay. Run full suite, build, lint, and report only genuine device coverage. Stop.

---

# Step 11 — Secure device-tool framework and core phone controls

Implement **Step 11 only**: give Orion validated, permission-safe control over common phone functions through app-owned tools.

## Tool framework

- Define typed tool descriptors, JSON schemas, argument parsers, permission/capability requirements, risk level, confirmation policy, execution result, and audit metadata.
- Add an Orchestrator-owned executor that allowlists tools, validates all arguments, checks permissions, obtains required user confirmation, times out safely, and returns structured results.
- Models can propose calls but never execute Android APIs directly. Web/model content cannot override confirmation requirements.
- Show a clear confirmation sheet describing the exact action and target. Sensitive actions require fresh confirmation every time by default.

## Core tools

Implement where Android permits, with graceful unsupported states:

- Open installed apps and search apps by label.
- Create alarms, timers, and reminders.
- Search contacts and initiate calls; sending SMS/calling requires confirmation and correct permissions/default-role constraints.
- Create/search calendar events with confirmation for modifications.
- Open maps/navigation queries.
- Control flashlight, brightness, volume, and media where public APIs/permissions allow.
- Wi-Fi/Bluetooth/Do-Not-Disturb: use system settings panels or public APIs rather than unsupported hidden toggles; explain when user action is required.
- Read/write clipboard only within Android background-access restrictions and with privacy controls.
- Add deterministic local parsing for simple offline commands such as timer, alarm, open app, and flashlight; if ambiguous, ask rather than guessing.

## Required tests

- Tool-schema and argument-validation tests, unknown-tool rejection, timeout, permission denial, confirmation denial/expiry, and duplicate-call prevention.
- Fake Android gateway tests for every tool, including unsupported API/device states.
- Tests proving sensitive actions never execute without explicit confirmation and audit records contain no message contents/secrets.
- Offline parser tests in Persian and English, including ambiguous times and app names.
- Compose tests for confirmation sheets and actionable permission/settings flows. Run full regression, build, lint, report, and stop.

---

# Step 12 — Notifications, Accessibility automation, routines, and advanced controls

Implement **Step 12 only**: add advanced integrations with strict user control and accessibility compliance.

## Requirements

- Notification access: setup/explanation, per-app allowlist, read/summarize active notifications, sensitive-content exclusion, and clear disabled states using `NotificationListenerService`.
- Accessibility automation: an optional, clearly disclosed service for user-requested click/type/scroll/open actions. Do not use it to bypass security, capture passwords, perform hidden surveillance, or imitate assistant launching.
- Accessibility actions must use visible task progress, allow immediate stop, constrain scope to user-approved target apps where possible, and require confirmation for consequential actions.
- Clipboard and media controls must integrate with the tool policy from Step 11.
- Add user-defined shortcuts/compound commands and scheduled/event-style Routines where Android permits. Example: Sleep mode can set an alarm, open DND settings or toggle it with granted access, and stop media.
- Routine editor: name, trigger/schedule, ordered actions, enabled state, next run, run history, validation, test-run with confirmations, pause, edit, and delete.
- Handle timezone, clock changes, reboot rescheduling, missed runs, duplicate WorkManager jobs, and device idle constraints.
- Add a central Permissions & Capabilities screen showing granted/missing/restricted status and exact purpose for every sensitive integration.

## Required tests

- Notification filtering/allowlist/redaction tests and listener lifecycle tests with fakes.
- Accessibility action validation, stop/cancellation, scope restrictions, and sensitive-field refusal tests.
- Routine scheduling tests with fake clock/timezone, reboot restoration, missed-run policy, deduplication, and disabled routines.
- Verify no routine bypasses action confirmations or tool policies.
- Compose tests for routine editor and capability screen. Run all tests, clean build, lint, report manual privileged-permission checks separately, and stop.

---

# Step 13 — Offline wake word, foreground service, boot recovery, and HyperOS resilience

Implement **Step 13 only**: make local activation reliable without keeping a permanent AI connection.

## Requirements

- Abstract wake-word engines and integrate an offline engine such as Porcupine or openWakeWord only through legally/technically compatible dependencies and user-supplied/custom model assets where required. Provide a working fallback push-to-talk path if no wake-word model is available.
- Default wake phrase follows the configurable assistant name. Explain and handle engines that require generating a new custom keyword model after renaming; never pretend arbitrary renamed wake words work automatically.
- Foreground microphone service with the correct service type, persistent notification, visible recording state, stop/pause action, and no hidden microphone use.
- Wake word opens the assistant overlay/session and starts recognition, subject to Android background-start and microphone restrictions.
- Implement boot/user-unlocked handling honestly: restart only what Android allows, and show a one-tap reactivation notification when user action is required.
- Add HyperOS/MIUI setup guidance for Autostart, No battery restrictions, and Recents locking, while retaining standard Android settings fallbacks.
- Include battery-saving modes, wake-word sensitivity, listening schedule if feasible, and pause when privacy mode is enabled.
- No raw audio retention by default and no remote API request merely because the wake-word engine is running.

## Required tests

- Fake wake-word engine tests for detection, duplicate events, sensitivity changes, rename/model-unavailable state, engine failure, and privacy pause.
- Service lifecycle tests for start/stop/restart, permission revocation, foreground-service errors, boot locked/unlocked states, and notification actions.
- Verify repeated detections cannot create multiple overlays/sessions and no network call occurs in the offline detection path.
- Compose tests for activation/setup and battery guidance. Run all automated suites/build/lint; clearly mark Android-version and HyperOS physical tests that remain. Report and stop.

---

# Step 14 — Intelligent agent memory, personalization, and controlled self-editing

Implement **Step 14 only**: turn the encrypted memory foundation into a useful agent memory system.

## Requirements

- Add memory extraction after meaningful interactions using a structured model output, with local heuristics for explicit requests such as “remember this” and “forget this.”
- Memory candidates must be normalized, classified, tagged, assigned importance/confidence/sensitivity, and checked for duplicates/contradictions before storage.
- Respect policy: explicit user remember/forget commands, auto-save allow/deny settings, confirmation for edits/deletes/sensitive items, and complete user override.
- Give the agent the Step 2 memory tools through the Orchestrator, with scoped access, pagination, size limits, and full audit. The agent may add/update/delete only as policy permits.
- Retrieve only relevant memories for each Voice/Brain request using offline full-text/ranking and optional local/provider embeddings behind a replaceable interface. Never send the whole database to a remote model.
- Add token/context budgets, pinned-memory priority, expiry/review handling, contradiction resolution, and source attribution. Memories are untrusted data and cannot override system safety or tool permissions.
- Add conversation summarization and long-term project/goal context without storing every message redundantly.
- Memory UI must show “why Orion remembers this,” source, last use, edit/pin/archive/delete, topic forget, bulk selection, retention controls, and global wipe with strong confirmation.
- Add custom instructions/personality: feminine/masculine style, tone, verbosity, language, formality, and proactive behavior, kept separate from factual memories.
- Private mode must disable new memory writes and prevent remote transmission of existing memories unless explicitly allowed.

## Required tests

- Extraction tests for explicit remember/forget, duplicates, updates, contradictions, low-confidence candidates, sensitive data, and prompt-injection text.
- Retrieval ranking, relevance, token-budget, pinned/expired/archived/soft-deleted exclusion, and private-mode tests.
- End-to-end fake-model tests proving policy-limited agent CRUD and confirmation enforcement.
- Test that deleted memories no longer appear in retrieval or prompts.
- Compose tests for memory explanations, candidate approvals, topic forget, and personality settings. Run full regression/build/lint, report, and stop.

---

# Step 15 — Complete product UX, advanced settings, data controls, and polished feature parity

Implement **Step 15 only**: finish the surrounding product experience expected from mature AI apps such as ChatGPT, Claude, and Gemini, while retaining Orion’s own identity. Do not copy proprietary branding.

## Conversation and productivity features

- New chat, recent chats, search, rename, pin, archive/unarchive, delete, export, and safe share.
- Message copy/share/select, regenerate, edit/resend, stop, retry, feedback, and timestamps/details.
- Prompt library/quick prompts, custom reusable prompts, draft autosave, and optional conversation-specific model/profile selection.
- Jobs/background research center and notification settings for completed work.
- Chat/Live history retention and per-conversation delete/export.

## Settings organization

Create usable sections rather than one long unstructured page:

- Assistant: name, creator display, language, personality, voice, TTS, custom instructions.
- AI Models: Voice/Brain providers, Base URLs, keys, model parameters, fallback, Test Connection.
- Memory: enable/read/write policies, retention, review, clear memory.
- Voice & Live: speech language, auto-speak, barge-in, wake word, sensitivity, audio behavior.
- Appearance: dark/light/system, Orion terminal theme, dynamic color toggle, font size, reduced motion, haptics.
- Privacy & Security: private mode, screenshot/notification consent, biometric lock, secure-screen option, diagnostics/log controls.
- Device Tools & Permissions: capability toggles, confirmations, Accessibility, notifications, calendar/contacts/phone.
- Network & Storage: mobile data, offline queue, retry behavior, cache/temp cleanup, storage usage, media retention.
- Notifications: job completion, service status, reconnect and alerts.
- Advanced/Diagnostics: sanitized provider status, export diagnostics with explicit review, reset settings.
- About: Orion, by Mohammad, version, licenses, privacy documentation.

## Data features

- Encrypted user-initiated backup/export and restore for settings, conversations, memories, and routines, with versioned schema, passphrase handling, integrity checks, and no plaintext API keys.
- Selective export where practical, clear import conflict behavior, and full app-data deletion with a summary of consequences.
- All destructive operations need confirmation and accessible undo where safely possible.

## Required tests

- Settings repository/DataStore tests, defaults, reset, migration, and independence of Voice/Brain profiles.
- Conversation operations/export/share tests and prompt-library CRUD tests.
- Encrypted export/restore round-trip, wrong passphrase, tampering, old/new schema, partial failure, and no-key-leak tests.
- Compose/navigation tests for every settings section, RTL, large fonts, destructive confirmations, and private mode.
- Run complete regression, clean build, lint, accessibility checks available in the environment, report, and stop.

---

# Step 16 — Security, privacy, and adversarial hardening

Implement **Step 16 only**: perform a real security review, fix findings, and add regression tests. Do not merely write a checklist.

## Review and hardening scope

- Threat-model providers, prompt injection, malicious web/page content, screenshots, notifications, memory poisoning, tool arguments, exported backups, logs, backups, IPC/deep links, and Accessibility actions.
- Verify exported Android components have the minimum necessary exposure and permissions; assistant/services cannot be abused by other apps to read memory, messages, keys, or invoke tools.
- Enforce certificate validation, safe URL handling, no cleartext traffic by default, no arbitrary header leakage across redirects/hosts, and protection against credential exfiltration through malicious Base URLs/redirects.
- Confirm Keystore-backed keys, encrypted database, safe Auto Backup exclusions, key invalidation behavior, biometric lock, and optional `FLAG_SECURE` behavior.
- Treat all model/web/screen/memory text as untrusted data. Add tests where malicious text instructs the agent to reveal secrets, change policies, send messages, call contacts, or delete memory; every attempt must fail policy enforcement.
- Add confirmation binding so approval applies to the exact displayed action/arguments and expires; changing arguments invalidates approval.
- Review logs, crash reports, exceptions, notifications, app switcher previews, shares, diagnostics, and accessibility output for sensitive-data leakage.
- Check file provider paths, exported media, temporary files, backup integrity, dependency vulnerabilities available through local tooling, and release minification/resource shrinking without breaking provider serialization.

## Required tests

- Add adversarial and authorization regression tests for every finding.
- Test cross-app/component exposure where instrumentation permits, unauthenticated intents, malformed URIs, replayed confirmations, and permission revocation.
- Run full unit/instrumentation/static/lint suites and a clean release build. Produce a prioritized security report: fixed issues, accepted limitations, and physical-device-only checks. Do not hide unresolved high-risk findings. Stop.

---

# Step 17 — End-to-end QA, performance, compatibility, and failure recovery

Implement **Step 17 only**: stabilize the whole product through integration, lifecycle, performance, and recovery testing. Fix defects rather than adding major new features.

## End-to-end scenarios

Automate with fakes/fake providers wherever physical Android services are unavailable:

1. Fresh install → configure Voice/Brain custom endpoints → test connections → send streaming chat.
2. Select assistant → open overlay → submit text → expand to chat without losing state.
3. Push-to-talk → response TTS → Live session → barge-in → transcript persisted.
4. Delegate research → continue another conversation → receive correct job result → cancel/retry jobs.
5. Go offline for hours using fake clock → queue work → change network/VPN → reconnect once without duplicates.
6. Remember a preference → verify retrieval in a later conversation → edit it → forget topic → verify it is absent from prompts.
7. Screen/image consent → multimodal request → cleanup temporary data.
8. Device action request → permission denial → retry → exact confirmation → execution/audit.
9. Routine creation → reboot/timezone simulation → exactly-once intended behavior.
10. Encrypted export → clear data → restore → verify conversations, memories, routines, and settings without restoring plaintext keys.
11. Process death/recreation during streaming, Live, job execution, editing, and queued work.
12. Permission revocation, provider key expiration, database/key errors, low storage, and malformed provider responses.

## Quality and performance

- Detect main-thread disk/network I/O, leaks, runaway coroutines, duplicate collectors, unbounded cache, excessive recompositions, and memory growth during long chats/Live sessions.
- Measure cold/warm startup, initial chat render, streaming update efficiency, database query behavior, and foreground/wake-word battery risks where tooling permits.
- Test Redmi Note 13-class screen sizes, small/large screens, RTL Persian, English, font scaling, dark/light themes, rotation, and background/foreground cycles.
- Ensure accessibility labels, focus order, contrast, touch targets, reduced motion, and TalkBack-critical flows are sane.

## Required results

- Run every automated suite, stress/repeated lifecycle tests, clean debug/release builds, lint, and static checks.
- Provide a test matrix with passed/failed/not-executed and reasons. Fix all reproducible failures and rerun affected plus full regression suites.
- Provide an honest physical-device checklist limited to what automation cannot prove: actual power invocation, HyperOS process killing, real microphone echo, Android engine voices, OEM settings, and real provider latency.
- Stop after stabilization; do not prepare final release documentation until the next step.

---

# Step 18 — Final release audit, installable artifacts, and complete handoff

Implement **Step 18 only**: prepare Orion for a clean handoff and release without changing its feature scope.

## Requirements

- Audit the entire repository against all prior steps. Identify missing or fake functionality and fix it before calling the release complete.
- Ensure no TODO/placeholder production implementations, hardcoded secrets, test API keys, debug logging of private data, dead demo buttons, or sample provider responses remain.
- Verify version name/code, application ID, adaptive icons, app name, licenses, privacy policy, permission explanations, release signing guidance, and secure debug/release separation.
- Produce clean debug and release artifacts supported by the environment. Do not invent a signing key or embed one in the repository. Explain exactly how the user signs the release locally.
- Write concise but complete documentation:
  - Product overview and architecture diagram
  - Setup/build requirements and exact commands
  - Voice/Brain custom Base URL and API-key configuration
  - Default assistant and power-button setup on Redmi Note 13/HyperOS
  - Wake-word model setup and Android/OEM limitations
  - Memory controls, private mode, backup/restore, and data deletion
  - Permissions/capabilities and sensitive-action confirmations
  - Testing guide and physical-device verification checklist
  - Troubleshooting for API, VPN/network, microphone, battery, boot, and provider errors
  - Security/privacy limitations and third-party provider warnings
- Add a machine-readable project/file map and a short maintenance guide so another AI or developer can safely continue later.

## Final verification and report

- Start from a clean checkout/state, install dependencies using locked versions, run the full build, unit tests, instrumentation tests available, lint/static checks, and release build.
- Record artifact paths, checksums, version information, test totals, and unresolved environment-only checks.
- Do not state “production ready” if critical tests failed or core functionality is mocked. Give an explicit readiness assessment and remaining risks.
- Deliver the final repository state, documentation, artifact locations, and a concise completion summary. Stop and wait for further instructions.
