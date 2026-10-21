# Setup guide

Prerequisites: a Google account, n8n (self-hosted or cloud) with Gmail and Google Sheets OAuth credentials, and a Google Gemini API key from [aistudio.google.com](https://aistudio.google.com).

## 1. Google Sheet
1. Create a spreadsheet called **Invoice Processor** with a tab named **Invoices**.
2. Paste the header row from `template/invoices-sheet-template.csv`:
   `received_at | vendor | invoice_no | invoice_date | billed_to | subtotal | gst | total | status | file_name`

## 2. Gmail label and filter
1. Create a top-level label **Invoices** (do not nest it).
2. Search `has:attachment filename:pdf subject:invoice` → **Create filter** → tick **Apply the label: Invoices** and **Never send it to Spam**. Leave *Skip the Inbox* and *Mark as read* unticked.

## 3. n8n credentials
Add three credentials: **Gmail OAuth2**, **Google Sheets OAuth2**, and **Google Gemini (PaLM) API** (paste the API key). Never paste the key into a node, a screenshot or a repo.

## 4. Import the workflow
1. **Create workflow → ⋯ → Import from File** → `workflows/ai-invoice-processor.json`.
2. On every node with a credential, re-select your own.
3. **Gmail Trigger** → Filters → Label: pick **Invoices** (only that one label).
4. **Check duplicate** and **Append invoice** → select your **Invoice Processor** sheet and the **Invoices** tab.
5. **Duplicate alert** and **Review alert** → set **To** to the owner's email.
6. **Google Gemini Chat Model** → pick a stable Flash or Flash-Lite model.

## 5. Node reference

| Node | Key settings |
|---|---|
| Gmail Trigger | Every minute · Message Received · **Simplify off** · Label `Invoices` · Download Attachments on |
| Extract from File | Extract From PDF · Input Binary Field `attachment_0` |
| Information Extractor | Text `{{ $json.text }}` · 7 attributes (vendor, invoice_no, invoice_date, billed_to, subtotal, gst, total) |
| Check duplicate | Get Row(s) · filters `invoice_no` and `vendor` · **Settings → Always Output Data: on** |
| Is duplicate? | `{{ $json.invoice_no }}` is not empty → true = duplicate |
| Append invoice | Columns mapped from `$('Information Extractor').item.json.output.*`; `status` = totals check |
| Needs review? | `{{ $json.status }}` equals `needs review` |

Totals check used for `status`:
```
{{ Math.abs(subtotal + gst - total) <= 1 ? 'processed' : 'needs review' }}
```

## 6. Test
Email yourself each file in `samples/` with "Invoice" in the subject, **one at a time**, and click **Execute workflow** after each. Compare with the *Tested scenarios* table in the README. Then **Publish/Activate** the workflow.

> Manual runs only fetch the single newest labelled email. Once active, n8n processes every new email since the last poll.
