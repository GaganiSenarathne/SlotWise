# SlotWise — Automatic Staff Assignment Requirements

**Document:** Functional Requirements — Appointment Assignment  
**Version:** 1.0 (Draft)  
**Status:** Proposed

## 1. Automatic Staff Assignment

**FR-ASSIGN-001 — Automatic employee assignment**

The system shall automatically assign an eligible and available staff member when a customer books a service offered by a business provider.

**FR-ASSIGN-002 — Staff-service eligibility**

The system shall only consider staff members who are authorized and qualified to perform the selected service.

**FR-ASSIGN-003 — Availability verification**

The system shall verify that the selected staff member is available for the entire requested appointment duration.

**FR-ASSIGN-004 — Conflict prevention**

The system shall prevent a staff member from being assigned to overlapping confirmed appointments.

**FR-ASSIGN-005 — No eligible employee**

If no eligible staff member is available for the requested time, the system shall reject the booking request and inform the customer that no suitable appointment slot is available.

**FR-ASSIGN-006 — Assignment persistence**

The system shall store the assigned staff member against the appointment record.

**FR-ASSIGN-007 — Assignment consistency**

The system shall ensure that staff assignment and appointment confirmation are performed consistently. A failed assignment shall not result in an incorrectly confirmed appointment.

## 2. Individual Provider Assignment

**FR-ASSIGN-008 — Individual provider scheduling**

For an individual provider, the system shall treat the individual professional as the bookable staff member.

The same availability and appointment conflict rules shall apply to individual providers and business employees.

## 3. Appointment Assignment Constraints

**FR-ASSIGN-009**

The system shall consider staff working schedules, availability exceptions, existing appointments and service eligibility when selecting an employee.

**FR-ASSIGN-010**

The system shall support a configurable assignment strategy. The initial assignment strategy shall be defined during system design.

---

*These requirements are proposed and will be refined with acceptance criteria during detailed requirements analysis.*