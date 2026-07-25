---
name: Respond to an access-control event
description: List access-control doors and events, pull the video clip for an event, and remotely unlock a door.
api: openapi/coram-ai-openapi-original.json
operations: [list_access_control_doors, list_access_control_events, GetEventClip, unlock_access_control_door]
---

# Respond to an access-control event

Investigate a door event and, when warranted, trigger a remote unlock. Requires an API key (`X-Auth-Token`); unlock needs `write:*` scope, the `admin` role, and per-door permission — a key whose creator cannot unlock the door in the app cannot unlock it here.

## Steps

1. **List doors.** `list_access_control_doors` (GET `/v1/access-control/doors`) to resolve door ids and names.
2. **Query events.** `list_access_control_events` (GET `/v1/access-control/events`) filtered by door/time (card scans, REX, forced-open / held-open, battery alerts). Handle `Invalid filter values` (400). Paginate on `has_more`.
3. **Pull the clip.** `GetEventClip` (GET `/v1/access-control/events/{event_id}/clip`) for the MP4 recorded by the door's primary camera. Handle `ACCESS_EVENT_NO_VIDEO_AVAILABLE` / `ACCESS_EVENT_CLIP_GENERATION_FAILED`.
4. **Unlock if needed.** `unlock_access_control_door` (POST `/v1/access-control/doors/{door_id}/unlock`) for a momentary remote unlock. A `403` means the key lacks scope/role, the door permission, or the feature is off for the org.

## Rules

- Unlock is a safety-critical write — confirm intent before calling; see `agentic-access/coram-ai-agentic-access.yml` (human-in-the-loop required).
- Retry `429`/`5xx` with backoff. See `conventions/coram-ai-conventions.yml` and `errors/coram-ai-error-codes.yml`.
