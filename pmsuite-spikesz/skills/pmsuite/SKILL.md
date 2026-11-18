---
name: pmsuite
description: Work with Spikesz records in PM Suite (leads, clients, proposals, projects, tasks, invoices). Use when the user asks about the pipeline, clients, projects, tasks or invoices, or asks to add one of these to PM Suite.
---

# PM Suite (Spikesz workspace)

PM Suite is the system of record for Spikesz: CRM, projects, timesheets and invoicing. Your tools depend on which connection this bot has:

- **pmsuite-crm**: list leads, clients, proposals; create leads and proposals.
- **pmsuite-finance**: list clients, projects, invoices; create invoices. (Client creation is deliberately not available here.)
- **pmsuite-projects**: list projects, tasks, clients; create projects and tasks.
- **pmsuite-readonly**: list everything, create nothing (for briefs and reports).

## What the API can and cannot do

- List tools return **recent** records only. They are not a full search or history. If something is not in the recent list, say so; do not conclude it does not exist.
- There is **no update and no delete**. To change or remove a record, tell the user exactly what to change in PM Suite itself.
- Expenses, payments, timesheets and deals are not in the API. Ask the user, or work from exports they give you.
- PM Suite has no public field list yet. If a create call is rejected, read the error: it names the missing or invalid fields. Fix them and preview again. Note the working field set in memory once it succeeds.

## How to create a record

Every record type has two tools: `pmsuite_preview_<type>` (read-only, creates nothing) and `pmsuite_create_<type>` (really writes to PM Suite).

1. Call the list tool and check for an existing record (same client, email, invoice number, project name). Also check any register you keep, because the API only shows recent records.
2. Call `pmsuite_preview_<type>` with the proposed fields. It checks recent records for duplicates and returns the exact payload and a `preview_token`.
3. Show that payload to the user in plain language and ask for approval. For invoices, show client, legal entity, currency, line items, net, VAT/tax, gross, dates and payment terms. Mention any warnings from the preview.
4. Only after approval, call `pmsuite_create_<type>` with the **same** fields and the `preview_token`. If anything changed, preview again and get fresh approval; the create tool refuses a payload that differs from the preview.
5. Verify with the list tool that the record exists and matches the approved payload, then report it as done. Track each item as proposed → approved → created → verified.
6. Read the create result's `status`:
   - `created`: verify as in step 5.
   - `blocked_possible_duplicate` or `blocked_nothing_to_compare`: nothing was sent. Ask the user; use `allow_duplicate: true` only with their explicit confirmation that it is new.
   - `refused_duplicate_check_failed`: nothing was sent because PM Suite could not be checked. Tell the user; try later.
   - `refused_in_progress` or `refused_recent_identical`: an identical create is running or just ran. Do not retry; list and reconcile.
   - `outcome_unknown`: it may or may not exist. **Never retry.** List recent records, reconcile, and tell the user.
   - An HTTP 4xx error: PM Suite rejected it, nothing was created. Fix the named fields and preview again.

## Rules

- Never create an invoice or proposal the user has not approved in this conversation.
- Never put client information from PM Suite into marketing content or outreach to other companies.
- If a tool returns an authorization error, run `pmsuite_auth_test` and tell the user which key or scope to check (PM Suite → Settings → Webhooks → API Keys).
