# Test Report — AI Lead & Enquiry Workflow Automation

## Purpose
Record the tests performed on the n8n workflow and distinguish verified results from checks that still need evidence.

## Test results

| ID | Test | Expected result | Observed result | Status |
|---|---|---|---|---|
| T01 | Submit a valid enquiry | The workflow validates the submission, creates a lead ID/timestamp, saves the lead, and processes it with AI. | Normal lead submissions and AI processing were exercised, including a sample enquiry for Maya Rao. | PASS — based on the tests reported |
| T02 | Submit an invalid email address | Validation rejects the input and records an email-format error. | Validator output showed `is_valid: false` and `email format is invalid`. | PASS |
| T03 | Submit a message longer than 3,000 characters | Validation rejects the input and reports the length limit. | Validator output showed `is_valid: false` and `message exceeds 3000 characters`. | PASS |
| T04 | Submit an urgent/high-priority enquiry | AI classifies the enquiry, assigns High priority where appropriate, and routes it for priority review. | Priya Sharma's urgent enquiry was classified High and shown with `Priority Review`. | PASS — observed example |
| T05 | Submit prompt-injection-style text | The workflow should not blindly follow instructions embedded in the enquiry; the result should remain subject to review. | Sam Lee's prompt-injection test was run and flagged for review. | PASS — observed example; do not treat this as a security guarantee |
| T06 | Submit multiple separate enquiries | Each enquiry should have its own lead ID/row, and AI results should update the matching row rather than create duplicates. | Multiple submissions were tested; separate rows and updates matched by `lead_id` were reported. | PASS — based on the tests reported |
| T07 | Simulate Gemini/API failure | The error branch should mark the existing lead as an AI error and require manual review, without creating a duplicate row. | An invalid API key/model produced an API error and the error branch was reached. The final Google Sheets update for that same failed execution was not conclusively confirmed. | PARTIAL — confirm the final sheet update |
| T08 | Verify AI prompt field mappings | The prompt should contain the submitted name, email, service, and message—not `undefined`. | One execution screenshot showed `Name: undefined`, `Email: undefined`, and similar values for Service/Message. | NEEDS FIX / RETEST |
| T09 | Verify no automatic customer reply is sent | The workflow should only create a draft for human approval. | The described workflow generates a draft and does not include an automatic send step. | PASS — based on the workflow described |
| T10 | Send malformed AI output through validation | Invalid/missing fields should be rejected and routed to the AI-output failure handler. | Validation and failure-routing nodes are configured, but a complete execution proving this branch was not conclusively documented. | CONFIGURED — execution not verified |

## Outstanding actions before presenting as fully verified

1. **Fix and retest AI prompt mappings.** Inspect the item entering `Basic LLM Chain` and use the exact field names present there. The incoming data may use `name`, `email`, `service`, and `original_message`, while the prompt currently references `Name`, `Email`, `Service`, and `Message`. Confirm the prompt preview contains real values.
2. **Retest the AI-model-error path after restoring valid Gemini credentials.** If intentionally simulating failure again, verify that the existing row in `AI Lead Tracker` is updated by `lead_id` with `processing_status = AI Error`, `review_status = Manual Review`, `needs_human_review = true`, and a useful `error_note`.
3. **Optionally test invalid AI output.** Confirm the `AI Output Valid?` false branch reaches `Handle AI Validation Failure` and updates the correct existing lead row.
4. Keep human approval in place. Do not automatically send generated reply drafts to customers.

## Evidence to save in the portfolio folder

- Workflow overview screenshot.
- Google Sheets screenshot showing representative processed rows.
- AI classification/output screenshot.
- Error-branch screenshot, clearly labelled as branch execution evidence if the sheet update is not yet confirmed.
- Exported n8n workflow JSON.

## Reporting standard
Only mark a test as fully passed when the expected end-to-end result is visible. Keep the `PARTIAL`, `NEEDS FIX / RETEST`, and `CONFIGURED — execution not verified` statuses until the missing evidence is captured.
