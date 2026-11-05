
erDiagram
USER ||--o{ PROVIDER_STAFF : joins
PROVIDER ||--o{ PROVIDER_STAFF : has_members
PROVIDER ||--o{ PROVIDER_SERVICE : offers
SERVICE_CATEGORY ||--o{ PROVIDER_SERVICE : classifies
PROVIDER_STAFF ||--o{ AVAILABILITY_RULE : defines
PROVIDER_STAFF ||--o{ AVAILABILITY_EXCEPTION : has
USER ||--o{ APPOINTMENT : books
PROVIDER ||--o{ APPOINTMENT : receives
PROVIDER_SERVICE ||--o{ APPOINTMENT : selected_for
PROVIDER_STAFF o|--o{ APPOINTMENT : assigned_to

    USER {
        bigint id PK
        string email
        string password_hash
    }

    PROVIDER {
        bigint id PK
        string name
        string provider_type
        string description
        string status
    }

    PROVIDER_STAFF {
        bigint id PK
        bigint user_id FK
        bigint provider_id FK
        string membership_role
        boolean is_bookable
    }

    SERVICE_CATEGORY {
        bigint id PK
        string name
    }

    PROVIDER_SERVICE {
        bigint id PK
        bigint provider_id FK
        bigint category_id FK
        string name
        int duration_minutes
        decimal price
        string status
    }

    AVAILABILITY_RULE {
        bigint id PK
        bigint provider_staff_id FK
        int day_of_week
        time start_time
        time end_time
    }

    AVAILABILITY_EXCEPTION {
        bigint id PK
        bigint provider_staff_id FK
        datetime start_at
        datetime end_at
        string exception_type
    }

    APPOINTMENT {
        bigint id PK
        bigint customer_id FK
        bigint provider_id FK
        bigint provider_service_id FK
        bigint provider_staff_id FK
        datetime start_at
        datetime end_at
        string status
    }
