# Product Requirements Document (PRD)

> Requirement IDs should remain stable after approval so downstream ARD and feature specifications retain traceability.

## Document Metadata

- **Product/Application:** Online Appointment Scheduling and Reminders
- **Document ID:** PRD-OASR
- **Version:** 0.1
- **Status:** Draft
- **Owner:** TBD
- **Contributors:** TBD
- **Last Updated:** 2026-11-19

---

## 1. Executive Summary

The clinic loses bookings because patients can only book by phone during office hours. This product enables new and existing patients to book and reschedule eligible routine/non-urgent appointments online, receive email reminders, and access appointment details securely with minimal protected health information exposed in email.

The product is intended for a small healthcare business operating under HIPAA obligations. Staff will be able to view the day’s schedule in one place, with both combined clinic and provider-filtered views, and authorized staff will manage which appointment types and slots are available online.

The expected value is fewer missed after-hours booking opportunities, a more convenient patient experience, and a more centralized daily scheduling workflow for clinic staff.

## 2. Problem Statement

Today, patients can only book appointments by phone during office hours. This creates missed booking opportunities outside office hours for new and existing patients and increases reliance on staff-mediated scheduling. Staff also need a consolidated way to see the day’s schedule.

## 3. Product Vision

Patients can self-serve routine/non-urgent scheduling needs online at any time, while the clinic maintains control over eligible appointment types, availability, reminders, and HIPAA-compliant handling of appointment information. Staff can manage the daily schedule from a single schedule view connected to the clinic’s existing practice management/EHR schedule.

## 4. Goals and Success Measures

### 4.1 Goals

- Reduce missed booking opportunities outside office hours.
- Allow new and existing patients to book eligible routine/non-urgent appointments online.
- Allow patients to directly reschedule eligible appointments online.
- Send HIPAA-conscious email reminders with minimal PHI and secure access to details.
- Give staff a unified day schedule view with combined clinic and provider-filtered views.

### 4.2 Success Measures

| ID | Measure | Baseline | Target | Measurement Method |
|---|---|---:|---:|---|
| KPI-001 | Completed online bookings outside office hours | TBD | TBD | Analytics event count for completed online bookings with timestamp outside clinic office hours |
| KPI-002 | Online reschedules completed by patients | TBD | TBD | Analytics event count for completed patient-initiated reschedules |
| KPI-003 | Email appointment reminders sent successfully | TBD | TBD | Reminder delivery logs and failure reports |

## 5. Scope

### 5.1 In Scope

- Online booking for new and existing patients.
- Direct booking into available slots for clinic-selected routine/non-urgent appointment types.
- Direct patient rescheduling for eligible appointments into available slots.
- Email reminders with minimal PHI and appointment details behind secure login.
- Email-based one-time-code identity verification before booking or rescheduling completion.
- Staff day schedule view with combined clinic view and provider-filtered views.
- Authorized staff management of online availability rules in the scheduling system.
- Integration with the existing practice management/EHR schedule.
- Guidance for urgent, unavailable, or ineligible appointment needs to call the clinic or seek emergency care as appropriate.

### 5.2 Out of Scope

- Telehealth visit delivery.
- Online payments and billing.
- Clinical messaging with providers.
- Online booking for urgent appointments.
- Online booking for appointment types not selected by the clinic as routine/non-urgent and eligible.

## 6. Personas and Users

### PERSONA-001 — Patient

- **Description:** A new or existing clinic patient who wants to book or reschedule an eligible routine/non-urgent appointment online.
- **Primary Need:** Complete appointment scheduling outside phone-only office hours.
- **Key Pain Point:** The patient may lose the chance to book if they cannot call during office hours.

### PERSONA-002 — Scheduling Staff

- **Description:** Clinic staff responsible for viewing and managing the daily appointment schedule.
- **Primary Need:** See the day’s schedule in one place, either across the clinic or filtered by provider.
- **Key Pain Point:** Schedule visibility is fragmented or dependent on manual phone-based workflows.

### PERSONA-003 — Authorized Staff Administrator

- **Description:** Authorized clinic staff member responsible for managing online scheduling availability rules.
- **Primary Need:** Control which routine/non-urgent appointment types and slots are available online.
- **Key Pain Point:** Uncontrolled online availability could expose inappropriate appointment types or slots.

## 7. Assumptions and Dependencies

### Assumptions

- ASM-001 — The clinic can identify which appointment types are routine/non-urgent and eligible for online booking.
- ASM-002 — Patients have access to email for one-time-code verification and reminders.
- ASM-003 — Email reminders can be implemented with minimal PHI and secure-login access to details in a HIPAA-conscious manner.
- ASM-004 — Clinic office hours will be available to classify bookings as inside or outside office hours.

### Dependencies

- DEP-001 — Integration access to the existing practice management/EHR schedule.
- DEP-002 — Email delivery capability for one-time codes and appointment reminders.
- DEP-003 — HIPAA-compliant hosting, access control, audit logging, and data handling practices.
- DEP-004 — Authorized staff availability to configure online appointment types and slot rules.

## 8. Functional Requirements

### PRD-FR-001 — Online Appointment Booking

- **Priority:** Must
- **Feature:** FEAT-001
- **Persona:** PERSONA-001

**Requirement**

The system shall allow new and existing patients to directly book clinic-selected routine/non-urgent appointment types into available slots online and receive immediate confirmation.

**Business Rationale**

Patients currently lose booking opportunities because phone booking is limited to office hours.

**Acceptance Criteria**

1. A patient can select an eligible appointment type, choose an available slot, verify identity by email one-time code, and complete the booking without staff intervention.
2. The booked appointment appears in the connected practice management/EHR schedule after completion.
3. The patient receives confirmation after a successful booking.
4. Ineligible appointment types are not bookable online.

---

### PRD-FR-002 — Patient Rescheduling

- **Priority:** Must
- **Feature:** FEAT-002
- **Persona:** PERSONA-001

**Requirement**

The system shall allow patients to directly reschedule eligible appointments into available slots online.

**Business Rationale**

Online rescheduling reduces patient dependence on phone access and supports appointment management outside office hours.

**Acceptance Criteria**

1. A patient can access an eligible appointment, verify identity by email one-time code, choose a new available slot, and complete the reschedule online.
2. The original appointment is updated in the connected practice management/EHR schedule after rescheduling.
3. The patient receives confirmation after a successful reschedule.
4. The system prevents online rescheduling for appointments that are not eligible.

---

### PRD-FR-003 — Email Appointment Reminders

- **Priority:** Must
- **Feature:** FEAT-003
- **Persona:** PERSONA-001

**Requirement**

The system shall send appointment reminders by email with minimal PHI in the email body and appointment details available only behind secure login.

**Business Rationale**

Patients need reminders, but the clinic must handle healthcare information under HIPAA obligations.

**Acceptance Criteria**

1. Reminder emails do not include detailed visit reason or other unnecessary PHI in the email body.
2. Reminder emails provide access to appointment details only after secure patient verification or login.
3. Reminder send attempts and delivery failures are logged for staff review.
4. Reminders are sent only for appointments eligible for reminders.

---

### PRD-FR-004 — Staff Day Schedule View

- **Priority:** Must
- **Feature:** FEAT-004
- **Persona:** PERSONA-002

**Requirement**

The system shall provide staff with the day’s schedule in both combined clinic view and provider-filtered day views.

**Business Rationale**

Staff need a single place to see the day’s appointments across the clinic while retaining provider-level visibility.

**Acceptance Criteria**

1. Staff can view all providers’ appointments for a selected day in one combined schedule view.
2. Staff can filter the selected day schedule by provider.
3. Appointments created or changed online are reflected in the staff day schedule view.
4. Staff can distinguish appointment time, patient identity, provider, and appointment status according to permitted access.

---

### PRD-FR-005 — Online Availability Rule Management

- **Priority:** Must
- **Feature:** FEAT-005
- **Persona:** PERSONA-003

**Requirement**

The system shall allow authorized staff to manage which appointment types and slots are available for online booking.

**Business Rationale**

The clinic must retain operational control over routine/non-urgent appointment eligibility and online availability.

**Acceptance Criteria**

1. Only authorized staff can configure online appointment type eligibility.
2. Only authorized staff can configure online slot availability rules.
3. Patient-facing booking options reflect the configured eligibility and availability rules.
4. Unauthorized users cannot create, edit, or delete online availability rules.

---

### PRD-FR-006 — Email One-Time-Code Verification

- **Priority:** Must
- **Feature:** FEAT-006
- **Persona:** PERSONA-001

**Requirement**

The system shall verify patient identity with an email-based one-time code before booking or rescheduling is completed.

**Business Rationale**

Identity verification helps protect patient appointment information while keeping the online flow accessible.

**Acceptance Criteria**

1. A patient cannot complete booking until a valid email one-time code is entered.
2. A patient cannot complete rescheduling until a valid email one-time code is entered.
3. Invalid or expired one-time codes prevent completion and show an error state.
4. Successful verification is recorded for the booking or rescheduling event.

---

### PRD-FR-007 — Urgent and Ineligible Need Guidance

- **Priority:** Must
- **Feature:** FEAT-001
- **Persona:** PERSONA-001

**Requirement**

The system shall show guidance to call the clinic or seek emergency care as appropriate when a patient need is urgent, unavailable, or ineligible for online booking.

**Business Rationale**

Patients should not use online routine scheduling for urgent care needs or unsupported appointment types.

**Acceptance Criteria**

1. The patient-facing booking flow displays guidance before completion when urgent or ineligible needs are identified.
2. The guidance includes calling the clinic for unsupported needs.
3. The guidance includes seeking emergency care when emergency-level care may be appropriate.
4. The system prevents completion of online booking for ineligible appointment needs.

---

### PRD-FR-008 — Practice Management/EHR Schedule Integration

- **Priority:** Must
- **Feature:** FEAT-007
- **Persona:** PERSONA-002

**Requirement**

The system shall integrate online booking, rescheduling, and staff schedule views with the existing practice management/EHR schedule.

**Business Rationale**

The clinic needs online and staff-facing scheduling to reflect the operational source of scheduling truth.

**Acceptance Criteria**

1. Completed online bookings are written to the existing practice management/EHR schedule.
2. Completed online reschedules update the existing practice management/EHR schedule.
3. Staff schedule views display appointment data from the integrated schedule source.
4. Integration failures prevent false confirmation and present an error state to the patient or staff user.

## 9. Non-Functional Requirements

### PRD-NFR-001 — HIPAA-Conscious Privacy and Security

- **Category:** Compliance
- **Priority:** Must

**Requirement**

The system shall handle appointment scheduling data in a HIPAA-conscious manner, including minimal PHI in email reminders, secure access to appointment details, role-based staff access, and auditability of access to scheduling data.

**Acceptance Criteria**

1. Email reminder content excludes unnecessary PHI and requires secure access for appointment details.
2. Staff access to scheduling information is restricted by authorized role.
3. Access to patient scheduling data is logged with user, timestamp, and action.
4. Unauthorized access attempts are denied and recorded.

### PRD-NFR-002 — Patient Booking Availability

- **Category:** Availability
- **Priority:** Must

**Requirement**

The system shall make the patient-facing booking and rescheduling experience available 24/7 except during scheduled maintenance.

**Acceptance Criteria**

1. Patients can access booking and rescheduling outside clinic office hours when the system is not in scheduled maintenance.
2. Scheduled maintenance states prevent booking completion and communicate that online scheduling is temporarily unavailable.

### PRD-NFR-003 — Patient Booking Performance

- **Category:** Performance
- **Priority:** Should

**Requirement**

The system shall meet patient-facing booking and rescheduling performance targets defined after technical discovery.

**Acceptance Criteria**

1. Performance thresholds are documented before launch readiness approval.
2. Booking and rescheduling flows are tested against the documented thresholds before launch readiness approval.

### PRD-NFR-004 — Accessibility

- **Category:** Accessibility
- **Priority:** Should

**Requirement**

The system shall meet the patient-facing accessibility standard selected before launch readiness approval.

**Acceptance Criteria**

1. The accessibility standard is documented before launch readiness approval.
2. The patient-facing booking, rescheduling, verification, and reminder-detail access flows are tested against the selected standard before launch readiness approval.

## 10. User Journeys

### JOURNEY-001 — Patient Books an Eligible Appointment Online

1. Patient opens the online booking experience outside or inside office hours.
2. Patient selects a clinic-approved routine/non-urgent appointment type.
3. Patient selects an available slot.
4. Patient verifies identity using an email one-time code.
5. Patient completes booking and receives confirmation.
6. Staff sees the appointment in the integrated schedule view.

**Related Requirements:** PRD-FR-001, PRD-FR-006, PRD-FR-008, PRD-NFR-001, PRD-NFR-002

### JOURNEY-002 — Patient Reschedules an Eligible Appointment

1. Patient accesses an eligible appointment online.
2. Patient verifies identity using an email one-time code.
3. Patient selects a new available slot.
4. Patient completes the reschedule and receives confirmation.
5. Staff sees the updated appointment in the integrated schedule view.

**Related Requirements:** PRD-FR-002, PRD-FR-006, PRD-FR-008, PRD-NFR-001, PRD-NFR-002

### JOURNEY-003 — Staff Views the Day’s Schedule

1. Staff opens the day schedule view.
2. Staff views all providers’ appointments in one combined clinic view.
3. Staff filters the day schedule by provider.
4. Staff sees online bookings and reschedules reflected in the schedule.

**Related Requirements:** PRD-FR-004, PRD-FR-008, PRD-NFR-001

### JOURNEY-004 — Patient Receives an Email Reminder

1. System sends an email reminder for an eligible appointment.
2. Patient opens the reminder email containing minimal PHI.
3. Patient follows the secure access path to view appointment details.
4. System logs reminder delivery activity.

**Related Requirements:** PRD-FR-003, PRD-NFR-001

## 11. Feature Breakdown

### FEAT-001 — Online Appointment Booking

- **Summary:** Patient-facing direct booking for eligible routine/non-urgent appointment types.
- **Priority:** Must
- **Related Requirements:** PRD-FR-001, PRD-FR-007, PRD-NFR-001, PRD-NFR-002, PRD-NFR-003, PRD-NFR-004
- **Expected Spec File:** `specs/FEAT-001-online-appointment-booking.spec.md`

### FEAT-002 — Online Rescheduling

- **Summary:** Patient-facing direct rescheduling for eligible appointments.
- **Priority:** Must
- **Related Requirements:** PRD-FR-002, PRD-NFR-001, PRD-NFR-002, PRD-NFR-003, PRD-NFR-004
- **Expected Spec File:** `specs/FEAT-002-online-rescheduling.spec.md`

### FEAT-003 — Email Reminders

- **Summary:** HIPAA-conscious appointment reminders using email only.
- **Priority:** Must
- **Related Requirements:** PRD-FR-003, PRD-NFR-001
- **Expected Spec File:** `specs/FEAT-003-email-reminders.spec.md`

### FEAT-004 — Staff Day Schedule View

- **Summary:** Staff-facing day schedule with combined clinic and provider-filtered views.
- **Priority:** Must
- **Related Requirements:** PRD-FR-004, PRD-FR-008, PRD-NFR-001
- **Expected Spec File:** `specs/FEAT-004-staff-day-schedule-view.spec.md`

### FEAT-005 — Availability Rule Management

- **Summary:** Authorized staff controls online appointment type and slot availability rules.
- **Priority:** Must
- **Related Requirements:** PRD-FR-005, PRD-NFR-001
- **Expected Spec File:** `specs/FEAT-005-availability-rule-management.spec.md`

### FEAT-006 — Email One-Time-Code Verification

- **Summary:** Email-based one-time-code verification before completing booking or rescheduling.
- **Priority:** Must
- **Related Requirements:** PRD-FR-006, PRD-NFR-001
- **Expected Spec File:** `specs/FEAT-006-email-one-time-code-verification.spec.md`

### FEAT-007 — Practice Management/EHR Integration

- **Summary:** Integration with the existing practice management/EHR schedule for bookings, reschedules, and staff views.
- **Priority:** Must
- **Related Requirements:** PRD-FR-008
- **Expected Spec File:** `specs/FEAT-007-practice-management-ehr-integration.spec.md`

## 12. Constraints

- CON-001 — The clinic is a healthcare business and must account for HIPAA requirements.
- CON-002 — The company is a small business, so the initial release should avoid unnecessary operational complexity.
- CON-003 — Reminder channel for launch is email only.
- CON-004 — Online self-service is limited to clinic-selected routine/non-urgent appointment types.
- CON-005 — Telehealth visit delivery, online payments and billing, and clinical messaging with providers are excluded from the initial release.
- CON-006 — No target launch timing has been set.

## 13. Risks

| ID | Risk | Impact | Likelihood | Mitigation |
|---|---|---|---|---|
| RISK-001 | Email reminders could expose PHI if content is not minimized | High | TBD | Keep minimal PHI in email body and require secure access for appointment details |
| RISK-002 | Practice management/EHR integration complexity could delay launch | High | TBD | Confirm integration capabilities during technical discovery before launch planning |
| RISK-003 | Incorrect availability rules could allow patients to book inappropriate appointment types or slots | Medium | TBD | Restrict configuration to authorized staff and test patient-facing eligibility rules |
| RISK-004 | Performance or accessibility expectations may be unresolved until late in delivery | Medium | TBD | Define performance thresholds and accessibility standard before launch readiness approval |

## 14. Open Questions

| ID | Question | Owner | Status | Decision/Due Date |
|---|---|---|---|---|
| OQ-001 | What baseline and target should be set for completed online bookings outside office hours after analytics are instrumented? | Product Owner | Open | TBD |
| OQ-002 | What baseline and target should be set for patient-completed online reschedules? | Product Owner | Open | TBD |
| OQ-003 | What baseline and target should be set for successful email reminder delivery? | Product Owner | Open | TBD |
| OQ-004 | What specific performance thresholds should patient-facing booking and rescheduling meet after technical discovery? | Technical Lead | Open | TBD |
| OQ-005 | What accessibility standard should the patient-facing experience meet? | Product Owner | Open | TBD |
| OQ-006 | What practice management/EHR system and integration mechanism will be used? | Technical Lead | Open | TBD |
| OQ-007 | What launch readiness date or release window should be set? | Product Owner | Open | TBD |
| OQ-008 | What likelihood ratings should be assigned to identified delivery and compliance risks? | Product Owner | Open | TBD |

## 15. Release and Rollout Considerations

- **Target Release:** No target launch timing set.
- **Rollout Approach:** Start with clinic-selected routine/non-urgent appointment types and email reminders only.
- **Migration Considerations:** Existing schedule data remains in the practice management/EHR schedule; integration details require decision in OQ-006.
- **Training/Support Considerations:** Staff need training to manage online availability rules and use combined and provider-filtered day schedule views.

## 16. Definition of Product Success

The product is successful when:

- New and existing patients can complete eligible routine/non-urgent bookings online outside office hours.
- Patients can directly reschedule eligible appointments online.
- Staff can view the day’s schedule in combined clinic and provider-filtered views.
- Email reminders are sent with minimal PHI and secure access to appointment details.
- All Must-priority requirements are accepted.
- Required quality, security, and compliance outcomes are evidenced.

## 17. Approval

| Role | Name | Decision | Date |
|---|---|---|---|
| Product Owner | TBD | Pending | TBD |
| Business Sponsor | TBD | Pending | TBD |