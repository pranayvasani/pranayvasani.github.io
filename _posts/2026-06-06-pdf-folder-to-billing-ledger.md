---
tags: [AI, Personal Finance, Claude, Cowork]
---

# From a folder of PDFs to a reconciled billing ledger — an AI-assisted audit

I had a folder of gas utility bills going back a few years. The kind of folder you know exists but never open. At some point I wanted to know: *am I up to date, or do I quietly owe money?*

The naive answer — subtract what I paid from what was invoiced — gave a number that looked alarming. It turned out to be completely wrong.

---

## Step 1: Extraction

The first task was straightforward: read every PDF and pull out the structured fields — bill date, billing period, consumption in standard cubic metres, the line-item breakdown (gas, tax, arrears, other charges), and the total due. One row per bill, tabulated in a spreadsheet.

A few bills were missing from the folder. Utility bills carry a "Bill Detail History" section showing the prior two bills' charges — enough to reconstruct the missing ones from context.

## Step 2: The arrears trap

Here's where the naive arithmetic breaks down. Utility billing often has a carry-forward mechanism: if a bill goes unpaid, the unpaid amount reappears as "Arrears" on the next bill. When you pay that next (larger) bill, you've cleared the prior one too.

Running a simple sum of `(invoice − payment)` across all bills double-counts the arrears. The debt that appeared on bill N was already included in bill N+1's total — paying N+1 settled both. The actual outstanding balance was a single unpaid bill, not a multi-year accumulation.

Tracing this required reconstructing the arrears chain: which bill's unpaid portion flowed into which successor, and whether that successor was ultimately paid.

## Step 3: Cross-referencing three sources

The billing picture wasn't complete from PDFs alone. I pulled in two more sources:

- **Portal payment history** — the utility's own payment log, with receipt IDs and amounts
- **Gmail payment confirmations** — transactional emails from the utility's no-reply address

Stacking all three side by side revealed a failed payment that the portal had marked "pending" for months with no corresponding email confirmation — and the actual replacement payment that settled the bill shortly after, which did have a Gmail receipt.

A handful of bills also had ₹1 discrepancies between the invoice and what was paid. The utility accepted all of them as settled, so no arrears propagated — a good sign.

## Step 4: Monthly consumption

Billing periods don't align to calendar months. A single bill might span six weeks, straddling two months. To get a clean month-by-month view of consumption and spend, each billing period was prorated by day-weighting: divide the period's total by the number of days, then allocate each day's share to its calendar month.

This produced a tidy time series — consumption in SCM and rupees for every month in the dataset — with the expected seasonal pattern (higher in winter, lower in summer).

---

## What the AI actually did

Every step above — PDF extraction, arrears chain reconstruction, payment reconciliation, day-weighted proration, spreadsheet construction with charts — was done conversationally. I described what I wanted, reviewed the output, corrected mistakes, and asked follow-up questions.

The interesting moments weren't the automation. They were the reasoning: explaining *why* the naive outstanding balance was wrong, tracing which arrears chain cleared which prior bill, identifying the failed payment from the absence of a Gmail confirmation. Those required understanding the billing structure, not just parsing fields.

The spreadsheet at the end has every bill, every payment source, and every discrepancy documented in one place. The entire reconciliation — years of billing history, untouched until today — was sorted out in a single session using Cowork.
