# CMB Protocol

CMB Protocol is the first Cazon AI bridge standard for real-time email events between a mailbox and AI agents.

The initial reference deployment is live at:

- Health: https://bridge.cazonai.com/health
- WebSocket: `wss://bridge.cazonai.com/v1/subscribe`

## Status

- Version: `v1`
- Reference implementation: CazonMail CMB Bridge
- Transport: WebSocket over HTTPS
- Auth: bearer token passed as a WebSocket subprotocol
- Mailbox source: IMAP IDLE, single persistent session

## WebSocket Handshake

Clients connect with two subprotocol values:

```txt
cmb.v1
cmb-bearer-<token>
```

The server selects `cmb.v1` and never echoes the bearer token as the selected protocol.

## Event Shape

Every event is JSON:

```json
{
  "type": "email.received",
  "version": "v1",
  "timestamp": "2026-05-07T01:00:00.000Z",
  "data": {}
}
```

See [SPEC.md](./SPEC.md) for the full event catalog.

## Minimal Client

```js
import WebSocket from "ws";

const token = process.env.CMB_TOKEN;
const ws = new WebSocket("wss://bridge.cazonai.com/v1/subscribe", [
  "cmb.v1",
  `cmb-bearer-${token}`
]);

ws.on("message", (payload) => {
  const event = JSON.parse(payload.toString());
  console.log(event.type, event.data);
});
```

## Security Notes

- Tokens are stored hashed in the bridge database.
- Raw tokens must stay in the operator vault, never in logs, screenshots, or issues.
- The reference implementation uses one IMAP IDLE session to respect Hostinger mailbox limits.
