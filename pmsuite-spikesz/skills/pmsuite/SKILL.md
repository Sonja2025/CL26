---
name: pmsuite
description: Work with Spikesz records in PM Suite (leads, clients, proposals, projects, tasks, invoices). Use when the user asks about the pipeline, clients, projects, tasks or invoices, or asks to add one of these to PM Suite.
---

# PM Suite (Spikesz workspace)

PM Suite is the system of record for Spikesz: CRM, projects, timesheets and invoicing. Your tools depend on which connection this bot has:

- **pmsuite-crm**: list leads, clients, proposals; create leads and proposals.
- **pmsuite-finance**: list clients, projects, invoices; create invoices and clients.
- **pmsuite-projects**: list projects, tasks, clients; create projects and tasks.
- **pmsuite-readonly**: list everything, create nothing (for briefs and reports).

## What the API can and cannot do

- List tools return **recent** records only. They are not a full search or history. If something is not in the recent list, say so; do not conclude it does not exist.
- There is **no update and no delete**. To change or remove a record, tell the user exactly what to change in PM Suite itself.
- Expenses, payments, timesheets and deals are not in the API. Ask the user, or work from exports they give you.
- PM Suite has no public field list yet. If a create call is rejected, read the error: it names the missing or invalid fields. Fix them and dry-run again. Note the working field set in memory once it succeeds.

## How to create a record

1. Call the list tool first and check for an existing record (same client, email, invoice number, project name).
2. Call the create tool with `dry_run` left at its default (true). It sends nothing and shows the exact payload.
3. Show that payload to the user in plain language and ask for approval. For invoices, show client, legal entity (UAE or US company), currency, line items, amounts, VAT/tax, dates and payment terms.
4. Only after approval, call again with `dry_run: false`.
5. Verify with the list tool that the record now exists, then report it as done. Track each item as drafted → approved → created → verified.
6. If a create call fails or times out, list recent records to check whether it was created **before** retrying. Never retry blindly.

## Rules

- Never create an invoice or proposal the user has not approved in this conversation.
- Never put client information from PM Suite into marketing content or outreach to other companies.
- If a tool returns an authorization error, run `pmsuite_auth_test` and tell the user which key or scope to check (PM Suite → Settings → Webhooks → API Keys).
