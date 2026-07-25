---
name: Manage the firearm alert lifecycle
description: Configure firearm-detection alerts, investigate an alert event with its AI description and video clip, and submit feedback.
api: openapi/coram-ai-openapi-original.json
operations: [create_firearm_alert, update_firearm_alert, delete_firearm_alert, get_alert_details, get_alert_clip, get_firearm_alert_feedback, submit_firearm_alert_feedback]
---

# Manage the firearm alert lifecycle

Set up firearm-detection alerting and triage alert events. Requires an API key (`X-Auth-Token`) with `write:*` scope and the `admin` role for create/update/delete; reads need `read:*`.

## Configure

1. **Create the alert.** `create_firearm_alert` (POST `/v1/alerts/firearm`) with monitored cameras, recipients (validated emails / push levels), stance type, sensitivity, and schedule. Handle `INVALID_RECIPIENT_EMAILS` and `PREMIUM_ALERT_CAMERA_LIMIT_EXCEEDED`.
2. **Adjust or remove.** `update_firearm_alert` (PATCH `/v1/alerts/firearm/{firearm_alert_id}`) or `delete_firearm_alert` (DELETE `/v1/alerts/firearm/{firearm_alert_id}`). `FIREARM_ALERT_NOT_FOUND` if the id is stale.

## Investigate an alert event

3. **Get details.** `get_alert_details` (GET `/v1/alerts/{alert_event_id}/details`) — includes the AI description (poll if `AIDescriptionStatus` is still processing).
4. **Get the clip.** `get_alert_clip` (GET `/v1/alerts/{alert_event_id}/clip`) — the clip may still be generating (`ClipStatus`); retry, or handle `ALERT_NO_VIDEO_AVAILABLE` / `ALERT_CLIP_GENERATION_FAILED`.
5. **Review / submit feedback.** `get_firearm_alert_feedback` (GET) then `submit_firearm_alert_feedback` (PUT `/v1/alerts/{alert_event_id}/feedback`) with the outcome reason.

## Rules

- Feedback is idempotent (last-write-wins): re-submitting replaces the previous feedback — safe to retry.
- Only firearm alert events accept feedback; a non-firearm event returns `ALERT_NOT_FIREARM`.
- See `conventions/coram-ai-conventions.yml` and `errors/coram-ai-error-codes.yml`.
