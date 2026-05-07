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
  "type": "event.name",
  "version": "v1",
  "timestamp": "2026-05-07T01:00:00.000Z",
  "data": {}
}
```

Fields:

- `type`: event name
- `version`: protocol version, currently `v1`
- `timestamp`: ISO 8601 UTC timestamp
- `data`: event-specific object

## 6. Server Events

### session.welcome

Sent immediately after a client is accepted.

```json
{
  "type": "session.welcome",
  "version": "v1",
  "timestamp": "2026-05-07T01:00:00.000Z",
  "data": {
    "agent": "codex",
    "scopes": ["email.received", "ai.presence"]
  }
}
```

### email.received

Sent when a new message is observed by IMAP IDLE.

```json
{
  "type": "email.received",
  "version": "v1",
  "timestamp": "2026-05-07T01:00:00.000Z",
  "data": {
    "uid": 48,
    "mailbox": "INBOX",
    "from": "sender@example.com",
    "subject": "Subject",
    "date": "2026-05-07T01:00:00.000Z"
  }
}
```

### heartbeat.ack

Sent after a client heartbeat.

```json
{
  "type": "heartbeat.ack",
  "version": "v1",
  "timestamp": "2026-05-07T01:00:00.000Z",
  "data": {
    "agent": "codex"
  }
}
```

### error

Sent when a recoverable protocol error occurs.

```json
{
  "type": "error",
  "version": "v1",
  "timestamp": "2026-05-07T01:00:00.000Z",
  "data": {
    "code": "invalid_message",
    "message": "Invalid JSON"
  }
}
```

## 7. Client Events

### heartbeat

```json
{
  "type": "heartbeat",
  "data": {}
}
```

### ai.online

```json
{
  "type": "ai.online",
  "data": {
    "agent": "codex"
  }
}
```

### ai.offline

```json
{
  "type": "ai.offline",
  "data": {
    "agent": "codex"
  }
}
```

### ai.reading_email

```json
{
  "type": "ai.reading_email",
  "data": {
    "agent": "codex",
    "uid": 48
  }
}
```

### ai.composing_email

```json
{
  "type": "ai.composing_email",
  "data": {
    "agent": "codex",
    "to": "recipient@example.com"
  }
}
```

### ai.sent_email

```json
{
  "type": "ai.sent_email",
  "data": {
    "agent": "codex",
    "message_id": "provider-message-id"
  }
}
```

### ai.replied_email

```json
{
  "type": "ai.replied_email",
  "data": {
    "agent": "codex",
    "uid": 48
  }
}
```

### ai.archived_email

```json
{
  "type": "ai.archived_email",
  "data": {
    "agent": "codex",
    "uid": 48
  }
}
```

### ai.idle

```json
{
  "type": "ai.idle",
  "data": {
    "agent": "codex"
  }
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
