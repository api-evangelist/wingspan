---
name: Invite and pay a contractor with Wingspan
description: Onboard a contractor (collaborator) and pay them by creating a payable and
  executing the approved payroll run.
api: openapi/wingspan-payments-openapi-original.yml
operations: [createCollaborator, getCollaborator, createPayable, getPayable, getApprovedPayables, executeApprovedPayrollTransaction]
generated: '2026-07-21'
method: generated
---

# Invite and pay a contractor with Wingspan

Ground rules (from `conventions/wingspan-conventions.yml` and `errors/wingspan-error-codes.yml`):

- Auth: every call sends `Authorization: Bearer <API Token>` (token generated in-app under Data & Integrations > Developer; separate tokens per environment).
- Base URL: `https://api.wingspan.app` (production) or `https://stagingapi.wingspan.app` (staging — use staging first).
- No idempotency-key contract exists: do NOT blindly retry POSTs; on a timeout, list the resource first to check whether it was created.
- Rate limits: 200 GET/s, 100 write/s per account; back off with jittered exponential retry on `429`.
- Errors arrive as `{"error": "<message>"}`; a `403` "Session expired" means regenerate the API token.

## Steps

1. **Invite the contractor** — `createCollaborator` with the contractor's email and your `clientId`. Wingspan sends the onboarding invite (W-9, payout setup).
2. **Confirm onboarding state** — `getCollaborator` by id; the contractor must complete eligibility/document requirements before they can be paid (watch `MemberClient.Active` via the webhook surface in `asyncapi/wingspan-webhooks.yml`).
3. **Create the payment obligation** — `createPayable` for the collaborator on behalf of the client (amount, line items, due date).
4. **Verify** — `getPayable` to confirm status and amounts.
5. **Approve + run payroll** — once payables are approved, check `getApprovedPayables`, then `executeApprovedPayrollTransaction` to execute the approved payroll run. This moves real money in production — confirm with a human before executing outside staging.
6. **Track settlement** — subscribe to `Paid`, `Deposited`, `PaymentFailed`, `PayoutFailed` webhook events rather than polling.
