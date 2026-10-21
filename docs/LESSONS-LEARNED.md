# Lessons learned

Real problems hit while building this, and the fix for each.

| Problem | Cause | Fix |
|---|---|---|
| Gmail Trigger: "No Gmail data found" | Two labels selected (`Invoices` and an accidental nested `Invoices/INVOICES`); Gmail requires an email to have **all** selected labels | Keep a single top-level label |
| Extract from File found no file | Default input field is `data`; Gmail names attachments `attachment_0` | Set Input Binary Field to `attachment_0` |
| Sheet received the text `output.vendor` instead of the vendor | Field was in Fixed mode, not Expression mode | Use `{{ ... }}` expressions or drag fields from the input panel |
| Values blank after inserting the duplicate check | `$json` now pointed at the lookup result, not the AI output | Reference the AI node by name: `$('Information Extractor').item.json.output.*` |
| Every invoice looked like a duplicate | *Get Row(s)* had no filters, so it returned the whole sheet | Filter on `invoice_no` and `vendor` |
| New invoices silently stopped at the lookup | *Get Row(s)* outputs nothing when there's no match | Settings → **Always Output Data** on |
| A real duplicate wasn't caught | A stray space (`= {{`) stored `" INV-1042"`, which never matches `"INV-1042"` | Remove the leading space; clean old rows |
| Same invoice processed twice, another skipped | Manual **Execute** only fetches the newest labelled email, every time | Test one email at a time; in production the active trigger tracks what it has seen |
| AI step very slow | Free-tier rate limits and a busy preview model | Use a stable Flash-Lite model and space out test runs |
