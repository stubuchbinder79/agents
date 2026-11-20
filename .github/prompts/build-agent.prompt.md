# Build Meeting Response Tracker Agent

Your goal is to implement a production-ready Microsoft 365 Copilot Declarative Agent called **Meeting Response Tracker**.

Follow the requirements below exactly.

---

# Objective

Build a Microsoft 365 Copilot agent that:

1. Finds meetings organized by the current user.
2. Identifies attendees that have not responded.
3. Creates or reuses a Microsoft Teams follow-up channel.
4. Adds non-responders to the channel.
5. Posts reminder messages.
6. Returns a summary of invitation status.

The solution must use only:

- Microsoft 365 Copilot
- Declarative Agents
- Copilot Studio
- Power Automate
- Outlook
- Teams
- Microsoft Graph

Do not introduce:

- Azure Functions
- Web APIs
- Containers
- Databases
- VM infrastructure
- External services

---

# Required Deliverables

Generate the following files if they are missing.

```text
agents.md
instructions.md
actions.json

power-automate/
    FollowUpNonResponders.md

schemas/
    request-schema.json
    response-schema.json

samples/
    sample-request.json
    sample-response.json
```

---

# Agent Name

```text
Meeting Response Tracker
```

---

# User Stories

## Story 1

User says:

```text
Who hasn't responded to Architecture Review?
```

Agent must:

1. Locate the meeting.
2. Read attendee statuses.
3. Return summary counts.
4. List non-responders.

---

## Story 2

User says:

```text
Create a reminder channel.
```

Agent must:

1. Locate non-responders.
2.