# AI Invoice Processor

**Reads supplier invoices from email, extracts the key data with AI, logs it to Google Sheets, and catches duplicates and wrong totals before anyone pays them. Built with n8n, Gmail, Google Sheets and Google Gemini.**

Built for small businesses and accounts teams that receive invoices as PDF attachments from many suppliers, each with its own layout, and still type them into a spreadsheet by hand.

📺 **Build series on YouTube:** [AI Invoice Processor playlist](https://www.youtube.com/@MitchellSwapnil)

---

## The business problem

| Pain | Consequence |
|---|---|
| Every invoice is typed into a sheet or accounting tool by hand | Hours of admin every week, typos in amounts |
| Every supplier uses a different layout ("Invoice No", "Bill No", "Ref No") | Simple rules and templates break on the first new supplier |
| Suppliers resend the same invoice | Risk of paying twice |
| Totals on the invoice don't always add up | Overpayments go unnoticed |

---

## What it does

1. **Watches Gmail.** A Gmail filter labels any email with "invoice" in the subject and a PDF attached. n8n checks that label every minute.
2. **Reads the PDF.** The attachment is converted to text.
3. **Extracts the data with AI.** Gemini pulls out vendor, invoice number, date, billed-to, subtotal, GST and total, whatever the layout. Dates are normalised to `YYYY-MM-DD`, amounts to plain numbers.
4. **Blocks duplicates.** Before saving, it looks up the same invoice number from the same vendor in the sheet. If found, nothing is saved and the owner gets a ⚠️ *Duplicate invoice* email.
5. **Checks the maths.** If `subtotal + GST` doesn't equal the total (±₹1 for rounding), the row is saved with status `needs review` and the owner gets a 🔍 *Needs review* email.
6. **Logs everything.** Clean rows land in the **Invoices** sheet with the received time and source file name.

```
Gmail Trigger → Extract from File → Information Extractor (Gemini)
  → Check duplicate → Is duplicate?
       ├─ yes → Duplicate alert (email)
       └─ no  → Append invoice → Needs review?
                                  └─ yes → Review alert (email)
```

---

## Tested scenarios

Sample invoices are in [`samples/`](samples/). All are fictional.

| File | Layout | Expected result |
|---|---|---|
| `1-bluepeak-INV-1042.pdf` | Classic, "Invoice Number" | Row added, `processed` |
| `2-northwind-BT-2026-0877.pdf` | Modern header, "Bill No" | Row added, `processed` |
| `3-kaveri-TX-55310.pdf` | Minimal, "Invoice #", amounts in INR text | Row added, `processed` |
| `4-bluepeak-INV-1042.pdf` | Exact resend of #1 | **Blocked** + duplicate alert |
| `5-ranchi-DN-2209.pdf` | Modern header, delivery note number | Row added, `processed` |
| `6-sunrise-SPW-3318.pdf` | Total deliberately ₹500 too high | Row added, `needs review` + review alert |

---

## Stack

| Part | Tool |
|---|---|
| Orchestration | n8n (self-hosted, Docker) |
| Intake | Gmail (label + filter) |
| AI extraction | n8n Information Extractor + Google Gemini (Flash-Lite) |
| Storage | Google Sheets |
| Alerts | Gmail |

Runs on free tiers: no per-task automation fees, and the Gemini free tier covers typical small-business volumes.

---

## Controls built in

| Control | How |
|---|---|
| Duplicate prevention | Lookup on vendor + invoice number before any write |
| Arithmetic validation | `subtotal + GST = total` check with ₹1 tolerance |
| Exception alerting | Owner is emailed for every blocked or flagged invoice |
| Traceability | Every row records when it arrived and which file it came from |
| Least data | Only invoice fields are stored, never full email bodies |

---

## Repository

| Path | Contents |
|---|---|
| [`workflows/`](workflows/) | n8n workflow export (credentials and IDs removed) |
| [`docs/SETUP.md`](docs/SETUP.md) | Step-by-step setup |
| [`docs/LESSONS-LEARNED.md`](docs/LESSONS-LEARNED.md) | Problems hit during the build and how they were solved |
| [`samples/`](samples/) | Six fictional test invoices |
| [`template/`](template/) | Sheet header template |

---

## Roadmap

- Process emails one at a time with **Loop Over Items**, so several invoices arriving in the same minute (including two copies of the same invoice) are handled safely
- Handle **multiple PDFs** in one email
- Scanned or photographed invoices via AI vision
- Push approved invoices to accounting software
- Weekly summary email of spend by vendor

---

**Author:** Mitchell Swapnil Chatterjee · [LinkedIn](https://www.linkedin.com/in/mitchell-chatterjee-7a042b287) · [YouTube](https://www.youtube.com/@MitchellSwapnil)
