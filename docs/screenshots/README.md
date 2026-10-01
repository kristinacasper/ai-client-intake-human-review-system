# Screenshot Evidence Guide

This folder is reserved for visual evidence of the working portfolio prototype.

Use these exact filenames when adding screenshots:

1. `01-v2-intake-scenario.png`
   - Make.com V2 scenario overview.
   - Show the full flow: Webhook → Google Sheets → AI Agent → JSON Parse → Google Sheets Update.

2. `02-v3-approval-scenario.png`
   - Make.com V3 scenario overview.
   - Show Google Sheets search → Gmail send → sent-status update.
   - Include the Gmail error-handler branch if visible.

3. `03-human-approval-filter.png`
   - Show the V3 condition that requires `approval_status = approved` and `response_status = not_sent`.

4. `04-crm-sheet-review.png`
   - Google Sheet showing the relevant columns such as `ai_reply_draft`, `approval_status`, `response_status`, `sent_at_utc`, and `send_error`.
   - Use only controlled test data. Hide or blur any private data before publishing.

5. `05-successful-test.png`
   - Evidence of the successful controlled test, for example the test row marked `sent` with its timestamp.

6. `06-received-test-email.png`
   - Optional proof that the approved test email was received.
   - Hide personal email addresses or other private information before publishing.

## Privacy rule

Never publish API keys, webhook URLs, OAuth details, connection IDs, private email addresses, access tokens, customer data, or other credentials in screenshots.

The repository should demonstrate architecture and workflow behaviour without exposing secrets.
