# Support Thread Manager — Bot specification

**Archetype:** support

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that routes user messages from a group to dedicated support topics, and forwards staff replies back to the original author as direct messages. Each user gets a single topic in a support group, which is reused until explicitly closed.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Group members seeking support
- Support staff and moderators

## Success criteria

- All user messages are routed to the correct support topic
- Staff replies are delivered to the original author as DMs
- Topics are automatically created and reused per author
- Topic closure prevents reuse until new messages are sent

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu
- **Close Topic** (button, actor: staff, callback: topic:close) — Close the current support topic for this user

## Flows

### User message to support
_Trigger:_ user message mentioning bot or replying to bot

1. Check for existing topic for user
2. Create new topic if none exists
3. Forward message to topic with author metadata
4. Send ephemeral confirmation to user

_Data touched:_ user, support topic, message mapping

### Staff reply to user
_Trigger:_ staff message in topic

1. Identify topic's mapped user
2. Forward staff message to user as DM
3. Preserve message content and attachments

_Data touched:_ message mapping, user

### Topic closure
_Trigger:_ /close or button click

1. Mark topic as closed in mapping
2. Prevent reuse of topic for future messages from user

_Data touched:_ support topic, message mapping

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **user** _(retention: persistent)_ — Telegram user who sent a message to the bot
  - fields: user_id, display_name, chat_id
- **support topic** _(retention: persistent)_ — Telegram topic (thread) in support group for a specific user
  - fields: topic_id, group_chat_id, user_id, is_closed
- **message mapping** _(retention: persistent)_ — Link between user messages and their support topic
  - fields: message_id, user_id, topic_id

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Designate the support group chat
- Configure topic naming convention
- Set rate limits and throttling policies
- Define closure triggers (button/command)

## Notifications

- Ephemeral confirmation messages to users
- Direct message notifications for staff replies

## Permissions & privacy

- Only staff can view support topics
- User identifiers are visible in support topics for context
- User messages are stored with mapping for reply routing

## Edge cases

- User sends message after topic closure
- Multiple messages from same user in quick succession
- Staff reply to closed topic
- User changes display name or chat context

## Required tests

- Verify message routing from user to correct topic
- Confirm staff replies are delivered as DMs
- Test topic creation and reuse logic
- Validate closure behavior prevents topic reuse

## Assumptions

- Support group has topics enabled
- Staff have access to the support group
- Users know how to mention or reply to the bot in group chat
