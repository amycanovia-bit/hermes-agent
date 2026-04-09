# DreamAI RunPod Action Items / Checklist

Date: 2026-04-07

This checklist follows the repository action-item template in `.plans/action-item-template.md` and only marks work done when it is grounded in current repo code or docs.

| Status | Code area | Supporting docs | Creator | Action item | Notes |
|-------|-----------|-----------------|---------|-------------|-------|
| done | `agent/credential_pool.py` | `website/docs/user-guide/features/credential-pools.md` | amy | Keep DreamAI RunPod on the existing custom-provider path | Current repo state uses `custom_providers` and custom credential pools rather than a dedicated RunPod integration |
| done | `agent/credential_pool.py` | `docs/plans/2026-04-07-dreamai-runpod-state-tracking.md` | amy | Refresh stale saved custom-provider `base_url` values from current config on pool load | Covers the RunPod case where a stopped/restarted pod comes back at a different public URL |
| done | `tests/test_credential_pool.py` | `docs/plans/2026-04-07-dreamai-runpod-handoff.md` | amy | Keep regression coverage for stale DreamAI RunPod URLs | Test uses `DreamAI RunPod` and verifies persisted pool state is updated to the fresh configured URL |
| planned | RunPod controller / gateway layer | `docs/plans/2026-04-07-runpod-autostart-autostop-outline.md` | amy | Add a stable controller that starts pods on demand and proxies traffic to the active pod | Not implemented in repo yet |
| planned | RunPod controller / health checks | `docs/plans/2026-04-07-runpod-autostart-autostop-outline.md` | amy | Poll pod/app readiness before forwarding traffic | Planning outline calls for a health endpoint such as `GET /healthz` |
| planned | Idle-state tracking / scheduler | `docs/plans/2026-04-07-runpod-autostart-autostop-outline.md` | amy | Persist last-activity timestamps and stop pods after the configured idle timeout | No repo implementation yet for stop-when-idle automation |
| planned | Operator docs | `docs/plans/2026-04-07-dreamai-runpod-state-tracking.md` | amy | Document the eventual controller ownership boundary between Hermes custom-provider config and RunPod lifecycle management | Keep this doc conservative until control-plane code exists |
