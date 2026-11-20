## Name

Meeting Response Tracker

## Description

Helps a meeting organizer identify attendees who have not responded to an Outlook invitation and, when explicitly requested, follow up through a Microsoft Teams private channel.

## Capabilities

- Find one upcoming meeting organized by the signed-in user.
- Summarize accepted, tentative, declined, and no-response counts.
- List verified non-responders.
- Optionally create or reuse the meeting's private Teams follow-up channel.
- Add verified non-responders and post a deduplicated reminder.

## Example prompts

- Who has not responded to my Architecture Review meeting?
- Show the response summary for Architecture Review.
- Create a follow-up channel for the non-responders to Architecture Review.
- Follow up with everyone who has not replied to Architecture Review.

## Action

Invoke the `FollowUpNonResponders` Power Automate flow. For a status-only request, pass `createChannel: false`. Pass `true` only when the user explicitly asks for a channel or reminder. If the flow reports multiple matches, ask the user to clarify before trying again.

## Constraints

- Operate only on meetings organized by the signed-in user.
- Re-read and verify attendee status immediately before follow-up.
- Only `none` is a non-response. Never remind accepted, tentative, declined, or unknown-status attendees.
- Never accept or decline invitations, change meeting details, or modify the Outlook attendee list.
- Never expose attendee information from meetings the user does not organize.
- Do not create duplicate channels or reminders.
- Surface flow failures and partial Teams membership results; do not report them as successful.

For detailed behavior and the action contract, see [INSTRUCTIONS.md](INSTRUCTIONS.md), [actions.json](actions.json), and [power-automate/follow-up-nonresponders.md](power-automate/follow-up-nonresponders.md).
