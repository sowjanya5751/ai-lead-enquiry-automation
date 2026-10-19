# AI Lead & Enquiry Workflow Automation

An AI-powered lead management and workflow automation project built using n8n Cloud, Google Sheets, Google Gemini, and JavaScript.

## 1. Project Overview

The AI Lead & Enquiry Workflow Automation system helps small businesses and service-based organizations manage incoming customer enquiries more efficiently.

The workflow captures lead details through a form, validates the submitted information, creates a unique lead ID, stores the lead in Google Sheets, and uses AI to classify the enquiry, assess its priority, identify missing information, and draft a professional response for human review.

The system combines workflow automation with AI-assisted decision support while keeping customer communication under human control.

## 2. Problem Statement

Businesses often receive customer enquiries that require manual review, categorization, prioritization, and follow-up. Handling these tasks manually can lead to delayed responses, disorganized customer information, difficulty identifying urgent enquiries, inconsistent follow-up messages, and time spent repeatedly reviewing similar enquiries.

## 3. Project Objectives

- Capture customer enquiries through a form.
- Validate required fields and email addresses.
- Normalize submitted information.
- Generate a unique lead ID and timestamp.
- Save original lead information in Google Sheets.
- Classify enquiries using predefined categories.
- Assign an appropriate priority and explain the decision.
- Identify missing customer information.
- Generate professional reply drafts.
- Flag enquiries that require human review.
- Update existing lead records instead of creating duplicate records.
- Handle invalid input and AI-processing failures safely.

## 4. Technology Stack

| Technology | Purpose |
|---|---|
| n8n Cloud | Workflow automation and orchestration |
| n8n Form Trigger | Capturing customer enquiries |
| Google Sheets | Lead storage and tracking |
| Google Gemini | AI classification and reply drafting |
| Structured Output Parser | Defining the expected AI response structure |
| JavaScript | Input validation and AI-output validation |

## 5. Workflow Architecture

1. **On form submission:** Receives customer information.
2. **Validate and Normalize Input:** Checks required fields, validates email format, removes unnecessary whitespace, and rejects messages exceeding 3,000 characters.
3. **Is Input Valid?:** Routes valid and invalid submissions.
4. **Create Lead ID and Timestamp:** Creates tracking information for the lead.
5. **Phase 3 Placeholder - Lead Saved:** Intended to save the original lead record before AI processing.
6. **Basic LLM Chain and Gemini Model:** Process the enquiry using the connected AI model.
7. **Structured Output Parser:** Specifies the expected response structure.
8. **Validate AI Output:** Checks categories, priority values, required fields, and data types.
9. **AI Output Valid?:** Routes valid and invalid AI results.
10. **Google Sheets Update Row:** Writes valid AI results to the corresponding existing lead record.
11. **Handle AI Validation Failure:** Handles invalid AI output.
12. **AI Model Error Handling:** Routes AI model failures to a separate handling path.

The intended design saves the original lead before AI processing and updates the same row using `lead_id`.

## 6. Form Inputs

The enquiry form collects Name, Email, Service, and Message.

The workflow also records a lead ID, timestamp, source, AI-generated classification, priority, priority explanation, missing information, reply draft, processing status, review status, human-review flag, and error note.

## 7. Google Sheets Database

The spreadsheet is named **AI Lead Tracker**, with a worksheet named `Leads`.

The worksheet contains these 16 columns:

`lead_id, created_at, source, name, email, service, original_message, category, priority, priority_reason, missing_information, reply_draft, processing_status, review_status, needs_human_review, error_note`

## 8. AI Classification Logic

### Enquiry categories

- Pricing
- Service Inquiry
- Consultation Request
- Booking or Availability
- Technical Support
- Complaint
- Other

### Priority levels

- **High:** Explicit urgent deadline, serious unresolved complaint, or another clearly urgent business issue.
- **Medium:** Clear service enquiry, pricing request, or consultation request without an explicit urgent deadline.
- **Low:** General information request, vague enquiry, or non-urgent message.

The AI also generates a priority explanation, identifies missing information, drafts a professional response, and determines whether human review is needed.

## 9. Human-in-the-Loop Design

- AI-generated replies remain drafts.
- No automatic customer email or message is sent.
- Ambiguous enquiries can be flagged for review.
- High-priority enquiries can receive a dedicated review status.
- AI output is validated before being written to the lead record.
- Original enquiry information is retained for reference.
- Processing failures should be recorded without inventing AI-generated results.

## 10. Validation and Error Handling

### Input validation

The workflow checks required name, email, service, and message fields; email format; and messages exceeding 3,000 characters.

### AI-output validation

The workflow checks whether the output is an object, whether the category and priority are permitted, whether the priority explanation and reply draft are valid strings, whether missing information is an array of valid strings, and whether `needs_human_review` is a Boolean.

### AI model failures

The AI chain is configured to provide an error output when model processing fails. A separate Google Sheets Update Row node is intended to mark the existing lead for manual review.

**Verification note:** An invalid Gemini API key produced an AI error during testing. The final end-to-end update of the existing Google Sheets row with the AI error status still requires confirmation.

## 11. Testing Summary

| Test | Observed result |
|---|---|
| Valid enquiry | Lead capture and processing tested |
| Invalid email | Input validation rejected the format |
| Message exceeding 3,000 characters | Input validation rejected the message |
| Urgent enquiry | High-priority classification and priority review observed |
| Prompt-injection attempt | Workflow executed and the enquiry was flagged for review |
| Multiple lead submissions | Separate lead records were created and updated |
| Invalid Gemini API key | AI model error was produced and the error branch was available |

The AI error-to-Google-Sheets update remains an outstanding verification item. Final testing should also confirm that the AI prompt receives the correct form data in every execution.

## 12. Setup Instructions

1. Create an n8n Cloud account.
2. Create a Google spreadsheet named `AI Lead Tracker`.
3. Create a worksheet named `Leads` and add the 16 column headers listed above.
4. Configure Google Sheets credentials in n8n.
5. Configure a valid Google Gemini credential.
6. Import the exported n8n workflow JSON.
7. Reconnect credentials as required.
8. Verify the form field names and AI prompt mappings.
9. Confirm that the initial lead row is saved before AI processing.
10. Verify Google Sheets row matching using `lead_id`.
11. Test successful enquiries, invalid inputs, AI-output validation, and AI model failures.
12. Confirm that error-handling paths update the correct existing row.
13. Verify that the workflow does not send customer replies automatically.

Node labels and available options may vary between n8n versions.

## 13. Known Limitations

- The current implementation uses a form as the primary enquiry source.
- AI-generated classifications may require correction.
- Reply drafts require human approval.
- AI model availability and API limits can affect processing.
- The final AI-error-to-Google-Sheets update needs end-to-end verification.
- Form-field mappings and AI inputs must be verified before the workflow is considered production-ready.
- Credentials must be configured separately when the workflow is imported.

## 14. Future Enhancements

- Integrate website contact forms and additional lead sources.
- Add notifications for high-priority enquiries.
- Build a dashboard for lead volume, categories, and conversion tracking.
- Add human approval and follow-up tracking.
- Improve error logging and recovery.
- Add configurable business-specific priority rules.
- Add analytics for response times and lead conversion rates.

## 15. Business Value

This project demonstrates how AI-assisted workflow automation can help businesses organize incoming enquiries, reduce repetitive administrative tasks, identify potentially urgent requests, and prepare consistent reply drafts.

## 16. Portfolio Deliverables

- `ai-lead-enquiry-automation.json` — exported n8n workflow.
- `README.md` — project documentation.
- Workflow overview screenshot.
- Google Sheets lead tracker screenshot.
- AI classification screenshot.
- AI error-handling screenshot.
- Test report with expected and actual results.
- Short project demonstration script.

## 17. Conclusion

The AI Lead & Enquiry Workflow Automation project demonstrates practical implementation of workflow orchestration, structured AI output, data validation, Google Sheets integration, and human-in-the-loop processing. It aims to make lead handling more organized and consistent while ensuring that customer communication remains under human control.

**Primary tools:** n8n Cloud, Google Sheets, Google Gemini, JavaScript.
