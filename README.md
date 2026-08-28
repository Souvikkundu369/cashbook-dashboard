# Cashbook Dashboard — Case Study

> A per-store daily cash-closing system with a consolidated HO view — replacing WhatsApp photos of register printouts with a structured, auditable record every store submits before close.

<p>
  <img src="https://img.shields.io/badge/role-Sole%20Builder-orange" alt="Sole Builder">
  <img src="https://img.shields.io/badge/stores-25%2B-blue" alt="25+ stores">
  <img src="https://img.shields.io/badge/Google%20Apps%20Script-34A853?style=flat&logo=google&logoColor=white" alt="Apps Script">
  <img src="https://img.shields.io/badge/JavaScript-F7DF1E?style=flat&logo=javascript&logoColor=black" alt="JavaScript">
</p>

> 🔒 **Why this is a case study, not full source.** This system is embedded in live store operations and linked to real financial data. Production scripts and sheet IDs are kept private.

---

## The problem

Each of 25+ entertainment outlets closes its register daily. The information needed at head office:

- How much cash is in the till?
- What did they deposit to the bank?
- What were Card / UPI / excess payments?

**Before this system:** store managers would photograph their register printout and post it in a WhatsApp group. HO would manually transcribe numbers from blurry photos into a spreadsheet. Errors were frequent. Missing submissions were discovered at month-end. No store could query its own history.

---

## What I built

### Store-side: daily closing form
Each store gets a unique URL. The form captures:

| Field | Definition |
|---|---|
| Card Total | Aggregate card payments received |
| Cash Total | Total cash collected |
| UPI Total | UPI/QR payments |
| Excess | Overage vs expected (common with round-up tips) |
| Deposits | Cash physically deposited to bank today |
| Notes | Anything unusual (machine down, event, etc.) |

**Grand = Card + Cash + UPI + Excess**  
**Cash in Hand = Cash Total − Deposits**

Validation runs client-side before submission — the form won't submit if the numbers don't balance to the till total.

### Backend: Google Apps Script
Submissions write to a central Google Sheet via a `doPost` webhook. Each row is timestamped, store-tagged, and flagged if submitted after the cutoff window. Late submissions are auto-highlighted for HO review.

### HO dashboard
A consolidated HTML dashboard reads from the Sheet and shows:

- All stores for today — submitted / not submitted
- Per-store drill-down: every metric vs the prior 7 days
- Cash-in-hand running total across all stores
- Deposit tracking: did the cash that was collected actually hit the bank?
- Alerts: stores not submitted by close time, unusual variance in any metric

---

## Architecture

```
Store manager → per-store HTML form
                  → Apps Script doPost webhook
                  → Central Google Sheet
                        ↓
              HO Dashboard (reads Sheet)
                  ├── Daily summary grid
                  ├── Per-store history
                  └── Variance alerts
```

**Why Google Apps Script + Sheets, not a database?**

The finance team already lives in Google Sheets. A database would require a separate login, separate training, and a separate backup strategy. Apps Script lets me deploy server-side logic with zero infrastructure — the Sheet *is* the database, with a 10-year audit trail for free.

**Why per-store links, not one login screen?**

Store managers close the register after a full shift. A login screen adds friction at exactly the wrong moment. Each store has its own bookmarked URL — open, fill, submit. No password to forget, no account to create.

---

## Key engineering decisions

**Client-side validation before submission**

The form validates that Grand = Card + Cash + UPI + Excess before sending. If a store manager accidentally transposes a digit, they see an error immediately rather than discovering the discrepancy during HO review three days later.

**Cutoff-window flagging**

Each store's submission is compared against a configurable cutoff time. Submissions after midnight are flagged — not rejected, but highlighted in HO view so finance can follow up if the pattern is consistent.

**Immutable submission log**

Each submission appends a new row — no edits, no deletes. If a store needs to correct an error, they submit again with a note, and HO can see the correction alongside the original. This gives an auditable trail that a single editable cell would destroy.

---

## Impact

- Cash closing time: from 10–15 minutes (photo + WhatsApp + HO transcription) to **~2 minutes** (form + submit)
- HO visibility: from "chase managers on WhatsApp" to **live dashboard open in one tab**
- Month-end reconciliation: deposit tracking shows which stores' cash hit the bank vs which is still outstanding
- Missing submissions surface **same day**, not at month-end

---

## Tech stack

`Google Apps Script` · `Google Sheets` · `JavaScript` · `HTML/CSS` · `Webhook (doPost)` · `Apps Script Web App`

---

### Author

**Souvik Kundu** — Business Intelligence & Automation Engineer.

📫 [LinkedIn](https://linkedin.com/in/souvik-kundu-bi) · [GitHub](https://github.com/Souvikkundu369)
