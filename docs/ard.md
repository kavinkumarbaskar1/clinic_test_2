# Architecture Requirements Document (ARD)

> Every material architecture decision or constraint has a stable ID that feature specifications can reference.

## Document Metadata

- **Product/Application:** Online Appointment Scheduling and Reminders
- **Document ID:** ARD-OASR
- **Version:** 0.1
- **Status:** Draft
- **Architecture Owner:** TBD
- **Contributors:** TBD
- **Last Updated:** 2026-11-19
- **Reference PRD:** `PRD.md`

---

## 1. Architecture Summary

Online Appointment Scheduling and Reminders will provide patient-facing online booking and rescheduling for clinic-selected routine/non-urgent appointments, staff-facing day schedule views, email one-time-code verification, and HIPAA-conscious email reminders.

The approved implementation stack is React for the web application, FastAPI for a modular monolith backend API, and SQLite for application persistence. The backend will expose scheduling, verification, staff schedule, availability-rule, reminder, and integration capabilities through separated modules within one deployable application. Background jobs will support reminder delivery and integration synchronization where required.

The existing practice management/EHR schedule integration is a core architectural dependency, but the specific EHR vendor, system-of-record posture, integration mechanism, hosting model, security quality gates, retention, encryption, SBOM, observability, CI/CD, testing thresholds, and performance/availability targets remain open pending discovery and review. HIPAA obligations are architecture constraints throughout the system.

## 2. Architecture Drivers

- PRD-FR-001 — New and existing patients must directly book clinic-selected routine/non-urgent appointments into available slots and receive immediate confirmation.
- PRD-FR-002 — Patients must directly reschedule eligible appointments online into available slots.
- PRD-FR-003 — The system must send email reminders with minimal PHI and secure access to appointment details.
- PRD-FR-004 — Staff must see the day’s schedule in combined clinic and provider-filtered views.
- PRD-FR-005 — Authorized staff must manage online availability rules for appointment types and slots.
- PRD-FR-006 — Patients must verify identity with an email one-time code before booking or rescheduling completion.
- PRD-FR-007 — The booking flow must guide urgent, unavailable, or ineligible needs to call the clinic or seek emergency care as appropriate.
- PRD-FR-008 — Online booking, rescheduling, and staff schedule views must integrate with the existing practice management/EHR schedule.
- PRD-NFR-001 — Scheduling data must be handled in a HIPAA-conscious manner with minimal PHI in email, role-based staff access, and auditability.
- PRD-NFR-002 — Patient booking and rescheduling must be available 24/7 except scheduled maintenance.
- PRD-NFR-003 — Patient-facing booking and rescheduling performance thresholds must be defined and validated before launch readiness approval.
- PRD-NFR-004 — The patient-facing accessibility standard must be selected and tested before launch readiness approval.
- CON-001 — The clinic is a healthcare business and must account for HIPAA requirements.
- CON-002 — The company is a small business, so the initial release should avoid unnecessary operational complexity.
- CON-003 — Reminder channel for launch is email only.
- CON-004 — Online self-service is limited to clinic-selected routine/non-urgent appointment types.
- CON-005 — Telehealth visit delivery, online payments and billing, and clinical messaging with providers are excluded from the initial release.
- CON-006 — No target launch timing has been set.

## 3. Architecture Principles

1. Minimize PHI exposure across patient-facing screens, email content, logs, analytics, and integration payloads.
2. Prefer a simple modular monolith over distributed services for the initial small-business release.
3. Reuse established repository and platform patterns where appropriate.
4. Keep presentation, orchestration, domain, integration, and persistence concerns separated.
5. Treat the practice management/EHR schedule integration as a critical dependency and prevent false booking confirmation when writes fail.
6. Keep launch scope constrained to routine/non-urgent scheduling, rescheduling, email reminders, and staff schedule visibility.

## 4. System Context

### 4.1 Actors

- Patient — Books eligible routine/non-urgent appointments, reschedules eligible appointments, verifies identity by email one-time code, and securely accesses appointment details.
- Scheduling Staff — Views the day’s schedule in combined clinic and provider-filtered views.
- Authorized Staff Administrator — Configures online appointment type eligibility and slot availability rules.
- Practice Management/EHR Schedule — Existing scheduling system integrated for bookings, reschedules, and staff schedule views.
- Email Delivery Provider — Sends one-time codes, confirmations, and appointment reminders by email.
- Product/Analytics Stakeholder — Reviews booking, rescheduling, and reminder delivery measures.
- Security/HIPAA Reviewer — Reviews HIPAA controls, retention, encryption, scanning, SBOM, and release gates.

### 4.2 External Systems

| ID | System | Purpose | Interface | Owner |
|---|---|---|---|---|
| EXT-001 | Existing practice management/EHR schedule | Provide or receive appointment availability, booking, rescheduling, and staff schedule data | REST/Event/File/TBD | Clinic/vendor, TBD |
| EXT-002 | Email delivery provider | Send one-time codes, confirmations, and email reminders | REST/SMTP/TBD | Vendor, TBD |
| EXT-003 | Hosting platform | Run React/FastAPI/SQLite application and provide operational controls | Platform services/TBD | Vendor, TBD |
| EXT-004 | Analytics/logging platform | Capture booking/rescheduling events, reminder delivery logs, audit logs, and operational telemetry | Event/API/TBD | Vendor/internal, TBD |

## 5. Technology Stack

| Layer | Technology | Version/Constraint | Rationale |
|---|---|---|---|
| Frontend | React | Version TBD; web application approach TBD during vendor/technical discovery | User-selected implementation framework; supports patient and staff web workflows |
| Backend | FastAPI | Version TBD; modular monolith API with background jobs | User-selected implementation framework; keeps initial architecture simple while separating modules |
| Data | SQLite | Version TBD; HIPAA suitability, encryption, backup, retention, and persistence model TBD during security/hosting review | User-selected implementation data store; simple operational profile for small-business launch |
| Hosting | TBD | Hosting model TBD after vendor/technical discovery; must account for HIPAA | Hosting has not been selected and must support HIPAA-conscious operation |
| Email | TBD | Email provider TBD; launch reminder channel is email only | Required for one-time codes, confirmations, and reminders |
| Integration | Vendor-supported EHR/practice management API if available; otherwise TBD | Direct database and RPA approaches are not selected | Supports PRD-FR-008 while deferring vendor-specific mechanism to discovery |

## 6. Logical Architecture

### 6.1 Components

#### COMP-001 — Patient Web Experience

- **Responsibility:** Provide patient booking, rescheduling, one-time-code verification, reminder-detail access, and urgent/ineligible guidance flows.
- **Inputs:** Patient selections, appointment type, available slot, email address, one-time code, reminder access link/token.
- **Outputs:** Booking requests, reschedule requests, verification requests, guidance messages, secure appointment detail views.
- **Dependencies:** COMP-002, COMP-003, COMP-006, COMP-007.
- **Repository Area:** `frontend/` or TBD.

#### COMP-002 — Scheduling API

- **Responsibility:** Orchestrate booking, rescheduling, eligibility checks, confirmation handling, and integration-safe transaction flow.
- **Inputs:** Booking/reschedule commands, appointment eligibility rules, verified patient context, availability data.
- **Outputs:** Confirmed booking/reschedule results, error states, audit events, analytics events.
- **Dependencies:** COMP-004, COMP-006, COMP-007, COMP-008, EXT-001.
- **Repository Area:** `backend/` or TBD.

#### COMP-003 — Email One-Time-Code Verification Service

- **Responsibility:** Issue, validate, expire, and audit email one-time codes before booking or rescheduling completion.
- **Inputs:** Patient email address, code request, submitted code, booking/rescheduling context.
- **Outputs:** Verification status, audit events, email send requests.
- **Dependencies:** COMP-008, EXT-002.
- **Repository Area:** `backend/verification/` or TBD.

#### COMP-004 — Availability Rules Administration

- **Responsibility:** Allow authorized staff to configure online appointment type eligibility and slot availability rules.
- **Inputs:** Staff configuration changes, appointment types, provider/slot rules.
- **Outputs:** Eligibility and availability rules consumed by patient booking/rescheduling flows.
- **Dependencies:** COMP-005, COMP-008.
- **Repository Area:** `backend/availability/` or TBD.

#### COMP-005 — Staff Schedule Experience

- **Responsibility:** Provide combined clinic and provider-filtered day schedule views for staff.
- **Inputs:** Staff user context, selected date, provider filter.
- **Outputs:** Staff day schedule view with permitted appointment details and statuses.
- **Dependencies:** COMP-002, COMP-006, COMP-007, COMP-008, EXT-001.
- **Repository Area:** `frontend/` and `backend/staff_schedule/` or TBD.

#### COMP-006 — Practice Management/EHR Integration Adapter

- **Responsibility:** Encapsulate vendor-supported integration to read availability/schedule data and write booking/rescheduling updates.
- **Inputs:** Booking/reschedule commands, schedule queries, availability queries.
- **Outputs:** EHR/practice management write results, schedule data, integration errors, reconciliation signals.
- **Dependencies:** EXT-001, COMP-008.
- **Repository Area:** `backend/integrations/ehr/` or TBD.

#### COMP-007 — Reminder Processor

- **Responsibility:** Determine eligible reminders and send email reminders with minimal PHI, either internally or through EHR/vendor capability pending discovery.
- **Inputs:** Eligible appointments, reminder schedule, reminder templates, delivery status.
- **Outputs:** Email reminder send requests, delivery logs, failure alerts, audit events.
- **Dependencies:** EXT-002, COMP-006, COMP-008.
- **Repository Area:** `backend/reminders/` or TBD.

#### COMP-008 — Persistence, Audit, and Analytics Store

- **Responsibility:** Store application configuration, verification state, audit logs, operational data, integration state, and analytics events as permitted by the chosen PHI storage posture.
- **Inputs:** Availability rules, verification events, audit events, integration state, analytics events, operational scheduling records as required.
- **Outputs:** Query results for workflows, audit review, analytics measurement, reconciliation.
- **Dependencies:** SQLite, hosting storage, backup mechanism TBD.
- **Repository Area:** `backend/db/` or TBD.

#### COMP-009 — Staff Authentication and Authorization

- **Responsibility:** Authenticate staff users and enforce role-based permissions for schedule access and availability rule management.
- **Inputs:** Staff credentials or identity provider assertions, staff role assignments, access requests.
- **Outputs:** Authenticated staff session, authorization decisions, denied-access audit events.
- **Dependencies:** Identity provider or local staff account model TBD, COMP-008.
- **Repository Area:** `backend/auth/` or TBD.

### 6.2 Data Flow

1. Patient uses COMP-001 to select a clinic-approved routine/non-urgent appointment type and available slot.
2. COMP-001 requests email one-time-code verification through COMP-003.
3. COMP-003 sends the one-time code through EXT-002 and records verification activity in COMP-008.
4. Patient submits the one-time code through COMP-001; COMP-003 validates it before completion is allowed.
5. COMP-002 validates eligibility and availability using COMP-004 and COMP-006.
6. COMP-002 sends booking or rescheduling updates to EXT-001 through COMP-006.
7. COMP-006 returns success or failure; COMP-002 confirms only after successful integration write or selected system-of-record transaction.
8. COMP-002 records audit and analytics events in COMP-008 and returns confirmation or error state to COMP-001.
9. Staff uses COMP-005 to view a selected day schedule; COMP-005 retrieves permitted schedule data through COMP-002/COMP-006 and enforces access through COMP-009.
10. COMP-007 identifies reminder-eligible appointments and sends minimal-PHI email reminders through EXT-002 or delegates to the EHR/vendor reminder engine if selected during discovery.
11. COMP-007 records reminder send attempts, delivery failures, and audit/operational events in COMP-008.

## 7. Architecture Decisions

### ARD-DEC-001 — Use React, FastAPI, and SQLite for Initial Implementation

- **Status:** Accepted
- **Related Requirements:** PRD-FR-001, PRD-FR-002, PRD-FR-004, PRD-FR-005, PRD-FR-006, PRD-FR-008, PRD-NFR-001

**Decision**

The initial implementation will use React for the web experience, FastAPI for the backend API, and SQLite for persistence.

**Rationale**

The selected stack matches the user’s implementation direction and keeps the initial small-business release technically simple.

**Consequences**

- The application can be delivered with a smaller operational footprint.
- SQLite persistence requires explicit security, backup, encryption, concurrency, and hosting validation before production use with scheduling PHI.
- Hosting and deployment architecture must account for persistent SQLite storage and recovery.

**Alternatives Considered**

- Off-the-shelf HIPAA-capable scheduling portal — not selected during the interview; web technology remains implementation-controlled.
- Separate patient and staff web applications — deferred because the web experience approach remains TBD during vendor/technical discovery.
- Microservices — not selected because initial scope and company size favor lower operational complexity.

### ARD-DEC-002 — Backend as Modular Monolith with Background Jobs

- **Status:** Accepted
- **Related Requirements:** PRD-FR-001, PRD-FR-002, PRD-FR-003, PRD-FR-004, PRD-FR-005, PRD-FR-006, PRD-FR-008, PRD-NFR-001

**Decision**

The FastAPI backend will be structured as a modular monolith with separated modules for scheduling, verification, availability rules, staff schedule, reminders, integrations, auth, audit, and persistence. Background jobs will support reminders and integration synchronization where needed.

**Rationale**

A modular monolith supports separation of concerns without microservice overhead, which fits the small-business constraint in CON-002.

**Consequences**

- Feature delivery and operations are simpler than distributed services.
- Internal module boundaries must be maintained to avoid a tightly coupled codebase.
- Background job implementation details remain TBD until hosting and reminder architecture are selected.

**Alternatives Considered**

- Microservices split by domain — not selected due to operational complexity.
- Serverless functions per workflow — not selected because hosting remains TBD and the selected backend is FastAPI.
- EHR/practice management backend only — not selected because the product requires custom patient and staff workflows.

### ARD-DEC-003 — Use Vendor-Supported EHR/Practice Management API if Available

- **Status:** Accepted
- **Related Requirements:** PRD-FR-001, PRD-FR-002, PRD-FR-004, PRD-FR-008

**Decision**

The integration adapter will use a vendor-supported EHR/practice management API if available. If no suitable API is available, the integration mechanism remains TBD and must be resolved before launch readiness.

**Rationale**

The PRD requires integration with the existing practice management/EHR schedule, but the vendor and supported interfaces are not yet known.

**Consequences**

- Integration discovery is a critical path activity.
- False confirmation must be prevented when integration writes fail.
- Contract tests will depend on the selected vendor interface.

**Alternatives Considered**

- Direct database integration — not selected because vendor support and safety are unknown.
- Manual CSV import/export — not selected for direct booking/rescheduling workflows requiring immediate confirmation.
- Screen automation/RPA — not selected due to reliability and maintainability concerns.

### ARD-DEC-004 — Email-Only Reminder and Verification Channel

- **Status:** Accepted
- **Related Requirements:** PRD-FR-003, PRD-FR-006, PRD-NFR-001

**Decision**

Email will be the launch channel for one-time codes, confirmations, and appointment reminders. Reminder emails must contain minimal PHI and route appointment details behind secure verification or login.

**Rationale**

The PRD selects email only for launch and requires minimal PHI exposure under HIPAA-conscious handling.

**Consequences**

- Patients without email access may not be able to use self-service workflows.
- Email provider selection must account for HIPAA obligations and minimal-PHI templates.
- Reminder delivery logs are required for staff review and operational monitoring.

**Alternatives Considered**

- SMS reminders — not selected for launch.
- Automated phone calls — not selected for launch.
- No reminders until secure messaging — not selected because email reminders are in scope.

### ARD-DEC-005 — Defer Hosting, Web Application Shape, and Staff Auth Model to Discovery

- **Status:** Proposed
- **Related Requirements:** PRD-FR-004, PRD-FR-005, PRD-NFR-001, PRD-NFR-002

**Decision**

The architecture records React, FastAPI, and SQLite as selected technologies, but the final web application shape, hosting platform, deployment model, staff authentication model, and authorization integration remain TBD during technical/vendor/security discovery.

**Rationale**

The user explicitly selected TBD for these topics, and the EHR/practice management system and hosting platform are not yet known.

**Consequences**

- Open architecture questions must be resolved before launch readiness approval.
- Security, deployment, and testing controls cannot be finalized until discovery decisions are made.
- Feature specifications should reference open questions where implementation depends on unresolved decisions.

**Alternatives Considered**

- Single responsive web app — recommended but not selected; approach remains TBD.
- HIPAA-eligible managed cloud hosting — recommended but not selected; hosting remains TBD.
- Local staff accounts — recommended but not selected; staff auth remains TBD.

## 8. API and Integration Architecture

### ARD-API-001 — Patient Booking API

- **Type:** REST
- **Operation/Topic:** `POST /api/bookings`
- **Provider:** COMP-002
- **Consumers:** COMP-001
- **Authentication:** Email one-time-code verification through COMP-003 before completion
- **Versioning:** URL path or header versioning TBD during API design

**Request/Input Contract**

Booking request containing selected eligible appointment type, selected slot, patient identifying/contact information required for verification and booking, and validated one-time-code context. Exact schema TBD during implementation design.

**Response/Output Contract**

Success response returns confirmed appointment reference, scheduled time, provider or clinic context as permitted, and confirmation status. Failure response returns safe error code/message for unavailable, ineligible, urgent, verification failed, maintenance, or integration failure states.

**Error Handling**

The API must not confirm bookings unless the required EHR/practice management write or selected system-of-record transaction succeeds. Integration failures return a non-confirmed error state. Idempotency key policy is TBD and tracked in Open Architecture Questions.

### ARD-API-002 — Patient Rescheduling API

- **Type:** REST
- **Operation/Topic:** `POST /api/appointments/{appointment_id}/reschedule`
- **Provider:** COMP-002
- **Consumers:** COMP-001
- **Authentication:** Email one-time-code verification through COMP-003 before completion
- **Versioning:** URL path or header versioning TBD during API design

**Request/Input Contract**

Reschedule request containing appointment reference, selected new slot, patient verification context, and eligibility context. Exact schema TBD during implementation design.

**Response/Output Contract**

Success response returns updated appointment reference, new scheduled time, and confirmation status. Failure response returns safe error code/message for ineligible appointment, unavailable slot, verification failure, maintenance, or integration failure.

**Error Handling**

The API must preserve or clearly report the original appointment state if rescheduling fails. Integration failures must prevent false confirmation. Retry and idempotency rules are TBD.

### ARD-API-003 — Email One-Time-Code API

- **Type:** REST
- **Operation/Topic:** `POST /api/verification/email-code` and `POST /api/verification/email-code/validate`
- **Provider:** COMP-003
- **Consumers:** COMP-001
- **Authentication:** Public initiation with abuse controls TBD; validation by submitted code
- **Versioning:** URL path or header versioning TBD during API design

**Request/Input Contract**

Code request contains email address and workflow context. Validation request contains email address, submitted code, and workflow context. Exact schema, expiry period, rate limits, and lockout rules are TBD.

**Response/Output Contract**

Code request returns accepted/rejected status without exposing whether sensitive patient records exist. Validation returns verified/not verified status and safe error states.

**Error Handling**

Invalid or expired codes prevent booking or rescheduling completion. Failed attempts and abuse-relevant events are recorded for audit/security review.

### ARD-API-004 — Staff Schedule API

- **Type:** REST
- **Operation/Topic:** `GET /api/staff/schedule?date={date}&provider_id={provider_id}`
- **Provider:** COMP-005/COMP-002
- **Consumers:** Staff web experience
- **Authentication:** Staff authentication model TBD; role-based authorization required
- **Versioning:** URL path or header versioning TBD during API design

**Request/Input Contract**

Schedule query with selected date and optional provider filter. Staff identity and role context are required.

**Response/Output Contract**

Permitted appointment records for the selected day, including appointment time, patient identity, provider, and appointment status according to staff authorization.

**Error Handling**

Unauthorized requests are denied and recorded. Integration failures return safe staff-facing error state and must not expose unauthorized PHI.

### ARD-API-005 — Availability Rules API

- **Type:** REST
- **Operation/Topic:** `GET/POST/PATCH/DELETE /api/admin/availability-rules`
- **Provider:** COMP-004
- **Consumers:** Authorized staff administration UI, COMP-002
- **Authentication:** Staff authentication model TBD; authorized staff role required for mutations
- **Versioning:** URL path or header versioning TBD during API design

**Request/Input Contract**

Availability rule configuration for eligible appointment types, slot rules, provider/clinic applicability, and active/inactive status. Exact schema TBD.

**Response/Output Contract**

Created/updated/deleted rule status and effective patient-facing availability configuration.

**Error Handling**

Unauthorized mutations are denied and recorded. Invalid rules are rejected before they affect patient-facing booking options.

### ARD-API-006 — Practice Management/EHR Schedule Integration

- **Type:** REST | Event | File
- **Operation/Topic:** `TBD by EHR/practice management vendor`
- **Provider:** EXT-001 through COMP-006
- **Consumers:** COMP-002, COMP-005, COMP-007
- **Authentication:** Vendor-supported authentication mechanism TBD
- **Versioning:** Vendor-supported versioning TBD

**Request/Input Contract**

Vendor-specific availability query, appointment create, appointment update/reschedule, and schedule read contracts. Exact schema TBD after EHR/practice management discovery.

**Response/Output Contract**

Vendor-specific schedule data, appointment write confirmations, error states, and reconciliation data. Exact schema TBD.

**Error Handling**

Integration failures must prevent false patient confirmation, produce staff-visible operational error states where applicable, and be logged for reconciliation and alerting. Retry and idempotency strategy TBD after vendor selection.

### ARD-API-007 — Email Delivery Integration

- **Type:** REST | SMTP
- **Operation/Topic:** `TBD by email provider`
- **Provider:** EXT-002
- **Consumers:** COMP-003, COMP-007
- **Authentication:** Provider-supported credential or token model TBD
- **Versioning:** Provider-supported versioning TBD

**Request/Input Contract**

Minimal-PHI email template payloads for one-time codes, confirmations, and reminders. Exact payload and template variables TBD.

**Response/Output Contract**

Accepted/rejected send status, delivery failure events if available, and provider message reference.

**Error Handling**

Send failures are logged. Reminder delivery failures must be available for staff review. One-time-code send failures must prevent completion and show a safe retry/error state.

## 9. Data Architecture

### DATA-001 — Appointment

- **Purpose:** Represent booked and rescheduled patient appointments for online workflows and staff schedule views.
- **System of Record:** TBD after EHR/practice management integration discovery
- **Classification:** Restricted
- **Retention:** TBD during HIPAA/security review
- **Key Relationships:** Patient, provider, appointment type, slot, reminder, audit event, EHR/practice management appointment reference.

### DATA-002 — Patient Scheduling Identity

- **Purpose:** Support email one-time-code verification and scheduling workflow association for patients.
- **System of Record:** TBD after EHR/practice management integration discovery
- **Classification:** Restricted
- **Retention:** TBD during HIPAA/security review
- **Key Relationships:** Appointment, verification event, audit event, email address.

### DATA-003 — Availability Rule

- **Purpose:** Define which appointment types and slots are available for online booking/rescheduling.
- **System of Record:** Scheduling application unless EHR/practice management discovery assigns this responsibility elsewhere
- **Classification:** Internal
- **Retention:** TBD during HIPAA/security review
- **Key Relationships:** Appointment type, provider, slot, authorized staff administrator, audit event.

### DATA-004 — Email One-Time Code and Verification Event

- **Purpose:** Verify patient identity before booking or rescheduling completion.
- **System of Record:** Scheduling application
- **Classification:** Confidential
- **Retention:** TBD during HIPAA/security review
- **Key Relationships:** Patient scheduling identity, booking/reschedule workflow, audit event, email delivery event.

### DATA-005 — Reminder Event

- **Purpose:** Track reminder eligibility, send attempts, and delivery failures for appointment reminders.
- **System of Record:** TBD; scheduling application or EHR/vendor reminder engine depending on discovery
- **Classification:** Restricted
- **Retention:** TBD during HIPAA/security review
- **Key Relationships:** Appointment, email delivery provider message reference, delivery status, audit event.

### DATA-006 — Staff User and Role

- **Purpose:** Authenticate staff and enforce role-based access to scheduling data and availability rule management.
- **System of Record:** TBD during staff authentication discovery
- **Classification:** Confidential
- **Retention:** TBD during HIPAA/security review
- **Key Relationships:** Role, schedule access, availability rule mutation, audit event.

### DATA-007 — Audit Event

- **Purpose:** Record access to patient scheduling data, configuration changes, verification outcomes, and denied access attempts.
- **System of Record:** Scheduling application or selected audit platform TBD
- **Classification:** Restricted
- **Retention:** TBD during HIPAA/security review
- **Key Relationships:** Staff user, patient scheduling identity, appointment, action, timestamp, source IP/device metadata as approved.

### DATA-008 — Analytics Event

- **Purpose:** Measure online bookings outside office hours, patient-completed reschedules, and reminder delivery success.
- **System of Record:** Analytics/logging platform TBD
- **Classification:** Internal or Restricted depending on payload design
- **Retention:** TBD during HIPAA/security review
- **Key Relationships:** Booking/reschedule/reminder workflow, timestamp, office-hours classification.

### Data Migration

- **Required:** No for existing appointment schedule migration, based on PRD rollout statement that existing schedule data remains in the practice management/EHR schedule; final migration/reconciliation needs are TBD after integration discovery.
- **Approach:** Existing schedule data is accessed through the EHR/practice management integration where possible. Any local operational/cache data and backfill approach remains TBD after the system-of-record and integration mechanism are selected.

## 10. Security Architecture

### ARD-SEC-001 — HIPAA-Conscious PHI Minimization and Access Control

- **Related Requirements:** PRD-NFR-001

The implementation must:

- Minimize PHI in email reminders, logs, analytics events, and operational telemetry.
- Keep appointment details behind secure verification or login.
- Enforce role-based staff access to scheduling data and availability rule management.
- Log access to patient scheduling data with user, timestamp, and action.
- Record denied unauthorized access attempts.
- Account for HIPAA requirements in hosting, persistence, backups, operations, vendor selection, and support workflows.

The implementation must not:

- Include detailed visit reason or unnecessary PHI in email reminder bodies.
- Confirm a booking or reschedule when required integration writes fail.
- Allow unauthorized users to create, edit, or delete availability rules.
- Store secrets in source code.
- Log one-time codes, raw secrets, or unnecessary PHI.

### ARD-SEC-002 — Patient Email One-Time-Code Verification

- **Related Requirements:** PRD-FR-006, PRD-NFR-001

The implementation must:

- Require a valid email one-time code before booking or rescheduling completion.
- Record successful and failed verification outcomes as security/audit events.
- Prevent invalid or expired codes from completing scheduling workflows.
- Define expiry, retry, rate-limit, and abuse-control policies before launch readiness.

The implementation must not:

- Reveal sensitive patient-record existence through verification errors.
- Permit booking or rescheduling completion using an expired or invalid code.
- Send one-time codes through non-email channels for launch unless the PRD changes.

### ARD-SEC-003 — Staff Authentication and Authorization

- **Related Requirements:** PRD-FR-004, PRD-FR-005, PRD-NFR-001

The implementation must:

- Authenticate staff before staff schedule access or availability rule management.
- Restrict staff access by authorized role.
- Distinguish staff users from patient verification flows.
- Audit schedule access, availability rule changes, and denied access attempts.

The implementation must not:

- Permit anonymous staff schedule access.
- Permit non-authorized staff to manage online availability rules.
- Expose patient scheduling details beyond the user’s permitted access.

### Security Controls

- **Authentication:** Patient booking/rescheduling uses email one-time-code verification; staff authentication model is TBD during technical discovery.
- **Authorization:** Role-based staff access is required for schedule views and availability rule management; exact role model is TBD.
- **Secrets Management:** TBD during security review; secrets must not be stored in source code.
- **Encryption:** TBD during security review; TLS in transit, persistence encryption, backup encryption, and SQLite storage encryption requirements must be resolved before launch readiness.
- **Audit Logging:** Required for patient scheduling data access, verification outcomes, availability rule changes, reminder activity where applicable, and unauthorized access attempts.
- **Dependency/CVE Scanning:** TBD during security review; blocking severity threshold unresolved.
- **SBOM:** TBD during security review.

## 11. Reliability, Performance, and Scalability

### ARD-OPS-001 — Patient Booking Availability

- **Target:** Best effort for initial release; measurable availability target TBD after hosting discovery.
- **Validation Method:** Hosting discovery must define uptime target and scheduled maintenance handling; pre-launch validation must verify booking and rescheduling are unavailable only during scheduled maintenance or controlled failure states.

### ARD-OPS-002 — Patient Booking Performance

- **Target:** Best effort for initial release; measurable performance threshold TBD after load testing and technical discovery.
- **Validation Method:** Load/performance test patient booking and rescheduling flows against the documented threshold before launch readiness approval.

### ARD-OPS-003 — Integration Reliability

- **Target:** 100% of patient confirmations occur only after required EHR/practice management write or selected system-of-record transaction succeeds.
- **Validation Method:** Integration tests and failure-injection tests verify failed EHR/practice management writes return non-confirmed error states.

### ARD-OPS-004 — Reminder Delivery Tracking

- **Target:** 100% of email reminder send attempts record success or failure status when sent by the scheduling application.
- **Validation Method:** Integration tests verify reminder send attempts create delivery log records and failures are reportable for staff review.

### ARD-OPS-005 — Staff Schedule Freshness

- **Target:** TBD after EHR/practice management integration discovery determines schedule read/write model.
- **Validation Method:** Contract/integration tests must validate that online bookings and reschedules appear in staff day schedule views according to the selected freshness target.

## 12. Observability

- **Logging:** Structured application logs, audit logs, and integration/reminder error logs are required in principle; exact platform, retention, and PHI redaction policy are TBD during implementation/security review.
- **Metrics:** Metrics must support KPI-001, KPI-002, KPI-003, booking/rescheduling error rates, reminder failures, verification failures, and integration failures; exact metrics platform is TBD.
- **Tracing:** TBD during implementation planning; distributed tracing may be limited by modular monolith and selected hosting platform.
- **Alerting:** Alerting requirements are TBD; at minimum, unresolved architecture questions must determine alerts for failed booking/reschedule integration writes and reminder send failures.
- **Sensitive Data Restrictions:** Logs, metrics, traces, and analytics must not contain unnecessary PHI, raw one-time codes, secrets, or detailed visit reasons.

## 13. Deployment Architecture

- **Environments:** TBD during engineering planning; dev/test/stage/prod expectations require confirmation.
- **Deployment Model:** TBD during hosting discovery; must support React, FastAPI, SQLite persistence, background jobs, and HIPAA-conscious operation.
- **CI/CD:** TBD during engineering planning.
- **Configuration Management:** TBD; configuration and secrets must remain outside source code.
- **Rollback:** TBD during engineering planning; rollback must account for SQLite schema changes and integration state.
- **Infrastructure as Code:** TBD during engineering planning and hosting discovery.

## 14. Testing Architecture

Required test levels:

- Unit tests
- Integration tests
- Contract tests
- End-to-end tests
- Security tests
- Performance tests, where applicable

Minimum quality expectations:

- **Coverage Threshold:** TBD during engineering planning.
- **Blocking Security Severity:** TBD during security review.
- **Required Build/Lint Checks:** TBD during engineering planning; React and FastAPI build/lint/type/static checks require definition before launch readiness.

## 15. Architecture Constraints

- ARC-CON-001 — HIPAA requirements must be accounted for across hosting, data handling, access control, audit logging, vendors, backups, and operations.
- ARC-CON-002 — The initial release must avoid unnecessary operational complexity because the company is a small business.
- ARC-CON-003 — Reminder channel for launch is email only.
- ARC-CON-004 — Online self-service is limited to clinic-selected routine/non-urgent appointment types.
- ARC-CON-005 — Telehealth visit delivery, online payments and billing, and clinical messaging with providers are excluded from the initial release.
- ARC-CON-006 — No target launch timing has been set.
- ARC-CON-007 — The approved implementation stack is React, FastAPI, and SQLite.
- ARC-CON-008 — The system must integrate with the existing practice management/EHR schedule, but vendor and mechanism are TBD.
- ARC-CON-009 — Patient-facing booking and rescheduling must be available 24/7 except during scheduled maintenance, with measurable target TBD after hosting discovery.
- ARC-CON-010 — Email reminders must contain minimal PHI and keep appointment details behind secure verification or login.

## 16. Risks and Trade-offs

| ID | Risk/Trade-off | Impact | Mitigation |
|---|---|---|---|
| ARC-RISK-001 | SQLite may not meet production concurrency, encryption, backup, or HIPAA operational needs without additional controls | High | Resolve hosting, encryption, backup, and persistence controls during security/hosting review before launch readiness |
| ARC-RISK-002 | EHR/practice management integration mechanism is unknown | High | Prioritize vendor discovery; use vendor-supported API if available; block false confirmations on write failures |
| ARC-RISK-003 | System-of-record posture for appointment data is unresolved | High | Resolve during EHR/practice management discovery and update data architecture before feature build completion |
| ARC-RISK-004 | Email reminders could expose PHI if templates or logs are not minimized | High | Use minimal-PHI templates, secure detail access, PHI log restrictions, and security review |
| ARC-RISK-005 | Staff authentication model is unresolved | Medium | Resolve during technical discovery; require role-based authorization and audit logging regardless of auth source |
| ARC-RISK-006 | Hosting and deployment are unresolved | High | Complete hosting discovery with HIPAA review before production readiness |
| ARC-RISK-007 | Testing thresholds, security gates, SBOM, and CI/CD are unresolved | Medium | Resolve during engineering/security planning before launch readiness approval |
| ARC-RISK-008 | Performance and accessibility targets remain TBD | Medium | Define measurable targets before launch readiness approval as required by PRD-NFR-003 and PRD-NFR-004 |
| ARC-RISK-009 | Reminder processing model is unresolved | Medium | Determine whether EHR/vendor reminder engine or internal background job owns reminder sends during vendor discovery |

## 17. Open Architecture Questions

| ID | Question | Owner | Status | Decision/Due Date |
|---|---|---|---|---|
| ARC-OQ-001 | Will the web experience be a single responsive React app, separate patient/staff apps, or another vendor-driven shape? | Technical Lead | Open | TBD |
| ARC-OQ-002 | Which hosting platform and deployment model will support React, FastAPI, SQLite, background jobs, backups, and HIPAA-conscious operation? | Technical Lead / Security Reviewer | Open | TBD |
| ARC-OQ-003 | What is the practice management/EHR system, supported integration mechanism, authentication model, versioning, and contract? | Technical Lead | Open | TBD |
| ARC-OQ-004 | What system is the system of record for appointment data after EHR/practice management discovery? | Technical Lead / Product Owner | Open | TBD |
| ARC-OQ-005 | What local PHI data storage posture is required for workflows, reminders, audit logs, analytics, cache, and reconciliation? | Security Reviewer / Technical Lead | Open | TBD |
| ARC-OQ-006 | What staff authentication source and role model will be used? | Technical Lead / Security Reviewer | Open | TBD |
| ARC-OQ-007 | Will reminders be processed by the EHR/practice management reminder engine, an internal background job, or another vendor capability? | Technical Lead | Open | TBD |
| ARC-OQ-008 | What observability platform, logging retention, alerting rules, and PHI redaction policy will be used? | Technical Lead / Security Reviewer | Open | TBD |
| ARC-OQ-009 | What measurable availability target will apply after hosting discovery? | Technical Lead | Open | TBD |
| ARC-OQ-010 | What measurable performance thresholds will patient booking and rescheduling meet after load testing? | Technical Lead | Open | TBD |
| ARC-OQ-011 | What CI/CD pipeline, required build/lint checks, rollback approach, and infrastructure-as-code standard will be used? | Engineering Lead | Open | TBD |
| ARC-OQ-012 | What data retention policy applies to locally stored scheduling, verification, reminder, analytics, and audit data? | Security Reviewer | Open | TBD |
| ARC-OQ-013 | What encryption controls are required for data in transit, SQLite/application storage, backups, and secrets? | Security Reviewer | Open | TBD |
| ARC-OQ-014 | What dependency/container/static security scanning gates and blocking severities apply? | Security Reviewer | Open | TBD |
| ARC-OQ-015 | Is SBOM generation required for frontend, backend, container/runtime, or other artifacts? | Security Reviewer | Open | TBD |
| ARC-OQ-016 | What minimum automated test coverage threshold and required test suite policy will block release? | Engineering Lead | Open | TBD |
| ARC-OQ-017 | What accessibility standard will the patient-facing experience meet? | Product Owner | Open | TBD |
| ARC-OQ-018 | What API idempotency, retry, and reconciliation policy will be used for booking/rescheduling integration writes? | Technical Lead | Open | TBD |
| ARC-OQ-019 | What one-time-code expiry, retry, rate-limit, and abuse-control rules will apply? | Security Reviewer / Technical Lead | Open | TBD |

## 18. Feature Mapping

| Feature | Relevant Components | Decision IDs | Interface IDs | Security IDs |
|---|---|---|---|---|
| FEAT-001 | COMP-001, COMP-002, COMP-004, COMP-006, COMP-008 | ARD-DEC-001, ARD-DEC-002, ARD-DEC-003 | ARD-API-001, ARD-API-006 | ARD-SEC-001, ARD-SEC-002 |
| FEAT-002 | COMP-001, COMP-002, COMP-006, COMP-008 | ARD-DEC-001, ARD-DEC-002, ARD-DEC-003 | ARD-API-002, ARD-API-006 | ARD-SEC-001, ARD-SEC-002 |
| FEAT-003 | COMP-007, COMP-008 | ARD-DEC-002, ARD-DEC-004 | ARD-API-007 | ARD-SEC-001 |
| FEAT-004 | COMP-005, COMP-006, COMP-009 | ARD-DEC-001, ARD-DEC-002, ARD-DEC-003, ARD-DEC-005 | ARD-API-004, ARD-API-006 | ARD-SEC-001, ARD-SEC-003 |
| FEAT-005 | COMP-004, COMP-008, COMP-009 | ARD-DEC-001, ARD-DEC-002, ARD-DEC-005 | ARD-API-005 | ARD-SEC-001, ARD-SEC-003 |
| FEAT-006 | COMP-003, COMP-008 | ARD-DEC-001, ARD-DEC-002, ARD-DEC-004 | ARD-API-003, ARD-API-007 | ARD-SEC-001, ARD-SEC-002 |
| FEAT-007 | COMP-006, COMP-002, COMP-005, COMP-007 | ARD-DEC-003, ARD-DEC-005 | ARD-API-006 | ARD-SEC-001 |

## 19. Approval

| Role | Name | Decision | Date |
|---|---|---|---|
| Solution Architect | TBD | Pending | TBD |
| Security Reviewer | TBD | Pending | TBD |
| Engineering Lead | TBD | Pending | TBD |