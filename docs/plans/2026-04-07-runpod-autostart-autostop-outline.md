# RunPod Autostart / Auto-stop Outline

Date: 2026-04-07

This is a cleaned-up copy of the existing RunPod wake-on-request / stop-when-idle outline that was being used as the planning reference for DreamAI RunPod work.

## Goal

RunPod pods do not scale to zero automatically based on website traffic. The intended architecture is an external controller that:

1. receives incoming requests,
2. ensures the pod is running,
3. waits until the application is healthy,
4. proxies the request to the pod, and
5. stops the pod after an idle timeout.

## Components

### A. GPU pod

The worker pod is expected to:

- run the model server (`FastAPI`, `vLLM`, `ComfyUI`, etc.),
- expose an HTTP port such as `8000`, `7860`, or `8888`, and
- provide a health endpoint such as `GET /healthz`.

### B. Always-on controller

The controller can be:

- the existing web backend,
- a small CPU VM/container, or
- a serverless function paired with durable state storage.

Controller responsibilities:

- start the pod on demand via RunPod,
- poll until the pod is ready,
- proxy the request to the live pod,
- update a last-activity timestamp, and
- coordinate idle shutdown.

### C. Scheduler for idle shutdown

The stop-when-idle loop can be driven by:

- an internal cron job in the controller, or
- an external scheduled job.

## RunPod Control-plane Calls

The original outline pointed at the RunPod pod management docs:

- Start/stop pod API examples: <https://docs.runpod.io/pods/manage-pods>

Operational notes carried forward from that outline:

- pod start is not instant,
- connection details can change after restart, and
- start requests need graceful timeout/error handling.

## Intended Request Flow

1. A request arrives at the stable controller.
2. If the pod is stopped, the controller starts it.
3. The controller polls until the application is healthy.
4. The controller discovers the current pod connection details.
5. The controller proxies the request upstream.
6. The controller records last activity for later idle-stop decisions.

## Upstream URL Handling

The original outline explicitly called out the need to treat the pod upstream URL as dynamic.

Recommended handling:

- fetch current runtime details after start,
- extract the exposed port mapping for the app port,
- construct the upstream base URL from those details,
- cache it only briefly, and
- invalidate and refresh it on proxy connection errors.

## Production Hardening Points From The Original Outline

- guard against concurrent "start storms" with a lock,
- let one request own start/poll while others wait or receive a warming response,
- treat the controller as the stable public entrypoint, and
- keep idle-stop logic separate from request forwarding if that simplifies operations.
