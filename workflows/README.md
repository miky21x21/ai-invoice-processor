# n8n workflow

| File | Workflow |
|---|---|
| `ai-invoice-processor.json` | Gmail → PDF text → AI extraction → duplicate and totals checks → Google Sheets + email alerts |

## Importing
n8n → **Create workflow → ⋯ → Import from File**, then:
1. Re-select your own Gmail, Google Sheets and Google Gemini credentials (the export contains `REPLACE_WITH_YOUR_CREDENTIAL`).
2. Point both Google Sheets nodes at your spreadsheet (the export contains `YOUR_SHEET_ID`).
3. In **Gmail Trigger**, select your **Invoices** label (the export contains `YOUR_INVOICES_LABEL_ID`).
4. Set the recipient on **Duplicate alert** and **Review alert** (the export contains `you@example.com`).

See [`../docs/SETUP.md`](../docs/SETUP.md) for the full guide.
