# Property Inbox AI Classifier

An n8n workflow that reads incoming property management emails directly from an inbox, classifies them with AI, drafts a reply, notifies the right team, and logs every ticket, with fallback handling if any step fails.

![Workflow Diagram](./Property%20Inbox%20AI%20Classifier.png)

## The Problem

A tenant reports no heating at 11pm. By the time someone on the property team checks the inbox the next morning, hours have already passed. The message sits behind a dozen other emails and nobody knows it needs urgent attention until someone happens to open it. Most property teams don't have someone dedicated to watching the inbox around the clock, they have one person handling this between everything else on their plate. That either slows down response times or turns into hours of manual sorting every week.

## What This Workflow Does

1. **Email Trigger (IMAP).** Reads new emails directly from a live inbox as they arrive, and marks them as read once processed so nothing gets picked up twice.
2. **Sanitize Email Body.** Long emails are trimmed to a safe length before being sent to the AI, keeping token usage predictable. The original message is never altered or lost, only what's sent to the AI is shortened.
3. **AI Classification (Groq).** The email is classified into Maintenance, Tenant Query, Viewing, or Enquiry, given an urgency score from 1 to 5, and given a suggested reply. The property address or unit reference is pulled directly from the email body, so the right team knows exactly which property it relates to without reading the full message. Automated notifications, newsletters, and non actionable alerts are recognized and filtered out as Spam rather than treated as real tickets, so a team is never falsely alerted about something that isn't an actual tenant or client inquiry.
4. **Manual Review Fallback.** If the AI can't confidently classify an email, it doesn't guess. The ticket is routed to a manual review queue instead of risking a wrong routing decision.
5. **Team Routing.** Based on the category, the right team is notified on Slack with full context: sender, property reference, subject, message, urgency, and the AI's suggested reply.
6. **Draft Reply.** A ready to review reply is created as a Gmail draft, never sent automatically. A human always reviews the full original email and the draft before anything goes to the tenant or client.
7. **Verified Logging.** Every ticket is saved to Airtable. If the Airtable write fails for any reason, the ticket is automatically saved to a backup Google Sheet instead, and a failure notification is sent, so no ticket is ever silently lost.

## Why It's Built This Way

A workflow that runs without errors isn't the same as a workflow that behaves correctly. During testing, a state handling issue was found where the workflow reused a previous test value instead of the newly received email. The execution looked successful, but the output was wrong. That's the exact kind of failure this project is built to catch. Every external call, whether it's AI classification or a CRM write, is treated as something to verify, not something to assume worked just because it didn't throw an error.

## Tools Used

`n8n` · `IMAP` · `Groq AI (Qwen 3.6 27B)` · `Airtable` · `Gmail` · `Slack` · `Google Sheets` · `JavaScript` (custom parsing and sanitization logic)

## Files in This Repo

- `Property Inbox AI Classifier.json`, the full exportable n8n workflow. Import it directly into your own n8n instance.
- `Property Inbox AI Classifier.png`, a visual diagram of the workflow.
- `README.md`, this file.

## Setup

1. Import the JSON file into your n8n instance using "Import from File."
2. Connect your own credentials for IMAP, Groq, Airtable, Gmail, Google Sheets, and Slack.
3. Update the Slack channel IDs, Airtable base and table references, and Google Sheets document ID to match your own setup.
4. In the "Email Trigger (IMAP)" node, confirm the option to mark emails as read is enabled so already processed emails aren't picked up again.
5. Update the system prompt in the Groq node with your own property categories and business context so classification reflects your actual use case.
6. Activate the workflow.

## About

Built by **Muhammad Sami Ullah**, an AI and workflow automation specialist for real estate and property teams, building n8n based automation systems that remove manual busywork from property inboxes.
