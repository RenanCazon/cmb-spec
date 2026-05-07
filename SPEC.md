# CMB Protocol v1

## 1. Purpose

CMB Protocol defines a small real-time event stream that lets AI agents observe email activity and publish presence state without direct mailbox credentials.

The protocol is designed for:

- AI inbox copilots
- automated triage agents
- mail-aware orchestration
- human-visible AI presence panels

## 2. Transport

The v1 transport is WebSocket over HTTPS.

Reference endpoint:

```txt
wss://bridge.cazonai.com/v1/subscribe
```

Clients must send these WebSocket subprotocol values:

```txt
cmb.v1
cmb-bearer-<token>
```

The server selects only:

```txt
cmb.v1
```

This prevents bearer tokens from being reflected as the negotiated protocol.

## 3. Authentication

Bearer tokens identify agents and allowed scopes. Raw tokens are secret material.

Servers should:

- hash tokens at rest
- redact tokens from logs
- reject unknown or malformed tokens before WebSocket upgrade
- close unauthorized connections with `1008`

## 4. Scopes

Initial scopes:

```txt
email.received
ai.presence
```

`email.received` allows receiving mailbox events.

`ai.presence` allows publishing AI state events.

## 5. Envelope

All messages are JSON objects with this envelope:

```json
{
  "event": "event.name",
  "version": "1.0",
  "timestamp": "2026-05-07T01:00:00.000Z"
}
```

Fields:

- `event`: event name
- `version`: payload version, currently `1.0`
- `timestamp`: ISO 8601 UTC timestamp
- event-specific fields may appear at the top level

## 6. Server Events

### session.welcome

Sent immediately after a client is accepted.

```json
{
  "event": "session.welcome",
  "version": "1.0",
  "timestamp": "2026-05-07T01:00:00.000Z",
  "session_id": "session-uuid",
  "ai_id": "codex",
  "ai_name": "Codex",
  "scopes": ["email.received", "ai.presence"]
}
```

### email.received

Sent when a new message is observed by IMAP IDLE.

```json
{
  "event": "email.received",
  "version": "1.0",
  "timestamp": "2026-05-07T01:00:00.000Z",
  "uid": 48,
  "mailbox": "renan.cazon@cazonai.com",
  "from": "Sender <sender@example.com>",
  "subject": "Subject",
  "preview": "",
  "message_id": "provider-message-id",
  "metadata": {
    "has_attachments": false,
    "flags": []
  }
}
```

### heartbeat.ack

Sent after a client heartbeat.

```json
{
  "event": "heartbeat.ack",
  "version": "1.0",
  "timestamp": "2026-05-07T01:00:00.000Z"
}
```

### error

Sent when a recoverable protocol error occurs.

```json
{
  "event": "error",
  "version": "1.0",
  "timestamp": "2026-05-07T01:00:00.000Z",
  "error": "invalid_json"
}
```

## 7. Client Events

### heartbeat

```json
{
  "event": "ai.heartbeat"
}
```

### ai.online

Server-emitted when an agent connects.

```json
{
  "event": "ai.online",
  "ai_id": "codex",
  "ai_name": "Codex"
}
```

### ai.offline

Server-emitted when an agent disconnects.

```json
{
  "event": "ai.offline",
  "ai_id": "codex",
  "ai_name": "Codex"
}
```

### ai.reading_email

```json
{
  "event": "ai.reading_email",
  "uid": 48
}
```

### ai.composing_email

```json
{
  "event": "ai.composing_email",
  "to": "recipient@example.com"
}
```

### ai.sent_email

```json
{
  "event": "ai.sent_email",
  "message_id": "provider-message-id"
}
```

### ai.replied_email

```json
{
  "event": "ai.replied_email",
  "uid": 48
}
```

### ai.archived_email

```json
{
  "event": "ai.archived_email",
  "uid": 48
}
```

### ai.idle

```json
{
  "event": "ai.idle"
}
```

## 8. Health Contract

Implementations should expose:

```txt
GET /health
HEAD /health
```

The response should include:

- service status
- IMAP connection state
- active IMAP IDLE session count
- WebSocket subscriber count
- current protocol version

## 9. Hostinger Reference Constraints

The first deployment uses Hostinger mail:

- IMAP: `imap.hostinger.com:993` SSL
- SMTP: `smtp.hostinger.com:465` SSL
- IMAP IDLE: one persistent session
- Avoid parallel mailbox sessions unless the provider plan allows it

## 10. Compatibility

Breaking changes require a new protocol version string and endpoint compatibility notes.
