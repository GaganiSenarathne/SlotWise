# SlotWise — Logical Entity Relationship Diagram

**Version:** 1.0  
**Status:** Draft

## 1. Overview

This diagram represents the logical relationships between SlotWise entities.

It includes provider management, service management, employee eligibility, availability management, appointment booking, appointment lifecycle history and rescheduling.

## 2. Logical ER Diagram

```mermaid
erDiagram

    USER {
        uuid id PK
        string first_name
        string last_name
        string email
        string phone_number
        string password_hash
        string account_status
    }

    PROVIDER {
        uuid id PK
        string provider_type
        string name
        string timezone
        string status
    }

    PROVIDER_STAFF {
        uuid id PK
        uuid provider_id FK
        uuid user_id FK
        string role
        string membership_status
    }

    PROVIDER_APPOINTMENT_POLICY {
        uuid id PK
        uuid provider_id FK
        int approval_timeout_minutes
        int reschedule_response_timeout_minutes
        int customer_cancellation_deadline_minutes
        int no_show_grace_period_minutes
        int completion_grace_period_minutes
    }

    SERVICE_CATEGORY {
        uuid id PK
        string name
        string description
    }

    PROVIDER_SERVICE {
        uuid id PK
        uuid provider_id FK
        uuid category_id FK
        string name
        int duration_minutes
        int buffer_before_minutes
        int buffer_after_minutes
        decimal price
        string currency
    }

    STAFF_SERVICE {
        uuid id PK
        uuid provider_staff_id FK
        uuid provider_service_id FK
    }

    AVAILABILITY_RULE {
        uuid id PK
        uuid provider_staff_id FK
        int day_of_week
        time start_time
        time end_time
    }

    AVAILABILITY_EXCEPTION {
        uuid id PK
        uuid provider_staff_id FK
        string exception_type
        datetime start_at
        datetime end_at
    }

    APPOINTMENT {
        uuid id PK
        uuid provider_id FK
        uuid customer_user_id FK
        uuid provider_service_id FK
        uuid assigned_provider_staff_id FK
        uuid rebooked_from_appointment_id FK
        datetime start_at
        datetime end_at
        datetime actual_start_at
        datetime actual_end_at
        string service_name_snapshot
        decimal service_price_snapshot
        string service_currency_snapshot
        int service_duration_minutes_snapshot
        string status
    }

    APPOINTMENT_STATUS_HISTORY {
        uuid id PK
        uuid appointment_id FK
        uuid actor_user_id FK
        string previous_status
        string new_status
        string actor_type
        string reason
        datetime created_at
    }

    APPOINTMENT_RESCHEDULE {
        uuid id PK
        uuid appointment_id FK
        uuid proposed_provider_staff_id FK
        uuid requested_by_user_id FK
        datetime proposed_start_at
        datetime proposed_end_at
        string response_status
        datetime response_deadline_at
        datetime responded_at
    }

    USER ||--o{ PROVIDER_STAFF : belongs_to
    PROVIDER ||--o{ PROVIDER_STAFF : has_members

    PROVIDER ||--o| PROVIDER_APPOINTMENT_POLICY : has_policy

    PROVIDER ||--o{ PROVIDER_SERVICE : offers
    SERVICE_CATEGORY o|--o{ PROVIDER_SERVICE : categorizes

    PROVIDER_STAFF ||--o{ STAFF_SERVICE : eligible_for
    PROVIDER_SERVICE ||--o{ STAFF_SERVICE : performed_by

    PROVIDER_STAFF ||--o{ AVAILABILITY_RULE : has_schedule
    PROVIDER_STAFF ||--o{ AVAILABILITY_EXCEPTION : has_exceptions

    PROVIDER ||--o{ APPOINTMENT : receives
    USER ||--o{ APPOINTMENT : books
    PROVIDER_SERVICE ||--o{ APPOINTMENT : requested_service
    PROVIDER_STAFF ||--o{ APPOINTMENT : assigned_to

    APPOINTMENT ||--o{ APPOINTMENT_STATUS_HISTORY : records
    USER o|--o{ APPOINTMENT_STATUS_HISTORY : performs

    APPOINTMENT ||--o{ APPOINTMENT_RESCHEDULE : has_proposals
    PROVIDER_STAFF ||--o{ APPOINTMENT_RESCHEDULE : proposed_employee
    USER ||--o{ APPOINTMENT_RESCHEDULE : requests

    APPOINTMENT o|--o{ APPOINTMENT : replacement_for
```

## 3. Relationship Constraints

The logical relationships above imply the following business constraints:

1. A user can have multiple provider memberships.
2. A provider can have multiple employees.
3. A provider has at most one appointment policy record.
4. An employee can be eligible for multiple services.
5. A service can be performed by multiple employees.
6. An appointment must belong to one provider, one customer, one service and one assigned employee.
7. An appointment can have multiple status history records.
8. An appointment can have multiple rescheduling proposals over its lifetime.
9. Only one rescheduling proposal may be pending for an appointment at a time.
10. A rescheduling proposal may assign a different eligible employee.
11. A replacement appointment can reference the rejected appointment from which it originated.

## 4. Important Notes

- Service snapshot fields are stored directly in `APPOINTMENT`; they do not create a separate relationship.
- `APPOINTMENT_RESCHEDULE` stores proposed changes. It does not replace the original appointment record.
- The original appointment's employee and schedule remain unchanged while a reschedule proposal is pending.
- Availability slots are dynamically generated and are not stored as permanent database records.
- Temporary reservations must be protected against concurrent booking requests.
- The 15-minute slot interval is a platform-wide setting, not an entity-specific attribute.

## 5. Next Design Phase

After validating this logical ERD, the next phase is physical database design.

This will define:
- PostgreSQL data types.
- Primary and foreign key constraints.
- Unique constraints.
- Check constraints.
- Indexes.
- Database-level protection against overlapping employee reservations.
- Database enforcement of one pending reschedule per appointment.