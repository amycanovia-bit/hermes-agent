# Telegram document attachments follow-up

## Status
Open - important, not urgent.

## Context
A recent Hermes fix restored `MEDIA:` parsing for document attachments such as `.md` and `.txt` without touching the true-Codex request normalization / instruction-slimming changes that fixed the earlier empty-output regression.

That parser fix appears necessary, but a read-only follow-up assessment found additional residual risk in the Telegram attachment flow.

## Assessment summary
### Confirmed / high-confidence issue
1. **Attachment delivery logic is still split across multiple parsers**
   - `gateway/platforms/base.py` has the improved `extract_media()` path.
   - `gateway/run.py` still appears to contain two older manual `MEDIA:(\S+)` regex paths used in tool-result / final-response handling.
   - This creates a likely failure mode where tool-generated attachments still break even after the parser fix, especially for quoted paths, backticked paths, or paths with spaces.

### Plausible remaining risks
2. **Document paths with spaces may still be fragile**
   - One parser branch appears to handle paths-with-spaces more generously for image/audio/video types than for document types such as `.pdf`, `.docx`, `.xlsx`, `.pptx`.
   - Example likely-bad case: `MEDIA:/tmp/my report.pdf` if unquoted.

3. **Telegram attachment sends may not have the same stale-thread fallback as text sends**
   - Text sends appear to retry without `message_thread_id` in stale-thread cases.
   - Attachment/document sends may not get equivalent retry handling.
   - Result: text may send successfully while an attachment fails or degrades to a plain file-path message.

4. **Inbound non-text documents appear more transport-capable than analysis-capable**
   - `.md` / `.txt` content appears easier to inject directly.
   - PDFs / Office documents appear more likely to be stored and surfaced as paths/context notes than made directly readable to the agent.
   - This is more of a product/usability limitation than a transport bug, but it should be tracked explicitly.

5. **Inbound document type support may be narrower than user expectations**
   - Common files like `.csv` or `.json` may still be rejected depending on the whitelist.
   - This is lower severity, but worth clarifying or widening later.

## Recommended action item
### Action item: harden Telegram document attachment flow without touching Codex empty-output fixes
**Priority:** Medium

**Goal:** Improve Telegram document attachment reliability while preserving the true-Codex request-shaping and instruction-slimming fixes that resolved the previous `completed + output=[]` regression.

**Implementation direction:**
1. Unify all `MEDIA:` extraction onto the shared parser in `BasePlatformAdapter.extract_media()`.
2. Remove or replace stale ad-hoc `MEDIA:(\S+)` parsing in `gateway/run.py`.
3. Ensure document paths with spaces are supported as reliably as image/audio/video paths.
4. Add Telegram attachment send fallback behavior comparable to text-send stale-thread handling.
5. Decide whether inbound document handling is intentionally transport-only or should support richer analysis for PDFs / Office files.

## Acceptance criteria for the future fix
- Tool-generated attachments and assistant-emitted `MEDIA:` attachments both use the same extraction logic.
- `.md`, `.txt`, `.pdf`, `.docx`, `.xlsx`, `.pptx` paths are handled correctly, including quoted paths and paths containing spaces.
- Telegram document sends do not silently fail in stale-thread cases where text sends would recover.
- No changes are made to true-Codex request normalization, `tool_choice` / `parallel_tool_calls` cleanup, or instruction-slimming logic unless separately justified.

## Guardrail
Do **not** solve this by broadening Codex instructions again without evidence. The current best hypothesis is a gateway / delivery-path issue, not a reversion of the original empty-output bug.

## Source
Derived from a read-only coding-agent assessment of the Hermes repo following the document-attachment parser fix.
