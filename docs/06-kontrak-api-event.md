# Kontrak API dan Event

## HTTP conventions
/api/v1; JSON; OpenAPI 3.1; Bearer OIDC access token; X-Request-ID; Idempotency-Key on retryable commands; If-Match/ETag on mutable resources. List {data,next_cursor}. Error: {type,title,status,code,detail,request_id,field_errors}. Timestamp RFC3339 UTC; business date/timezone explicit. Actor/scope never accepted from request body.

## Attendance contract (Phase 2)
- POST /attendance/remote-work-requests — {mode,from_date,to_date,requested_area_label,reason}; reason required.
- GET /attendance/remote-work-requests/me?from=&to=
- GET /attendance/remote-work-requests/pending — manager scoped.
- POST /attendance/remote-work-requests/{id}/decision — {decision,reason}; manager scope, no self approval, reason required.
- POST /attendance/evidence-upload-intents — {mime_type,size_bytes,checksum}; returns {evidence_id,object_key,upload_url,expires_at,required_headers}; authorization and limits apply.
- POST /attendance/evidence-upload-intents/{id}/confirm — verifies private object metadata/ownership.
- POST /attendance/punches — {kind,work_mode,evidence_id,latitude,longitude,accuracy_m,location_captured_at,remote_work_request_id?}; WFO validates trusted request IP against configured office CIDR; WFH/WFA requires approved request covering date. No IP or employee ID accepted from body.
- GET /attendance/me?from=&to=; GET /attendance/team?from=&to=&cursor= manager/HR scope; GET /attendance/punches/{id}/evidence-url short-lived after authz; POST /attendance/adjustments; POST /attendance/adjustments/{id}/approve.

WFO response includes matched WorkLocation and configured kecamatan label, not IP-derived locality. Remote response returns submitted area label and captured coordinates only to authorized readers. Errors include 403 policy, 409 duplicate/state, 413/415 upload limits, 422 location/approval validation, 429 throttling.

## Upload trust sequence
Upload bytes direct from browser to Supabase private bucket via short-lived signed upload URL. API/Vercel must never receive file bytes. Confirm endpoint validates object key namespace, size, MIME, checksum if supported, upload intent expiry and employee ownership. Read evidence only via short-lived signed URL after authorization. Do not expose service role key to client.

## Event envelope — Phase 5 only
```json
{"id":"uuid","type":"employee.assignment.changed","version":1,"occurred_at":"RFC3339","producer":"employee-management","aggregate":{"type":"employee","id":"uuid","version":7},"correlation_id":"uuid","causation_id":null,"data":{},"metadata":{"schema_uri":"..."}}
```
No tokens, photos, storage URLs, raw coordinates, raw IP, private HR/payroll or decision reason. Prefer reference IDs/status codes and authorized API retrieval. Outbox atomic; inbox dedupe; at-least-once delivery; DLQ/replay permissioned and audited. Do not add RabbitMQ before a real subscriber exists.
