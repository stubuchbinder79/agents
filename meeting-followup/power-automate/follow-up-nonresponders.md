# FollowUpNonResponders Power Automate Flow

## Overview

This flow is invoked by the Meeting Response Tracker Copilot Agent.

Its purpose is to:

1. Locate a meeting organized by the current user.
2. Retrieve attendee response status.
3. Identify attendees that have not responded.
4. Create or reuse a Teams follow-up channel.
5. Add non-responders to the channel.
6. Send a reminder message.
7. Return a structured summary to the agent.

---

# Flow Metadata

Name:

```text
FollowUpNonResponders
```

Type:

```text
Instant Cloud Flow
```

Trigger:

```text
When an agent invokes the flow
```

---

# Inputs

## meetingSubject

Type:

```text
String
```

Example:

```json
{
  "meetingSubject": "Architecture Review"
}
```

---

## daysAhead

Type:

```text
Integer
```

Default:

```text
7
```

Example:

```json
{
  "daysAhead": 7
}
```

---

## createChannel

Type:

```text
Boolean
```

Default:

```text
true
```

Example:

```json
{
  "createChannel": true
}
```

---

# Variables

## NonResponders

Type:

```text
Array
```

Initial Value:

```json
[]
```

---

## TeamName

Type:

```text
String
```

Value:

```text
Meeting Follow-Ups
```

---

# Step 1

## Get Calendar Events

Action:

```text
Office 365 Outlook
Get calendar view of events (V3)
```

Start Time:

```text
utcNow()
```

End Time:

```text
addDays(
 utcNow(),
 triggerBody()?['daysAhead']
)
```

Purpose:

Retrieve upcoming meetings.

---

# Step 2

## Filter Meetings