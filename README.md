# AI Lead & Enquiry Workflow Automation

An AI-powered lead management and workflow automation project built using n8n Cloud, Google Sheets, Google Gemini, and JavaScript.

## 1. Project Overview

The AI Lead & Enquiry Workflow Automation system helps small businesses and service-based organizations manage incoming customer enquiries more efficiently.

The workflow captures lead details through a form, validates the submitted information, generates a unique lead ID, stores the original enquiry in Google Sheets, and uses AI to classify the enquiry, assess its priority, identify missing information, and draft a professional response for human review.

The system combines workflow automation with AI-assisted decision support while keeping customer communication under human control.

## 2. Problem Statement

Businesses often receive customer enquiries that require manual review, categorization, prioritization, and follow-up. Handling these tasks manually can lead to delayed responses, disorganized customer information, difficulty identifying urgent enquiries, inconsistent follow-up messages, and time spent repeatedly reviewing similar enquiries.

This project demonstrates how workflow automation and AI-assisted classification can reduce repetitive administrative work while retaining human oversight.

## 3. Project Objectives

- Capture customer enquiries through a form.
- Validate required fields and email addresses.
- Normalize submitted information.
- Generate a unique lead ID and timestamp.
- Save original lead information in Google Sheets before AI processing.
- Classify enquiries using predefined categories.
- Assign an appropriate priority and explain the decision.
- Identify missing customer information.
- Generate professional reply drafts.
- Flag enquiries that require human review.
- Update existing lead records using `lead_id`.
- Handle invalid input and AI-processing failures through separate workflow paths.
- Validate AI output before updating the lead record.

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

The workflow is organized into the following stages:

1. **On form submission:** Receives customer information.
2. **Validate and Normalize Input:** Checks required fields, validates email format, normalizes whitespace, and rejects messages exceeding 3,000 characters.
3. **Is Input Valid:** Routes valid submissions toward processing and invalid submissions to a validation-failure path.
4. **Create Lead ID and Timestamp:** Generates lead-tracking information.
5. **Save Initial Lead to Lead Tracker:** Appends the original enquiry to Google Sheets before AI processing.
6. **Basic LLM Chain and Gemini Model:** Analyze the enquiry using the configured AI model.
7. **Structured Output Parser:** Defines the expected structure of the AI response.
8. **Validate AI Output:** Checks the AI-generated fields, permitted category and priority values, and required data types.
9. **AI Output Valid?:** Routes validated results toward the successful update path and invalid results toward manual-review handling.
10. **Update Lead with AI Classification:** Updates the existing lead record with classification, priority, explanation, missing information, reply draft, and review status.
11. **Handle AI Validation Failure:** Prepares invalid AI results for manual review.
12. **Update Lead - AI Error or Manual Review:** Provides a Google Sheets update path for AI-processing errors and handled validation failures.

The intended design saves the original enquiry first and updates the same row using `lead_id`. The error-handling paths must be tested end to end before the workflow is considered production-ready.

## 6. Form Inputs

The enquiry form collects:

- Name
- Email
- Service
- Message

The workflow also tracks the lead ID, creation timestamp, source, AI-generated classification, priority, priority explanation, missing information, reply draft, processing status, review status, human-review flag, and error note.

## 7. Google Sheets Database

The workflow is designed to use a Google spreadsheet named **Lead Tracker**, with a worksheet named `Leads`.

The worksheet contains these 16 columns:

`lead_id, created_at, source, name, email, service, original_message, category, priority, priority_reason, missing_information, reply_draft, processing_status, review_status, needs_human_review, error_note`

The initial save operation appends a new lead record. Subsequent processing is intended to update the corresponding record by matching `lead_id`, avoiding duplicate rows during normal processing.

## 8. AI Classification Logic

### Enquiry categories

The AI classifies enquiries into one of seven categories:

- Pricing
- Service Inquiry
- Consultation Request
- Booking or Availability
- Technical Support
- Complaint
- Other

### Priority levels

- **High:** An explicit urgent deadline, a serious unresolved complaint, or another clearly urgent business issue.
- **Medium:** Clear interest in a service, a pricing request, or a consultation request without an explicit urgent deadline.
- **Low:** A general information request, vague enquiry, or non-urgent message.

The AI also generates a priority explanation, identifies missing information, drafts a professional response, and determines whether human review is required.

Customer-provided instructions that attempt to manipulate classification or override the AI's rules are treated as untrusted input. Such attempts should trigger the human-review flag.

## 9. Human-in-the-Loop Design

Human oversight is built into the workflow:

- AI-generated replies remain drafts.
- No automatic customer email or message is sent.
- Ambiguous enquiries can be flagged for review.
- High-priority enquiries can receive a dedicated review status.
- AI output is validated before the successful classification update.
- Original enquiry information is retained for reference.
- Processing failures should be recorded without inventing AI-generated results.
- Human review is required when classification is uncertain or processing fails.

## 10. Validation and Error Handling

### Input validation

The workflow checks that the name, email, service, and message fields are populated. It validates email format, normalizes whitespace, and rejects messages exceeding 3,000 characters.

Invalid submissions are routed to the validation-failure path rather than continuing through normal lead processing.

### AI-output validation

The validation logic checks:

- The AI output is an object.
- The category belongs to the permitted category list.
- The priority is `High`, `Medium`, or `Low`.
- `priority_reason` is a valid, non-empty string.
- `missing_information` is an array containing valid strings.
- `reply_draft` is a valid, non-empty string.
- `needs_human_review` is a Boolean.

Invalid AI output is routed toward manual-review handling rather than the successful classification update.

### AI model failures

The workflow template includes a separate AI error-handling route intended to update the existing lead record with an error status and manual-review information.

**Verification status:** An invalid Gemini API key previously produced an AI-processing error during testing. The complete error-to-Google-Sheets update must still be confirmed in a separate test workflow. The existence of an error branch alone does not prove that the correct existing row is updated successfully.

## 11. Testing Summary

The following results have been reported from workflow testing:

| Test | Observed result / status |
|---|---|
| Valid enquiry | Lead capture and processing tested |
| Invalid email | Input validation rejected the format |
| Message exceeding 3,000 characters | Input validation rejected the message |
| Urgent enquiry | High-priority classification and priority review observed |
| Prompt-injection attempt | Workflow executed and the enquiry was flagged for review |
| Multiple lead submissions | Separate lead records were created and updated |
| Invalid Gemini API key | AI model error was produced |
| AI error to Google Sheets update | End-to-end confirmation still required |
| Correct AI prompt inputs for every execution | Final verification required |

These results describe the reported tests, not a guarantee that every path in the sanitized template has been independently re-executed. Refer to `TEST_REPORT.md` for the detailed test record.

## 12. Setup Instructions

The public workflow JSON is a sanitized template. It is not intended to run immediately after import without configuration.

1. Create an n8n Cloud account.
2. Create a Google spreadsheet named `Lead Tracker`.
3. Create a worksheet named `Leads`.
4. Add the 16 column headers listed in Section 7.
5. Import `ai-lead-enquiry-automation.json` into a separate test workflow.
6. Configure your own Google Sheets credentials.
7. Configure your own Google Gemini credentials.
8. Select your spreadsheet and `Leads` worksheet in the relevant Google Sheets nodes.
9. Verify all Google Sheets column mappings and `lead_id` matching.
10. Confirm that the initial lead record is saved before AI processing.
11. Verify that the AI prompt receives the correct name, email, service, and message from the submitted form.
12. Test valid enquiries, invalid inputs, AI-output validation failures, and AI model errors.
13. Confirm that error-handling paths update the correct existing row.
14. Keep the workflow inactive until configuration and testing are complete.
15. Verify that the workflow does not send customer replies automatically.

Node labels and available options may vary between n8n versions. Credentials and spreadsheet configuration must be supplied by the person importing the template.

## 13. Known Limitations

- The current implementation uses a form as the primary enquiry source.
- AI-generated classifications may require correction.
- Reply drafts require human approval.
- AI model availability and API limits can affect processing.
- The AI error-to-Google-Sheets update requires end-to-end verification.
- Form-field mappings and AI inputs must be verified before production use.
- The sanitized workflow requires the user's own credentials and spreadsheet configuration.
- The included invalid-email test node is intended for testing, not normal customer submissions.

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

The design emphasizes traceable lead records, structured AI output, input validation, and human oversight rather than fully autonomous customer communication.

## 16. Portfolio Deliverables

- `ai-lead-enquiry-automation.json` — sanitized n8n workflow template.
- `README.md` — project documentation.
- Workflow overview screenshot.
- Google Sheets lead tracker screenshot.
- AI classification screenshot.
- `TEST_REPORT.md` — test cases and reported results.

Additional screenshots or a demonstration video can be added after the corresponding workflow paths have been verified.

## 17. Conclusion

The AI Lead & Enquiry Workflow Automation project demonstrates practical use of workflow orchestration, structured AI output, data validation, Google Sheets integration, and human-in-the-loop processing.

It aims to make lead handling more organized and consistent while keeping customer communication under human control.

**Primary tools:** n8n Cloud, Google Sheets, Google Gemini, JavaScript.
