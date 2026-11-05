# SlotWise — Appointment Lifecycle and State Transition Specification

**Version:** 1.0  
**Status:** Draft  
**Phase:** Requirements Analysis

## 1. Purpose

This document defines the lifecycle of an appointment in SlotWise, including its statuses, permitted transitions, triggering actors, and schedule-reservation rules.

The objective is to ensure consistent appointment processing, prevent unauthorized status changes, and protect employee availability.

## 2. Appointment Status Definitions

| Status | Description |
|---|---|
| `PENDING_APPROVAL` | A customer has submitted a booking, and provider-side approval is required. |
| `CONFIRMED` | The appointment has been approved and confirmed. |
| `REJECTED` | An authorized provider-side user has rejected the booking. |
| `RESCHEDULE_REQUESTED` | A new time has been proposed for an existing confirmed appointment and customer acceptance is pending. |
| `IN_PROGRESS` | The appointment service has started. |
| `COMPLETED` | The appointment service has been completed. |
| `CANCELLED` | The appointment has been cancelled by an authorized actor. |
| `EXPIRED` | A pending appointment exceeded its approval deadline without a decision. |
| `NO_SHOW` | The customer did not attend the confirmed appointment. |

## 3. Permitted State Transitions

| Current Status | Next Status | Trigger | Authorized Actor |
|---|---|---|---|
| `PENDING_APPROVAL` | `CONFIRMED` | Booking approved | Authorized provider-side user |
| `PENDING_APPROVAL` | `REJECTED` | Booking rejected | Authorized provider-side user |
| `PENDING_APPROVAL` | `EXPIRED` | Approval deadline exceeded | System |
| `PENDING_APPROVAL` | `CANCELLED` | Customer cancels pending booking | Customer |
| `CONFIRMED` | `RESCHEDULE_REQUESTED` | New appointment time proposed | Authorized provider-side user |
| `RESCHEDULE_REQUESTED` | `CONFIRMED` | Customer accepts proposed time | Customer |
| `RESCHEDULE_REQUESTED` | `CONFIRMED` | Customer declines or response deadline expires | Customer or system |
| `CONFIRMED` | `IN_PROGRESS` | Service starts | Authorized provider-side user or system |
| `CONFIRMED` | `CANCELLED` | Appointment cancelled | Customer or authorized provider-side user |
| `CONFIRMED` | `NO_SHOW` | Customer does not attend | Authorized provider-side user or system |
| `IN_PROGRESS` | `COMPLETED` | Service completed | Authorized provider-side user |
| `IN_PROGRESS` | `CANCELLED` | Service cancelled after starting | TBD |

Any transition not explicitly permitted by the system must be rejected.

## 4. Approval Workflow

1. Customer selects an available time and submits a booking.
2. SlotWise automatically assigns an eligible employee.
3. Appointment is created with `PENDING_APPROVAL`.
4. The assigned employee's time is temporarily reserved.
5. An authorized provider-side user receives an approval notification.
6. The approval deadline is set to 30 minutes after booking creation by default.
7. The deadline is configurable.
8. If approved, the appointment becomes `CONFIRMED`.
9. If rejected, the appointment becomes `REJECTED`, and a rejection reason must be recorded.
10. If no decision is received before the deadline, the system changes the status to `EXPIRED` and releases the reserved time.

The system must automatically process expired appointments without requiring manual intervention.

## 5. Rescheduling Workflow

Rescheduling an existing confirmed appointment requires customer acceptance.

1. An authorized provider-side user proposes a new appointment time.
2. SlotWise validates the proposed time.
3. The original appointment remains confirmed.
4. The original employee schedule remains reserved.
5. The proposed time is temporarily held.
6. The appointment enters `RESCHEDULE_REQUESTED`.
7. The customer receives a notification with the proposed time.
8. The customer has 30 minutes by default to respond.
9. If accepted, SlotWise revalidates the proposed time and confirms the rescheduling.
10. The original reservation is released after successful confirmation.
11. If declined or the response deadline expires, the original appointment remains unchanged and the proposed time is released.

The reschedule response deadline must be configurable.

## 6. Rejected Appointment and Alternative Booking

When a provider rejects a booking:

- The original appointment remains recorded as `REJECTED`.
- A rejection reason must be stored.
- The customer must be notified.
- The customer can check available times and submit another booking.
- A replacement booking should be linked to the rejected appointment for traceability.

A replacement booking is a new appointment, not a status change to the rejected appointment.

## 7. Schedule Reservation Rules

| Appointment State | Schedule Treatment |
|---|---|
| `PENDING_APPROVAL` | Temporarily reserves assigned employee's time |
| `CONFIRMED` | Reserves employee's time |
| `RESCHEDULE_REQUESTED` | Original time remains reserved; proposed time is temporarily held |
| `REJECTED` | Does not reserve time |
| `EXPIRED` | Does not reserve time |
| `CANCELLED` | Does not reserve time |
| `IN_PROGRESS` | Occupies the employee's time until service completion |
| `COMPLETED` | Historical record; no longer reserves time |
| `NO_SHOW` | Historical record; no longer reserves time after the appointment period |

All temporary reservations must be released when their associated workflow expires or is cancelled.

## 8. Audit History

Every appointment status transition must be recorded.

The history should capture:

- Appointment ID
- Previous status
- New status
- Actor ID, where applicable
- Actor type
- Transition timestamp
- Reason, where applicable

The history must be retained even if an appointment is later cancelled or rejected.

## 9. General Business Rules

1. Customers cannot directly confirm their own appointments when provider approval is required.
2. Only authorized provider-side users can approve or reject bookings.
3. A rejection must include a reason.
4. Customers must accept provider-initiated rescheduling proposals.
5. The system must revalidate availability before confirming a booking or reschedule.
6. Concurrent booking requests must not reserve overlapping time for the same employee.
7. Expired temporary reservations must be released automatically.
8. All appointment timestamps must be handled consistently with the provider's configured time zone.
9. All status transitions must be auditable.
10. Appointment operations must respect provider-level permissions.

## 10. Open Decisions

The following policies require further clarification before final database design:

- Whether an employee can cancel an appointment after service has started.
- Whether a customer can cancel a booking while it is pending approval.
- Whether a customer can cancel a confirmed appointment at any time or only before a cancellation deadline.
- Whether an appointment can be manually marked as `NO_SHOW`, and when this becomes permitted.
- Whether the system should automatically mark appointments as `IN_PROGRESS` and `COMPLETED`.
- Whether proposed reschedule holds expire exactly 30 minutes after the proposal or follow another configurable rule.
- How long an appointment can remain pending if the platform's default approval timeout is changed while it is already pending.

## 11. Customer Cancellation Policy

### 11.1 Configurable Cancellation Deadline

Each provider must be able to configure a customer cancellation deadline.

The deadline defines the minimum amount of time before an appointment's scheduled start at which a customer can cancel directly through SlotWise.

**Initial platform default:** 2 hours before the appointment.

The default must be configurable at provider level.

### 11.2 Cancellation Rules

1. Customers may cancel confirmed appointments before the configured cancellation deadline.
2. Customers cannot directly cancel confirmed appointments after the deadline has passed.
3. After the deadline, customers must contact the provider to request cancellation.
4. Authorized provider-side users may cancel appointments even after the customer cancellation deadline.
5. A cancellation reason must be recorded.
6. The system must record who cancelled the appointment and when.
7. Once cancellation is completed, the employee's reserved time must be released.
8. The cancellation policy must be validated using the provider's configured time zone.

### 11.3 Cancellation Deadline Calculation

The cancellation deadline is calculated relative to the appointment's scheduled start time.

For example, if an appointment is scheduled for 3:00 PM and the provider's cancellation deadline is 2 hours, the customer must cancel before 1:00 PM.

### 11.4 Pending Approval Cancellation

Customers may cancel their own pending appointments before approval.

The appointment must transition to `CANCELLED`, and its temporary employee reservation must be released.

### 11.5 Audit Requirements

All cancellations must be recorded in `AppointmentStatusHistory`, including:

- Previous appointment status
- New appointment status
- Actor
- Cancellation timestamp
- Cancellation reason

## 12. No-Show Handling

### 12.1 Manual No-Show

An authorized provider-side user may manually mark a confirmed appointment as `NO_SHOW` when the customer fails to attend.

The system must record the actor, timestamp and optional reason.

### 12.2 Automatic No-Show

SlotWise must support automatic no-show processing.

- The initial default grace period is 15 minutes after the scheduled appointment start time.
- The grace period must be configurable.
- If the appointment remains `CONFIRMED` after the grace period and has not been checked in or started, the system may automatically change its status to `NO_SHOW`.
- Appointments that are already cancelled, rejected, expired, in progress or completed must not be automatically marked as no-show.
- The automatic transition must be recorded in appointment status history, with the system as the actor.

Manual and automatic no-show handling must coexist.

## 13. Appointment Completion

### 13.1 Manual Completion

An authorized provider-side user may manually mark an appointment as `COMPLETED` when the service has finished.

The system must record the actor and completion timestamp.

### 13.2 Automatic Completion

SlotWise must support automatic appointment completion.

- The initial default grace period is 15 minutes after the scheduled appointment end time.
- The grace period must be configurable.
- An appointment that remains `IN_PROGRESS` after the scheduled end time and grace period may automatically transition to `COMPLETED`.
- Appointments that are cancelled, rejected, expired, no-show or already completed must not be automatically completed.
- The automatic transition must be recorded in appointment status history, with the system as the actor.

Manual and automatic completion must coexist.

### 13.3 Automatic Processing Safeguards

1. Automatic processing must verify the current appointment status before making a transition.
2. A manual status update must not be overwritten by an automatic job.
3. Each appointment must have only one successful completion or no-show transition.
4. All automatic transitions must be auditable.
5. Time calculations must respect the provider's configured time zone.

## 14. Additional Open Design Consideration

Automatic completion depends on the appointment entering `IN_PROGRESS`.

The system must therefore define how an appointment starts:
- Manually by an authorized employee.
- Automatically at its scheduled start time.
- Or through a combination of both.

This decision must be finalized before implementing the appointment lifecycle scheduler.

**End of document.**