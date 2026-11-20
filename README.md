# Meeting Response Tracker Agent

## Overview

Meeting Response Tracker is a Microsoft 365 Copilot Declarative Agent that identifies meeting invitees who have not responded to Outlook meeting requests and automatically creates a Microsoft Teams follow-up channel for them.

The agent uses:

- Microsoft 365 Copilot Declarative Agent
- Copilot Studio
- Power Automate
- Microsoft Graph
- Microsoft Teams
- Outlook Calendar

## Business Problem

Meeting organizers frequently spend time manually checking Outlook invitations and chasing attendees who haven't responded.

This agent automates the process by:

1. Finding meetings organized by the current user.
2. Identifying attendees whose response status is `none`.
3. Creating or reusing a Teams follow-up channel.
4. Adding non-responders to the channel.
5. Posting a reminder message.
6. Returning a summary to the organizer.

---

# Solution Architecture

```text
Copilot Agent
      │
      ▼
Power Automate Flow
      │
      ▼
Outlook Calendar
      │
      ▼
Attendee Response Analysis
      │
      ▼
Microsoft Teams
      ├── Create Channel
      ├── Add Users
      └── Send Reminder
```

---

# Repository Structure

```text
meeting-response-tracker/

├── agents.md
├── instructions.md
├── actions.json

├── power-automate/
│   └── FollowUpNonResponders.md

├── samples/
│   ├── sample-request.json
│   └── sample-response.json

└── README.md
```

---

# Agent Responsibilities

The agent must:

- Identify meetings organized by the current user.
- Locate attendees with response status of:
  - none
- Generate attendance statistics.
- Create Teams follow-up channels.
- Add users to channels.
- Send reminder messages.

The agent must not:

- Accept meetings.
- Decline meetings.
- Modify attendee lists.
- Access meetings not organized by the current user.

---

# Expected User Prompts

Examples:

```text
Who hasn't responded to Architecture Review?

Check responses for Design Review.

Create a reminder channel for non-responders.

Follow up with attendees who haven't responded.

Show invite status for tomorrow's meetings.
```

---

# Power Automate Flow

Name:

```text
FollowUpNonResponders
```

Trigger:

```text
When an agent invokes the flow
```

Inputs:

```json
{
  "meetingSubject": "string",
  "daysAhead": 7,
  "createChannel": true
}
```

Outputs:

```json
{
  "meeting": "string",
  "accepted": 0,
  "tentative": 0,
  "declined": 0,
  "noResponse": 0,
  "channelName": "string",
  "usersAdded": 0
}
```

---

# Teams Channel Strategy

Preferred Team:

```text
Meeting Follow-Ups
```

Channel Naming Convention:

```text
{Meeting Subject} - Pending Responses
```

Examples:

```text
Architecture Review - Pending Responses

Quarterly Planning - Pending Responses

Product Launch Review - Pending Responses
```

Always reuse existing channels when present.

Never create duplicate channels.

---

# Reminder Template

```text
Hello,

You have not yet responded to the meeting invitation:

{Meeting Subject}

Meeting Time:
{Meeting DateTime}

Please accept or decline the Outlook invitation.

Thank you.
```

---

# Required Power Platform Connectors

- Microsoft Teams
- Office 365 Outlook
- Microsoft 365 Users

Optional:

- HTTP with Microsoft Entra ID
- Microsoft Graph

---

# Copilot Agent Instructions

The agent should always:

1. Determine the target meeting.
2. Retrieve attendee statuses.
3. Identify non-responders.
4. Reuse an existing follow-up channel when possible.
5. Create a channel when necessary.
6. Add non-responders.
7. Post reminder messages.
8. Return a concise summary.

The agent should never invent meeting details or attendee information.

Always use data returned from connected actions.

---

# Development Notes

When generating code or configuration:

- Prefer strongly typed JSON schemas.
- Use Power Automate expressions instead of custom code when possible.
- Avoid duplicate Teams channels.
- Handle empty attendee lists.
- Handle meetings with no non-responders.
- Handle recurring meeting instances.
- Handle guest users and external attendees.

---

# Success Criteria

A successful run produces:

```text
Meeting: Architecture Review

Accepted: 10
Tentative: 1
Declined: 2
No Response: 3

Follow-Up Channel:
Architecture Review - Pending Responses

Reminders Sent:
3
```

The solution must be deployable entirely through Microsoft 365 Copilot, Copilot Studio, Power Automate, and Microsoft Teams without custom infrastructure.

# Workflow
Build the complete repository structure, generate all missing files, and create production-ready Power Automate flow documentation with request and response schemas.

```
Read .github/copilot-instructions.md and .github/prompts/build-agent.prompt.md.
```

