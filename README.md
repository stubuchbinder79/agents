# Meeting Response Tracker

Meeting Response Tracker finds non-responders for meetings organized by the signed-in user and can follow up through a Microsoft Teams private channel. The flow performs business logic; the agent interprets the user's request and presents the flow result.

## What is included

- [AGENTS.md](AGENTS.md) — agent summary and safety constraints.
- [INSTRUCTIONS.md](INSTRUCTIONS.md) — conversation behavior and action-call rules.
- [actions.json](actions.json) — strongly typed Power Automate action contract.
- [schemas/request-schema.json](schemas/request-schema.json) and [schemas/response-schema.json](schemas/response-schema.json) — request and response schemas.
- [samples/sample-request.json](samples/sample-request.json) and [samples/sample-response.json](samples/sample-response.json) — example payloads.
- [power-automate/follow-up-nonresponders.md](power-automate/follow-up-nonresponders.md) — flow build, connection, and test instructions.

## Make the agent runnable

The repository contains the agent instructions and action contract, but it does not contain a provisioned Microsoft 365 tenant, Power Automate connections, or an exported solution package. Complete these tenant-specific steps to run it:

1. In Power Automate, create an instant cloud flow named `FollowUpNonResponders` with the **When an agent calls the flow** trigger.
2. Implement the lookup, organizer verification, response counting, channel reuse/creation, membership, reminder de-duplication, and response paths in [power-automate/follow-up-nonresponders.md](power-automate/follow-up-nonresponders.md).
3. Configure the flow trigger and response using [schemas/request-schema.json](schemas/request-schema.json) and [schemas/response-schema.json](schemas/response-schema.json). Keep the flow aligned with [actions.json](actions.json).
4. In Copilot Studio, create the agent, add the content of [INSTRUCTIONS.md](INSTRUCTIONS.md) as its instructions, add the saved flow as the `FollowUpNonResponders` action, and map each input/output.
5. Sign in with an organizer account, verify Outlook and Teams connector access, test all cases documented in the flow guide, then publish the agent to the intended Microsoft 365 Copilot channel.

The flow defaults to reporting only. The agent must pass `createChannel: true` only for an explicit follow-up request. Required Teams membership and Graph permissions depend on the tenant's channel configuration; configure the least-privilege access described in the flow guide.

## Example

Request:

```json
{
  "meetingSubject": "Architecture Review",
  "daysAhead": 7,
  "createChannel": true
}
```

The sample response is in [samples/sample-response.json](samples/sample-response.json). A status-only request sets `createChannel` to `false`; it returns counts and any verified non-responders without creating a channel or sending a reminder.
