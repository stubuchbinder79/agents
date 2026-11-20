### Copilot Instructions
## Project Overview
This repository contains a Microsoft 365 Copilot Declarative Agent named Meeting Response Tracker.

The solution identifies Outlook meeting attendees who have not responded to invitations and creates Microsoft Teams follow-up communications.

The implementation should prioritize:

- Microsoft 365 native capabilities
- Power Automate
- Microsoft Graph
- Copilot Studio
- Declarative Agents
- Teams and Outlook integrations

Avoid introducing Azure Functions, custom web APIs, containers, databases, or any infrastructure unless explicitly requested.

## Architecture Principles
The architecture is:

Microsoft 365 Copilot
    │
    ▼
Declarative Agent
    │
    ▼
Power Automate Flow
    │
    ├── Outlook Calendar
    ├── Microsoft Graph
    └── Microsoft Teams
The Power Automate flow performs the business logic.

The agent performs orchestration and conversation.

## Agent Responsibilities
The agent should:

1. Identify meetings organized by the current user.
2. Read attendee response status.
3. Find attendees with response status of:
none
4. Create or locate follow-up Teams channels.
5. Add non-responders.
6. Post reminder messages.
7. Provide response summaries.

The agent should not:

- Accept invitations
- Decline invitations
- Reschedule meetings
- Modify attendee lists
- Remove attendees
- Access meetings not organized by the current user

## Power Automate Standards
When generating flows:

Prefer:
- Built-in Outlook actions
- Teams actions
- Microsoft Graph where required
- Expressions over custom code

Avoid:
- Premium connectors unless required
- Azure Functions
- Custom APIs
- Manual data entry

All flows must:
- Handle empty collections safely
- Avoid duplicate Teams channels
- Avoid duplicate reminders
- Return structured JSON responses

## Teams Standards
Preferred Team:
Meeting Follow-Ups

Preferred channel naming:
{Meeting Subject} - Pending Responses
Examples:

Architecture Review - Pending Responses
Project Kickoff - Pending Responses
Quarterly Planning - Pending Responses

Before creating a channel:
- Check whether the channel already exists.
- Reuse existing channels.
- Only create channels when necessary.

## Outlook Response Status Values
Consider the following values:
- accepted
- declined
- tentativelyAccepted
- none

The value:
- none
means the attendee has not responded.

Only attendees with a status of none should receive follow-up reminders.

## Preferred Graph Endpoints
Meetings:
GET /me/events

Attendees:
GET /me/events/{event-id}

Teams Channels:
GET /teams/{team-id}/channels
POST /teams/{team-id}/channels

Chat Messages:
POST /teams/{team-id}/channels/{channel-id}/messages

Members:
POST /teams/{team-id}/members

Use Microsoft Graph whenever native connectors cannot expose required attendee data.

## Power Automate Expression Guidance
Use expressions rather than custom code whenever possible.

Filter organizer-owned meetings:

@equals(item()?['isOrganizer'], true)

Find matching meetings:

@contains(
  toLower(item()?['subject']),
  toLower(triggerBody()?['meetingSubject'])
)
Find non-responders:

@equals(
  item()?['status']?['response'],
  'none'
)
Count records:

length(...)

Generate channel names:

concat(
  outputs('Compose_Meeting')?['subject'],
  ' - Pending Responses'
)

## Copilot Response Behavior
When generating summaries use:

Meeting: {Meeting Name}

Accepted: X
Tentative: X
Declined: X
No Response: X

Channel:
{Channel Name}
When no non-responders exist:

All invited attendees have responded.
No follow-up action is required.

## Coding Standards
When generating JSON:

- Use camelCase.
- Generate strongly typed schemas.
- Include example payloads.
- Keep actions idempotent where possible.

When generating documentation:
- Prefer practical implementation details.
- Include Power Automate expressions.
- Include Graph endpoint examples.
- Avoid placeholder text when possible.

## Repository Goal
The final solution should be deployable by a Microsoft 365 administrator using:
- Microsoft 365 Copilot
- Copilot Studio
- Power Automate
- Microsoft Teams
- Outlook

with no custom infrastructure dependencies.