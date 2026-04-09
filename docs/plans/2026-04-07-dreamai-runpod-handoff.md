# DreamAI RunPod Handoff

Date: 2026-04-07

## Summary

DreamAI RunPod is still represented in-repo as a **custom OpenAI-compatible provider** rather than a dedicated RunPod integration. The main code-level change worth carrying forward is the new protection against stale cached `base_url` values after a pod endpoint changes.

## What Changed Most Recently

- `agent/credential_pool.py`
  - Custom-provider pools now sync stored entry `base_url` values back to the current configured `base_url` during pool load.
  - This avoids reusing an older RunPod URL after the configured endpoint has moved.

- `tests/test_credential_pool.py`
  - Added regression coverage for a `DreamAI RunPod` custom provider whose saved pool entry still points at an old RunPod URL.
  - The expected behavior is that pool selection returns the fresh configured URL and persists that update.

- `website/docs/user-guide/features/credential-pools.md`
  - Custom endpoint docs now call out that Hermes refreshes saved endpoint URLs from current config when loading a custom-provider pool.

## What Is Still Only Planned

The repo still does **not** include the controller described in `docs/plans/2026-04-07-runpod-autostart-autostop-outline.md`.

That outline remains the source for the next implementation phase:

1. start the pod on demand,
2. poll until the app is healthy,
3. proxy requests through a stable controller,
4. track last activity, and
5. stop the pod after an idle timeout.

## Recommended Next Starting Point

If implementation resumes, start from the existing custom-provider path rather than introducing a new abstraction first:

1. keep DreamAI RunPod traffic flowing through the existing custom-provider model config,
2. add a small controller/service that owns RunPod start/stop state,
3. update the configured `base_url` whenever the controller learns a fresh pod endpoint,
4. rely on the current credential-pool base-URL refresh behavior to keep saved entries aligned.

## Risks / Constraints To Keep In Mind

- RunPod pod connection details can change after a restart.
- The repo currently has no persisted idle-state tracker for pod lifecycle management.
- There is no existing scheduler/job wiring for stop-when-idle behavior.
- Documentation should continue to describe the feature as groundwork + planning until control-plane code actually lands.

## Reference Files

- `agent/credential_pool.py`
- `tests/test_credential_pool.py`
- `website/docs/user-guide/features/credential-pools.md`
- `docs/plans/2026-04-07-runpod-autostart-autostop-outline.md`
