# Case Study — AI Client Intake & Human-Approved Follow-up System

## Context

Many client-enquiry processes contain the same repetitive steps: collecting information, identifying key details, preparing a response, reviewing it, sending it, and updating the record afterward.

The goal of this portfolio project was to explore how AI and workflow automation can reduce that repetitive work without giving the AI uncontrolled authority over customer-facing communication.

This was built as a portfolio prototype, not as a live production system for a real client.

---

## Objective

Design an end-to-end workflow that can:

- receive and store a client enquiry,
- extract useful information from unstructured text,
- prepare a structured response draft,
- place the draft into a human-review state,
- require explicit manual approval before sending,
- send the approved response through Gmail,
- update the record after a successful send,
- and record failures for manual review.

A key requirement was to keep the AI useful without allowing it to make unsupported business decisions.

---

## System Design

I separated the automation into two independent workflows.

### Workflow 1 — Intake and AI Preparation

`Webhook → Google Sheets → AI Analysis → JSON Parsing → Google Sheets Update`

The first workflow handles intake and preparation only.

It receives the enquiry, stores the raw data before AI processing, extracts structured information such as budget and timeline where possible, prepares a reply draft, and updates the record for human review.

No outbound email can be sent from this workflow.

This separation was intentional: the AI can prepare information and suggest a response, but it cannot execute the final client-facing action.

---

### Workflow 2 — Human-Approved Sending

`Google Sheets → Approval Filter → Gmail → Status Update`

The second workflow handles outbound communication.

It searches only for records that have already been manually approved and have not yet been sent. Before the Gmail action, the workflow also checks that the required recipient and reply content are present.

After a successful send, the record is updated to `sent` and a timestamp is stored.

If Gmail fails, the record is changed to `send_failed`, the failure is logged, and the workflow stops rather than repeatedly retrying the same message without review.

---

## Human-in-the-Loop Design

The human approval gate is the central safety decision in the project.

The AI is allowed to:

- extract information from the enquiry,
- identify missing or unclear information,
- structure the result,
- and draft a response.

The AI is not allowed to:

- approve or reject a customer,
- decide eligibility,
- invent qualification rules,
- assign lead scores without defined business criteria,
- or send the final email.

This keeps automation focused on preparation and repetitive work while reserving higher-impact decisions for a human reviewer.

---

## Prompt and Input Safety

Customer-provided text is treated as untrusted input.

The AI instructions were designed so that enquiry text cannot override system rules, alter the expected output format, change approval behaviour, or introduce unsupported business claims.

The model is also instructed not to invent prices, availability, eligibility criteria, delivery promises, project feasibility, or company policies that were not provided.

The goal was not only to generate a useful draft, but to reduce the chance of the AI confidently fabricating business information.

---

## Structured AI Output

Rather than returning only free-form text, the AI returns structured JSON.

The structured output includes fields for items such as:

- budget status,
- extracted budget,
- timeline status,
- desired launch date,
- manual-review requirements,
- review reason,
- and the customer-facing draft.

The JSON is parsed before the data is written back into Google Sheets.

This makes the AI output easier to validate, map, and use inside deterministic workflow logic.

---

## Data and Status Model

Google Sheets acts as a lightweight CRM and audit layer.

The workflow tracks the original enquiry, extracted information, AI draft, approval state, response state, timestamps, and send errors.

The most important status transitions are:

`pending_review → approved → sent`

or, if sending fails:

`approved → send_failed → manual review`

The schema also contains `lead_score` and `is_qualified`, but those fields are intentionally not automated.

No real qualification rules were provided for the prototype, so allowing the AI to create its own scoring logic would make the system less reliable rather than more intelligent.

---

## Error Handling

The outbound workflow includes a dedicated Gmail failure branch.

If sending fails, the system:

1. does not mark the email as sent,
2. changes the response status to `send_failed`,
3. records the failure,
4. stops automatic processing for that row,
5. and leaves the next action to human review.

This prevents a failed send from being silently treated as a successful one and avoids uncontrolled automatic retries.

---

## End-to-End Validation

The prototype was tested using controlled test data.

The complete flow was validated as:

`Approved record → Gmail send → Email received → Spreadsheet updated to sent`

The test confirmed that the approval gate, Gmail integration, and status tracking work together correctly.

The repository includes sanitized screenshots of the intake workflow, the approved-send workflow, and the final `approved → sent` record state.

---

## Key Design Decisions

### 1. Save the enquiry before AI processing

The original enquiry is stored first so the raw input is not lost if later AI processing fails.

### 2. Separate preparation from execution

AI preparation and outbound sending are handled in different workflows. This reduces the chance of an AI-generated draft being sent without review.

### 3. Do not invent missing business rules

Lead scoring and qualification are deliberately left unautomated until real business criteria exist.

### 4. Use structured output instead of free-form AI responses

Structured JSON makes the workflow easier to map, test, and control.

### 5. Treat send failures as review events

A failed email is logged and stopped rather than retried indefinitely.

---

## What I Learned

This project reinforced an important principle for me: good AI automation is not about automating every possible step.

The more useful design question is:

**Which tasks should AI assist with, which decisions need clear deterministic rules, and where should a human remain in control?**

Building the project also gave me practical experience with workflow architecture, structured LLM output, prompt safety, data mapping, status-driven automation, error handling, debugging, and end-to-end testing.

---

## Skills Demonstrated

- AI automation design
- Workflow architecture
- Make.com scenario design
- Prompt engineering
- Structured LLM / JSON output
- Human-in-the-loop workflow design
- Google Sheets as a lightweight CRM
- Gmail integration
- Webhook-based intake
- Conditional logic and filters
- Error handling
- Status tracking
- Testing and debugging
- Documentation and portfolio presentation

---

## Production Considerations

This project is intentionally a portfolio prototype.

A production implementation would require additional controls such as authenticated access, real business qualification rules, duplicate protection, stronger observability, retry policies, role-based permissions, monitoring, and potentially a dedicated CRM rather than a spreadsheet.

Those features were kept outside the scope so the prototype could focus on the core architecture: **AI-assisted intake, structured processing, human approval, controlled sending, and traceable workflow state.**
