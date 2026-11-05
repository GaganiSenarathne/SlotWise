# SlotWise — Logical Data Model

**Version:** 1.1  
**Status:** Draft  
**Phase:** Database Design

## 1. Purpose

This document defines the logical data model for SlotWise. It identifies the main entities, their responsibilities, important attributes, and relationships.

The logical model describes what information the system must store without committing to database-specific implementation details.

## 2. Entity Overview

| Entity | Responsibility |
|---|---|
| `User` | Stores customer, provider and employee account information. |
| `Provider` | Represents an individual professional or a business offering services. |
| `ProviderStaff` | Associates users with providers and defines provider-level roles and permissions. |
| `ProviderAppointmentPolicy` | Stores configurable appointment workflow settings for a provider. |
| `ServiceCategory` | Groups services into categories. |
| `ProviderService` | Defines services offered by a provider. |
| `StaffService` | Defines which employees are qualified to perform which services. |
| `AvailabilityRule` | Stores recurring employee working schedules. |
| `AvailabilityException` | Stores employee-specific schedule exceptions. |
| `Appointment` | Stores customer bookings, their current lifecycle state and service snapshots. |
| `AppointmentStatusHistory` | Records appointment status transitions for auditing. |
| `AppointmentReschedule` | Stores proposed changes to existing confirmed appointments. |

## 3. Entity Attributes and Responsibilities

### 3.1 User

- `id`
- `first_name`
- `last_name`
- `email`
- `phone_number`
- `password_hash`
- `account_status`
- `created_at`
- `updated_at`

A user may act as a customer and may also belong to one or more providers through `ProviderStaff`.

### 3.2 Provider

- `id`
- `provider_type` — `INDIVIDUAL` or `BUSINESS`
- `name`
- `description`
- `email`
- `phone_number`
- `address`
- `timezone`
- `status`
- `created_at`
- `updated_at`

The provider's IANA time zone is used when interpreting working schedules, appointment times and policy deadlines.

### 3.3 ProviderStaff

- `id`
- `provider_id`
- `user_id`
- `role`
- `permissions`
- `membership_status`
- `joined_at`

Connects users to providers and defines their provider-level access.

### 3.4 ProviderAppointmentPolicy

- `id`
- `provider_id`
- `approval_timeout_minutes`
- `reschedule_response_timeout_minutes`
- `customer_cancellation_deadline_minutes`
- `no_show_grace_period_minutes`
- `completion_grace_period_minutes`
- `automatic_appointment_start_enabled`
- `automatic_no_show_enabled`
- `automatic_completion_enabled`
- `created_at`
- `updated_at`

Initial platform defaults:

| Setting | Default |
|---|---:|
| Approval timeout | 30 minutes |
| Reschedule response timeout | 30 minutes |
| Customer cancellation deadline | 120 minutes |
| No-show grace period | 15 minutes |
| Completion grace period | 15 minutes |

Provider-specific values may override platform defaults.

The fixed 15-minute slot interval is a platform-wide configuration and is not an employee-specific attribute.

### 3.5 ServiceCategory

- `id`
- `name`
- `description`
- `created_at`
- `updated_at`

Groups similar services.

### 3.6 ProviderService

- `id`
- `provider_id`
- `category_id`
- `name`
- `description`
- `duration_minutes`
- `buffer_before_minutes`
- `buffer_after_minutes`
- `price`
- `currency`
- `status`
- `created_at`
- `updated_at`

Defines a service offered by a provider. Service duration and buffers determine the time required for an appointment.

### 3.7 StaffService

- `id`
- `provider_staff_id`
- `provider_service_id`
- `created_at`

Maps employees to services they are qualified to perform.

### 3.8 AvailabilityRule

- `id`
- `provider_staff_id`
- `day_of_week`
- `start_time`
- `end_time`
- `effective_from`
- `effective_until`
- `status`
- `created_at`
- `updated_at`

Stores recurring employee working hours, interpreted using the provider's configured time zone.

### 3.9 AvailabilityException

- `id`
- `provider_staff_id`
- `exception_type`
- `start_at`
- `end_at`
- `reason`
- `created_at`
- `updated_at`

Stores exceptions such as employee leave, temporary unavailability, additional working hours and special working days.

### 3.10 Appointment

- `id`
- `provider_id`
- `customer_user_id`
- `provider_service_id`
- `assigned_provider_staff_id`
- `rebooked_from_appointment_id` — optional self-reference to a previously rejected appointment.
- `start_at`
- `end_at`
- `actual_start_at`
- `actual_end_at`
- `service_name_snapshot`
- `service_description_snapshot`
- `service_duration_minutes_snapshot`
- `service_price_snapshot`
- `service_currency_snapshot`
- `buffer_before_minutes_snapshot`
- `buffer_after_minutes_snapshot`
- `status`
- `customer_notes`
- `created_at`
- `updated_at`

The appointment retains a reference to the original `ProviderService`, while snapshot attributes preserve the service details applicable when the appointment was booked.

Subsequent changes to the provider's service definition must not alter existing appointment snapshots.

`assigned_provider_staff_id` identifies the employee currently assigned to the appointment.

### 3.11 AppointmentStatusHistory

- `id`
- `appointment_id`
- `previous_status`
- `new_status`
- `actor_user_id` — nullable for system-triggered transitions.
- `actor_type`
- `reason`
- `created_at`

Records every appointment status transition for auditing.

### 3.12 AppointmentReschedule

- `id`
- `appointment_id`
- `proposed_start_at`
- `proposed_end_at`
- `proposed_provider_staff_id`
- `requested_by_user_id`
- `response_status`
- `reason`
- `response_deadline_at`
- `responded_at`
- `created_at`

The proposed employee is selected by the provider-side user initiating the reschedule. It may be the same employee currently assigned to the appointment.

The proposed employee must be eligible to perform the service and available at the proposed time.

The original appointment remains confirmed and retains its existing employee assignment and schedule while the proposal is pending.

If the customer accepts:
1. SlotWise revalidates the proposed time and employee.
2. The appointment's scheduled times are updated.
3. The appointment's assigned employee is updated to the proposed employee.
4. The original schedule reservation is released.
5. The proposal is recorded as `ACCEPTED`.

If the customer declines or the response deadline expires, the original appointment remains unchanged and the proposed reservation is released.

A confirmed appointment may have multiple rescheduling proposals over its lifetime, but only one proposal may have `PENDING` status at a time.

Possible response statuses:
- `PENDING`
- `ACCEPTED`
- `DECLINED`
- `EXPIRED`

## 4. Confirmed Design Rules

1. A user can belong to multiple providers.
2. A provider can have multiple employees.
3. A provider can offer multiple services.
4. An employee can perform multiple services.
5. An appointment belongs to one provider and one customer.
6. An appointment is associated with one provider service and one currently assigned employee.
7. The provider decides whether the employee assignment changes during rescheduling.
8. Customers must accept provider-initiated rescheduling proposals.
9. Appointment service details and price are preserved as snapshots at booking time.
10. Multiple rescheduling proposals may exist over an appointment's lifetime, but only one may be pending.
11. Appointment availability is calculated dynamically using schedules, exceptions, service duration, buffers and existing reservations.
12. Appointment status changes must be recorded in `AppointmentStatusHistory`.
13. A rejected appointment remains in the database if the customer creates a replacement booking.
14. Provider-specific appointment policies override platform defaults where configured.
15. All appointment timestamps must be handled consistently with the provider's configured time zone.
16. Temporary booking and rescheduling reservations must be revalidated and protected against concurrent booking conflicts.

## 5. Remaining Design Considerations

- Whether provider-specific appointment policies need versioning or historical snapshots.
- Whether availability exceptions need to support partial-day availability and multiple intervals.
- Whether staff permissions should use predefined permission records rather than a permissions attribute.
- Whether appointment cancellation and rejection reasons require separate structured reason entities.
- How service snapshot values are populated and retained if a provider changes a service before an appointment is confirmed.