# Build and connect `FollowUpNonResponders`

This guide creates the Power Automate flow called by the Meeting Response Tracker agent. The flow reports invitation status by default. It creates a channel and sends a reminder only when the agent explicitly passes `createChannel: true`.

The flow must run with the signed-in organizer's Outlook and Teams connections. Do not use a shared service identity to read another user's calendar.

## 1. Create the agent flow

In Power Automate, create an instant cloud flow with the **When an agent calls the flow** trigger. Name it `FollowUpNonResponders`. Configure the trigger inputs from [request-schema.json](../schemas/request-schema.json):

| Input | Type | Required | Default |
|---|---|---:|---:|
| `meetingSubject` | Text | Yes | — |
| `daysAhead` | Number (integer) | No | `7` |
| `createChannel` | Yes/No | No | `false` |

Reject an empty subject and a `daysAhead` value outside 1–365. Do not send a reminder when input validation fails.

Use the flow response in [response-schema.json](../schemas/response-schema.json). Every branch must return the same response shape. Set `status` and `message` explicitly for no-match, ambiguity, no-response, follow-up-disabled, partial, and error branches; do not report success when an operation failed.

## 2. Find exactly one organizer-owned meeting

1. Use **Office 365 Outlook — Get calendar view of events (V3)**, from `utcNow()` through `addDays(utcNow(), triggerBody()?['daysAhead'])`.
2. Filter the returned events to events where `isOrganizer` is `true` and the subject contains the requested subject, case-insensitively:

   ```text
   @and(
     equals(item()?['isOrganizer'], true),
     contains(
       toLower(item()?['subject']),
       toLower(triggerBody()?['meetingSubject'])
     )
   )
   ```

3. If there are no matches, return `status: "noMeetingFound"`. If there are multiple matches, return `status: "multipleMeetingsFound"` and ask the organizer to provide a more specific subject. Never pick an arbitrary match.
4. Retrieve the selected event's latest details and attendees immediately before processing. Verify `isOrganizer` again. If this check fails, stop without returning attendee details or making Teams changes.

If Outlook does not expose the attendee response fields, use Microsoft Graph with the signed-in user's delegated connection:

```http
GET /me/events/{event-id}?$select=id,subject,isOrganizer,organizer,start,end,attendees,iCalUId
```

Use `Calendars.Read` delegated permission. Do not use `/users/{id}` or another user's calendar. Graph access is only a fallback when the Outlook connector does not return the required fields.

## 3. Count statuses and identify non-responders

Use the event's `attendees` array. The response is at `status.response`. Recognize only:

- `accepted` → accepted
- `tentativelyAccepted` → tentative
- `declined` → declined
- `none` → no response

For each count, filter the attendees array by the matching response and use `length(...)`. Filter non-responders with:

```text
@equals(item()?['status']?['response'], 'none')
```

An empty attendee array is valid: return zero counts and an empty `nonResponders` array. If an attendee has a missing or unrecognized response value, stop and return `status: "failed"` rather than silently treating that attendee as a non-responder or as a completed response.

If there are no non-responders, return `status: "noNonResponders"` with the counts. Do not create a channel or post a message.

If `createChannel` is false, return `status: "followUpDisabled"` with the counts and non-responders. Do not make Teams changes.

## 4. Create or reuse the Teams follow-up channel

Use the team named **Meeting Follow-Ups** and this exact channel name:

```text
{Meeting Subject} - Pending Responses
```

Resolve the team by exact display name and fail with a clear error if it is missing or ambiguous. List its channels and compare `displayName` exactly. Reuse an exact match; otherwise create a **private** channel. Do not create a duplicate.

Add only attendees whose latest response is `none`. A private-channel member must be a member of its parent team. If adding a non-responder to the team is part of the approved workflow, do so explicitly and report the resulting team and channel membership changes; otherwise leave that person unadded and add them to `followUpFailures`. Do not change the Outlook meeting or its attendee list.

Preferred Graph operations when the Teams connector lacks an equivalent action:

```http
GET  /teams/{team-id}/channels
POST /teams/{team-id}/channels
POST /teams/{team-id}/channels/{channel-id}/members
```

For channel creation, use `membershipType: "private"`. Use the required delegated Teams permissions and tenant consent for the exact operations enabled in the flow. Do not grant broader permissions than needed.

## 5. Avoid duplicate reminders and post the message

Post one generic reminder to the private follow-up channel; do not include attendee names or email addresses in the message body. Build a stable reminder marker from the event's `iCalUId` (or its immutable event ID if unavailable). Before posting, list the channel's existing messages and check for that marker. If it already exists, skip posting and return `remindersSent: 0`. Keep the marker in the message so later runs can detect it. Handle message pagination; checking only the first page is not sufficient.

Example message:

```text
Hello,

You have not yet responded to the meeting invitation:
{Meeting Subject}

Meeting time: {Meeting start time}

Please respond to the Outlook invitation by accepting or declining.

Thank you.
Reference: MRT-{stable-event-marker}
```

Use `POST /teams/{team-id}/channels/{channel-id}/messages` only if a native Teams action cannot post the channel message. Grant the least-privilege delegated permission required for channel messages.

## 6. Return the structured result

Return the same fields on every path:

- `status`: `completed`, `noMeetingFound`, `multipleMeetingsFound`, `noNonResponders`, `followUpDisabled`, `partial`, or `failed`
- `message`: concise outcome or next step
- `meetingSubject`: matched subject, or an empty string when no single meeting was selected
- `meetingStart`: ISO 8601 start time, or an empty string
- `accepted`, `tentative`, `declined`, `noResponse`: integer counts
- `channelName`: channel name, or an empty string
- `usersAdded`, `remindersSent`: integer counts
- `nonResponders`: array of `{ "name": string, "email": string }`
- `followUpFailures`: array of `{ "name": string, "email": string, "reason": string }`; use `status: "partial"` if some follow-up operations succeeded and others failed
- `errorCode`: empty string unless `status` is `failed` or `partial`

Use [sample-request.json](../samples/sample-request.json) and [sample-response.json](../samples/sample-response.json) to test the trigger and response. Test no match, multiple matches, an empty attendee list, each supported response value, missing response data, an existing channel, an existing reminder marker, and an attendee who cannot be added before enabling the flow for users.

## 7. Connect the flow to the agent

In Copilot Studio, add the saved flow as an agent action named `FollowUpNonResponders`, map its three trigger inputs, and expose its structured outputs. Keep the action contract in [actions.json](../actions.json) aligned with the flow. Publish the agent to the intended Microsoft 365 Copilot channel only after testing with an organizer account that has the required Outlook and Teams access.

For a status-only question, the agent must pass `createChannel: false`. Pass `true` only when the user explicitly asks to create a follow-up channel or send reminders. If multiple meetings match, ask the user to narrow the subject before calling the flow again.
