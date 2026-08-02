---
name: Manage 1099 tax forms for contractors
description: Calculate 1099 amounts, create and retrieve 1099 tax forms, deliver PDFs, and
  handle re-mails for a tax year.
api: openapi/wingspan-payments-openapi-original.yml
operations: [calculate1099Amounts, list1099TaxForms, create1099TaxForm, getTaxForm, updateTaxForm, getForm1099PDF, remail1099Form, request1099InviteEmail]
generated: '2026-07-21'
method: generated
---

# Manage 1099 tax forms for contractors

Ground rules: bearer-token auth per environment; tax data is sensitive — prefer staging for
integration tests and never log form payloads. Errors use the `{"error": "..."}` envelope.

## Steps

1. **Calculate** — `calculate1099Amounts` to compute 1099 totals for collaborators for the tax year.
2. **Create forms** — `create1099TaxForm` per recipient; then `list1099TaxForms` to enumerate what exists for the year.
3. **Review / correct** — `getTaxForm` to inspect a form; `updateTaxForm` for corrections (W-9 data changes flow through `updatePayeeW9Information` on the payee side).
4. **Deliver** — `getForm1099PDF` to fetch the filed PDF; `remail1099Form` when a recipient reports non-delivery, and `request1099InviteEmail` to re-invite recipients who have not claimed their forms.
5. **Track document events** — `Document.MemberSigned` / `Document.Completed` webhook events signal signature-requirement completion (see `asyncapi/wingspan-webhooks.yml`).
