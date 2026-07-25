---
name: Onboard Coram camera infrastructure
description: Register NVRs and cameras for a site, verify NVR connectivity, and organize cameras into locations and groups.
api: openapi/coram-ai-openapi-original.json
operations: [create_locations, register_nvrs, ping_nvr, speedtest_nvr, list_nvrs, register_cameras, list_cameras, create_camera_group, move_cameras]
---

# Onboard Coram camera infrastructure

Bring a new site online in Coram: create its location, register the recorder, register cameras, and group them. Requires an API key (header `X-Auth-Token`) whose creator has `write:*` scope and the `admin` role.

## Steps

1. **Create the site.** `create_locations` (POST `/v1/locations`) with the facility name(s). Capture the returned location id.
2. **Register the recorder.** `register_nvrs` (POST `/v1/nvrs`) with the NVR name, credentials, and the location id from step 1.
3. **Verify connectivity.** `ping_nvr` (POST `/v1/nvrs/ping`) and optionally `speedtest_nvr` (POST `/v1/nvrs/speedtest`) to confirm the NVR is reachable and has adequate uplink. Retry on `PING_REQUEST_TIMEOUT` / `SPEEDTEST_REQUEST_TIMEOUT`.
4. **Confirm registration.** `list_nvrs` (GET `/v1/nvrs`) — page with `has_more` until the new NVR appears.
5. **Register cameras.** `register_cameras` (POST `/v1/cameras`) with MAC/IP and the target NVR. This is a batch operation: check the top-level `status`, then each `results[].status`; handle per-item `INVALID_MAC_ADDRESS`, `CAMERA_ALREADY_EXISTS`, `NO_LICENSES_AVAILABLE`.
6. **Organize.** `create_camera_group` (POST `/v1/camera-groups`) to make a group, then `move_cameras` (POST `/v1/cameras/move`) to assign cameras. Confirm with `list_cameras` (GET `/v1/cameras`).

## Rules

- Batch endpoints report an overall `status` plus a per-item `results[].status`; never assume all-or-nothing.
- Paginate every list via the `has_more` flag (page size 100).
- Retry `429` and `5xx` with exponential backoff. See `conventions/coram-ai-conventions.yml` and `errors/coram-ai-error-codes.yml`.
