# SlotWise — Provider Management Requirements

**Document:** Functional Requirements — Provider Management  
**Version:** 1.0 (Draft)  
**Status:** Proposed

## 1. Provider Registration and Management

**FR-PROV-001 — Provider registration**

The system shall allow an authenticated user to register as a service provider.

**FR-PROV-002 — Provider type selection**

The system shall support two provider types:

- Individual provider
- Business provider

**FR-PROV-003 — Provider profile management**

The system shall allow authorized provider users to create, view and update provider profile information.

**FR-PROV-004 — Provider services**

The system shall allow providers to create, update, activate and deactivate the services they offer.

Each service shall have a name, description, duration and price.

## 2. Staff Management

**FR-STAFF-001 — Staff registration**

The system shall allow an authorized business owner or administrator to add staff members to a business provider.

**FR-STAFF-002 — Staff association**

The system shall associate each staff member with a provider account.

**FR-STAFF-003 — Staff availability**

The system shall allow authorized users to define and manage the availability of bookable staff members.

**FR-STAFF-004 — Staff permissions**

The system shall restrict staff management operations according to the user's permissions within the provider account.

**FR-STAFF-005 — Individual provider representation**

An individual provider shall be represented by a provider account with its owner associated as a bookable provider staff member.

## 3. Appointment Management

**FR-BOOK-001 — Appointment booking**

The system shall allow customers to book an appointment with an individual provider or a business provider.

**FR-BOOK-002 — Staff assignment**

The system shall support assigning an appointment to an eligible staff member.

**FR-BOOK-003 — Availability validation**

The system shall validate provider staff availability before confirming an appointment.

**FR-BOOK-004 — Booking conflict prevention**

The system shall prevent overlapping confirmed appointments for the same bookable staff member.

**FR-BOOK-005 — Appointment lifecycle**

The system shall support appointment statuses, including pending, confirmed, cancelled and completed. Additional statuses may be defined during detailed requirements analysis.

## 4. Authorization and Access Control

**FR-AUTH-001**

The system shall distinguish platform-level roles from provider-level permissions.

**FR-AUTH-002**

Only authorized provider owners and administrators shall be allowed to manage provider profiles and staff membership.

**FR-AUTH-003**

Staff members shall only be able to access operations permitted by their provider-level permissions.

**FR-AUTH-004**

Customers shall only be able to manage their own appointments, subject to the defined business rules.

---

*Note: These are proposed functional requirements. Their acceptance criteria, priorities and detailed business rules will be refined before implementation.*