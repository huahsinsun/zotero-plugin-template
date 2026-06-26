# Zotero LLM Plugin Plan

## Goal

Turn the current Zotero plugin template into a Zotero 7 paper-reading assistant that can run an OpenAI-compatible model against the active paper context, preserve a per-paper Session, support Canvas-style long-form outputs, and save confirmed Canvas content into Zotero Notes.

The first implementation target is a narrow closed loop: configure a model backend, open the assistant in a Reader context, send selected text plus paper metadata, receive a streamed or non-streamed answer, and restore the same paper Session after switching away and back.

## Scope

In scope:

- Replace template identity, example hooks, menu entries, and preferences with Zotero LLM plugin behavior.
- Add a Reader-side assistant surface with Session history, message input, and mode controls.
- Add model backend configuration for OpenAI-compatible `/v1/chat/completions`.
- Add Context Strategy handling for selected text, surrounding page/paragraph context, and item metadata.
- Add Session persistence keyed by Zotero item identity, shared across windows.
- Add Canvas Block and Canvas Draft data model after the plain chat loop works.
- Add Note Ownership so confirmed Canvas Blocks create or update Zotero child Notes.

Out of scope for the first closed loop:

- Multi-provider model-specific adapters beyond the OpenAI-compatible contract.
- Full RAG/vector indexing of the library.
- Cross-paper research workspace.
- Cloud sync outside Zotero's normal item/note data model.
- Fine-grained permission/security UI beyond local preference storage and clear error states.

## Invariants

- A Session belongs to one paper item and is shared by all windows showing that item.
- Window Runtime State is temporary UI state and must not fork the Session.
- A Canvas Draft belongs to one Canvas Block; a Session may contain multiple drafts.
- The LLM Backend is globally configured and OpenAI-compatible.
- API keys must stay in local Zotero preferences and must not be logged.
- Zotero Note writes only happen after explicit user confirmation.
- Template examples should be removed from startup paths before adding product behavior.
- The first milestone must stay closed-loop and testable before Thinking Mode, Teaching Mode, and Canvas expansion.

## Risks / Unknowns

- Zotero Reader APIs for selection/page context may need concrete probing inside Zotero, not just TypeScript checks.
- Zotero preference storage may not be appropriate for long Session payloads; Session storage may need Zotero item attachments, hidden notes, or local profile storage.
- Streaming responses may be awkward in Zotero's XUL/Firefox 115 environment; non-streaming should be the fallback.
- Multiple windows editing the same Canvas Draft need conflict behavior before Canvas ships.
- Saving Canvas Blocks back into Zotero Notes needs a stable note identity mapping to avoid duplicate notes.
- Current repo is still mostly upstream template code, so removing examples is a prerequisite for reducing blast radius.

## Verification Strategy

Use milestone-specific gates:

- Static build/typecheck after every source milestone: `npm run build`.
- Lint/format after structural cleanup: `npm run lint:check`.
- Startup smoke test after replacing template hooks: `npm test`.
- Manual Zotero run for Reader integration: `npm start`, then verify assistant open/send/restore/save flows in a development profile.
- Targeted unit tests for pure modules: prompt/context assembly, backend request serialization, Session repository behavior, and Canvas-to-note mapping.

## Checklist

- [x] FINISHED Audit current template entrypoints, config, preferences, and glossary.
- [x] FINISHED Record initial domain terms for Session, Window Runtime State, Canvas Block, Canvas Draft, Context Strategy, LLM Backend, and Note Ownership.
- [ ] TODO Rename plugin identity and strip template examples from startup/user-facing UI.
- [ ] TODO Define module boundaries for backend client, context assembly, session storage, reader UI, canvas model, and note writer.
- [ ] TODO Implement model backend preferences and a minimal OpenAI-compatible client.
- [ ] TODO Implement Reader assistant shell with message input and response display.
- [ ] TODO Implement Context Strategy MVP using selected text plus item metadata.
- [ ] TODO Implement per-paper Session repository shared across windows.
- [ ] TODO Close first milestone with a manual Reader chat loop and persistence verification.
- [ ] TODO Add Canvas Block model and per-block Canvas Draft restoration.
- [ ] TODO Add explicit Canvas confirmation flow that creates or updates a Zotero child Note.
- [ ] TODO Add Thinking Mode and Teaching Mode as prompt/mode layers after the base loop is stable.

## Rollback Notes

- Keep template-to-product cleanup separate from feature milestones where possible, so broken Reader behavior can be reverted without restoring template examples.
- Keep backend client and context assembly pure enough that failed UI work does not invalidate tested request construction.
- Gate Note writes behind an explicit command until the Canvas mapping is stable; disabling the command should leave chat and Canvas generation usable.
