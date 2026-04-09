# DreamAI RunPod State Tracking

Date: 2026-04-07

## Scope

This note tracks the current repository state for the DreamAI-on-RunPod work. It is intentionally conservative: it only records behavior that is already present in repo code or explicitly captured in the existing RunPod planning outline.

## Current Repo State

### What already exists

1. **Custom OpenAI-compatible endpoint support is already in Hermes.**
   - Custom providers are configured via `custom_providers` in `config.yaml`.
   - Custom endpoint credential pools are documented in `website/docs/user-guide/features/credential-pools.md`.
   - Pool selection and persistence live in `agent/credential_pool.py`.

2. **DreamAI RunPod currently fits the existing custom-provider path.**
   - The staged regression coverage in `tests/test_credential_pool.py` uses the concrete provider name `DreamAI RunPod` and pool key `custom:dreamai-runpod`.
   - This means the current repo path for a DreamAI RunPod endpoint is still "custom provider + credential pool", not a dedicated RunPod-specific subsystem.

3. **Recent groundwork now protects against stale cached RunPod pod URLs.**
   - `agent/credential_pool.py` now realigns saved custom-pool entry `base_url` values to the provider's currently configured `base_url` when the pool is loaded.
   - The associated regression test in `tests/test_credential_pool.py` covers the case where an older cached RunPod URL is replaced by a fresh pod URL from config.
   - This is useful for stop/start workflows where a resumed or replacement pod can come back with a different public endpoint.

### What does *not* exist yet

1. **There is no repo implementation yet that starts or stops RunPod pods.**
   - A repository-wide search only shows RunPod references in custom-endpoint docs, the credential-pool code/tests above, and the planning outline.
   - No committed controller, scheduler, or RunPod control-plane client is present.

2. **There is no repo implementation yet for idle tracking / automatic pod shutdown.**
   - The existing auto-stop planning is still captured as design work, not production code.

3. **There is no DreamAI-specific handoff automation yet.**
   - The current handoff surface is still operational notes plus the generic Hermes custom-provider flow.

## Planning Source Already In Repo

The current implementation plan is still the outline in `docs/plans/2026-04-07-runpod-autostart-autostop-outline.md`. That outline describes an external controller that would:

- receive requests,
- start a stopped pod,
- wait for readiness,
- proxy traffic to the active pod,
- track last-used time, and
- stop the pod after an idle timeout.

## Practical Read On The Current State

The safest description of the project today is:

- **Implemented:** custom-provider wiring for DreamAI RunPod endpoints, plus fresh-endpoint recovery for cached credential-pool entries.
- **Planned but not implemented:** a RunPod wake-on-request controller, readiness polling, idle state tracking, and stop-when-idle automation.

## Files To Re-check Before More Implementation Work

- `agent/credential_pool.py`
- `tests/test_credential_pool.py`
- `website/docs/user-guide/features/credential-pools.md`
- `docs/plans/2026-04-07-runpod-autostart-autostop-outline.md`
