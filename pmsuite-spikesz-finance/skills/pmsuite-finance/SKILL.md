---
name: pmsuite-finance
description: Work with Spikesz invoices, clients and projects in PM Suite through the pmsuite-finance connection. Use when the user asks about invoices, billing, clients or projects for invoicing, or asks to prepare an invoice in PM Suite.
---

# PM Suite finance connection (Spikesz workspace)

PM Suite is the system of record for Spikesz invoicing. This plugin has one connection, **pmsuite-finance**:

- `pmsuite_list_clients`, `pmsuite_list_projects`, `pmsuite_list_invoices`: recent records, read-only.
- `pmsuite_preview_invoice`: read-only; checks recent invoices for duplicates and returns the exact payload and a `preview_token`.
- `pmsuite_create_invoice`: really writes an invoice to PM Suite.
- `pmsuite_auth_test`: checks the key.

There is no client creation, no update and no delete. If you ever see a `dry_run` parameter or a client-creation tool, you are connected to an old version: stop and tell the user.

## What the API can and cannot do

- List tools return **recent** records only. If something is not in the recent list, say so; do not conclude it does not exist.
- Payments, expenses and timesheets are not in the API. Ask the user, or work from exports they give you.
- If a create call is rejected, read the error: it names the missing or invalid fields. Fix them and preview again.

## How to create an invoice

1. Check the bot's invoice registers and the recent PM Suite invoices for an existing invoice (entity, customer, contract, billing period or milestone, invoice number).
2. Call `pmsuite_preview_invoice` with the proposed fields.
3. Show the user the payload in plain language: client, legal entity, currency, line items, net, VAT/tax, gross, dates, payment terms and any warnings. Ask for approval.
4. Only after approval, call `pmsuite_create_invoice` with the **same** fields and the `preview_token`. If anything changed, preview again and get fresh approval.
5. Read the result's `status`:
   - `created`: list invoices, confirm every field matches the approved payload, then report it as done.
   - `blocked_possible_duplicate` or `blocked_nothing_to_compare`: nothing was sent. Ask the user; use `allow_duplicate: true` only with their explicit confirmation that it is new.
   - `refused_duplicate_check_failed`: nothing was sent. Tell the user; try later.
   - `refused_in_progress` or `refused_recent_identical`: do not retry; list and reconcile.
   - `outcome_unknown`: **never retry.** List recent invoices, reconcile, and tell the user.
   - An HTTP 4xx error: nothing was created. Fix the named fields and preview again.

## Rules

- Never create an invoice the user has not approved in this conversation.
- Never send invoices, reminders or emails to clients.
- If a tool returns an authorization error, run `pmsuite_auth_test` and tell the user which key or scope to check (PM Suite → Settings → Webhooks → API Keys).
