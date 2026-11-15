# ERD Global dan Ownership

```mermaid
erDiagram
 Organization ||--o{ OrgUnit : contains
 OrgUnit ||--o{ OrgUnit : parent
 Organization ||--o{ WorkLocation : has
 WorkLocation ||--o{ OfficeNetwork : validates
 Principal ||--o{ RoleAssignment : receives
 Employee ||--o{ Employment : has
 Employment ||--o{ EmploymentAssignment : assigned
 Employee ||--o| DirectoryProfile : projects
 Employee ||--o{ RemoteWorkRequest : requests
 RemoteWorkRequest ||--o{ RemoteWorkDecision : decisions
 Employee ||--o{ AttendancePunch : records
 AttendancePunch ||--|| AttendanceEvidence : requires
 WorkLocation ||--o{ AttendancePunch : identifies
 Employee ||--o{ LeaveRequest : requests
 LeaveRequest ||--o{ LeaveApproval : approvals
 OutboxEvent ||--o{ InboxMessage : delivers
 AuditRecord }o--|| Principal : actor
```

## Attendance logical entities
- WorkLocation(id, org_id, name, address_label, kecamatan_code, latitude, longitude, timezone, geofence_radius_m, active). Kecamatan/work location disetel sebagai master demo; jangan geocode/IP lookup otomatis.
- OfficeNetwork(id, work_location_id, cidr, label, active, valid_from, valid_to). CIDR hanya egress IP publik kantor; unique/conflict checks dan audit perubahan. Jangan simpan data router rahasia.
- RemoteWorkRequest(id, employee_id, mode WFH|WFA, from_date, to_date, requested_area_label, reason, status, submitted_at, version). Alasan wajib dan tidak boleh disimpan di log.
- RemoteWorkDecision(id, request_id, manager_id, decision, reason, decided_at). Decision reason wajib, approver berbeda dari requester, manager scope enforced.
- AttendancePunch(id, employee_id, kind CHECK_IN|CHECK_OUT, occurred_at, business_date, timezone, work_mode WFO|WFH|WFA, source WEB, work_location_id nullable, remote_request_id nullable, office_network_id nullable, network_match boolean, latitude/longitude, accuracy_m, location_captured_at, idempotency_key, status, version). Hindari menyimpan raw client-supplied IP bila tidak diperlukan; simpan network match/config reference dan audit metadata minimal.
- AttendanceEvidence(id, punch_id unique, storage_bucket, object_key unique, mime_type, size_bytes, checksum, upload_status, captured_at, uploaded_at, verified_at, retention_until). Private bucket; signed URL bukan disimpan. Constraint ukuran/format/status dan relasi ke upload intent.

## Ownership dan integritas
Organization owns WorkLocation/OfficeNetwork configuration. Employee owns employment identity. Attendance owns remote requests, decisions, punches, upload intents and attendance history. IAM owns principal/grants. Audit platform owns redacted audit trail. Domain references lintas context memakai opaque UUID, tanpa FK/query lintas konteks.

## Invariants
WFO must match active office CIDR to active WorkLocation; remote must reference approved request covering local business date; punch requires verified evidence and location capture. Unique employee+business_date+kind (atau state machine constraint yang eksplisit) prevents duplicates; CHECK_OUT requires valid prior CHECK_IN. Use transaction/locking to prevent races. No cascade-delete history; correction is append-only adjustment.


## Entitas policy presensi
- AttendancePolicy(id, org_id, work_location_id nullable, name, status).
- AttendancePolicyVersion(id, policy_id, version, effective_from, effective_to, gps_radius_m, gps_max_accuracy_m, shift_config, timezone, work_calendar_id, evidence_retention_days, exception_rules, change_reason, changed_by, approved_by nullable, created_at). Unique(policy_id,version); cegah periode overlap pada scope sama.
- AttendancePunch menyimpan policy_version_id. Radius dipindahkan dari WorkLocation ke versi policy; policy owner tetap Attendance. CIDR kantor dikelola admin dengan tanggal efektif dan audit.

Versi immutable setelah dipakai. Histori audit menyimpan actor, alasan, waktu, perubahan sebelum/sesudah, tanpa mencatat secret.

## RBAC role/permission seed
Permission seeds:
- Employee: attendance.self.read, attendance.remote_work.create, attendance.punch.create, attendance.evidence.read.self.
- Manager: attendance.team.read, attendance.remote_work.review (team scope), attendance.adjustment.review (team scope).
- Attendance Policy Admin: attendance.policy.read/manage dan attendance.location.manage, attendance.office_network.manage (org/location scope); tidak otomatis evidence.read.
- Attendance Policy Approver: attendance.policy.approve; approval scope terpisah, tidak boleh approve perubahan sendiri.
- HR Attendance Admin: attendance.team.read, attendance.adjustment.create/approve dan attendance.report.export (HR scope); evidence.read memerlukan grant tersendiri.
- Org Admin: organization.location.manage (assigned org scope), tidak otomatis policy/evidence access.
- IAM Admin: iam.role_assignment.manage pada scope yang ditugaskan; tidak dapat self-grant atau grant Platform Admin.
- Auditor: audit.read dan attendance.policy.history.read (read-only scoped), tanpa foto secara default.
- Platform Admin: platform.config.manage; tidak otomatis HR/attendance data access.

Role assignments memuat principal_id, role_id, scope_type, scope_id, valid_from/to, assigned_by. Admin dashboard menampilkan hanya actions yang authorized; API memeriksa ulang semua permission.
