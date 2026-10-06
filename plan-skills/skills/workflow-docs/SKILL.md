---
name: workflow-docs
description: Maintain short business-workflow documents in docs/workflows/ as context for humans and AI agents. Use when changing or adding an important business process (booking, cancellation, payment, refund, reminders), when the user says "workflow doc", "document this workflow", or when a feature touches an existing file in docs/workflows/.
---

# Workflow Docs

`docs/workflows/*.md` capture how an important business operation works, which rules must hold, and what can go wrong. Business context, not implementation.

## Before coding

Feature touches a workflow? Read its doc first.

```
Changing appointment cancellation -> docs/workflows/cancellation.md
```

## During coding

Treat **Important Rules** as constraints. Don't break them accidentally.

Code and doc disagree:

1. Investigate which behavior is intended.
2. Never silently pick one.
3. Intended behavior changed? Update the doc.

## After coding

Ask: *Did this change alter an important business workflow?*

- Yes: update the doc.
- No: leave docs alone.

## When to create a doc

Create when:

- Multiple parts of the system are involved.
- Important business rules exist.
- Meaningful failure/edge cases exist.
- An agent could reasonably implement it wrong.
- It will stay important over time.

Examples: booking, cancellation, reminder sending, payment, refund, staff onboarding, customer registration, gift card purchase, loyalty redemption.

Skip simple CRUD.

## Format

A doc answers: trigger, ordered steps, involved entities, invariant rules, failure/cancellation behavior, edge cases.

Target 20-50 lines. Sections optional; use only what helps.

```md
# Booking

## Trigger

Customer submits a booking request.

## Flow

1. Validate the organization.
2. Validate the customer.
3. Check service availability.
4. Check staff availability.
5. Create the appointment.
6. Send confirmation.

## Important Rules

- Availability is evaluated using the organization's IANA timezone.
- Staff must belong to the organization.
- Confirmation is sent only after the appointment is persisted.

## Failure

If booking fails, no appointment remains partially created.

## Related

- Appointment
- Customer
- Service
```

## Content rules

Document **what must happen**, not how code does it.

Good: "A customer can only cancel their own appointment."

Avoid: "Request goes to `POST /api/appointments/:id/cancel`, which calls `cancelAppointment()`."

Mention implementation only for a real architectural constraint.

Don't document every function, endpoint, query, or UI component.
