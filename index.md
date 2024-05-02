# STRIN
STRIN is a protocol for managing arbitrarily many typed connection streams to a server. It runs on top of [WebSocket](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API), interpreting each complete frame as a packet:

~~~rs
struct Packet {
  variant: u8;
  sequence: u16;
  packetdata: [u8];
}
~~~

## Specification Syntax
Packet type definitions are similar to Rust `struct` declarations. However:
* Array literals may reference prior fields as their length:

  ~~~rs
  struct List<T> {
    len: u32;
    data: [T; len];
  }
  ~~~

* `if` statements may occur in types (and reference prior fields):

  ~~~rs
  struct Cond<T> {
    present: bool;
    if present {
      value: T;
    }
  }
  ~~~

* Integer types for all bit widths exist (but all other types are multiples of 8 bits):

  ~~~rs
  struct Bitfield {
    a: u1;
    c: u3;
    r: u12;
  }
  ~~~

## Common Definitions

~~~rs
struct ChanID {
  type_part: u32;
  id_part: u96;
}
~~~

## Client→Server Packets

### `00` Client Data
[Client Data]: #00-client-data
Represents data being sent to the server on a connection created with [Connect]. This packet **MUST NOT** be sent to a channel until a corresponding [Connect Response] is received. Implementations **MUST NOT** interpret the payload as anything other than raw bytes — only the channel type may interpret this data.

~~~rs
struct ClientData {
  payload: [u8];
}
~~~

### `01` Connect
[Connect]: #01-connect
Connects to a given channel by ID. The upper 32 bits of the ID define the channel type, as with all channel IDs. This must be responded to with either [Connect Response] or [Failure].

~~~rs
struct Connect {
  id: chanid;
  payload: [u8];
}
~~~

#### Errors

| ID | Description
|:--:| :-
|`40`| Nonexistent channel type
|`44`| Channel ID does not exist for channel type

### `02` Disconnect
[Disconnect]: #02-disconnect
Terminates a connection (and acts as an acknowledgement for [Disconnect Request]). Clients **MUST NOT** send further [Client Data] packets on this channel, but **SHOULD** expect further [Server Data] packets until a [Disconnect Response] is received.

~~~rs
struct Disconnect {
  id: chanid;
  payload: [u8];
}
~~~

## Server→Client Packets

### `00` Server Data
[Server Data]: #00-server-data
The equivalent of [Client Data] but traveling the opposite direction.

~~~rs
struct ServerData {
  payload: [u8];
}
~~~

### `01` Connect Response
[Connect Response]: #01-connect-response
Completes a connection handshake initiated by [Connect]. `handle` is the ID of a connection to the channel, and `payload` is a response to [Connect]'s payload.

~~~rs
struct ConnectResponse {
  reply: u16;
  id: ChanID;
  handle: u128;
  payload: [u8];
}
~~~

### `02` Disconnect Response
[Disconnect Response]: #02-disconnect-response
This acknowledges a client's disconnection. This packet **MUST** be the first packet sent by the server after it receives a [Disconnect]. Servers **MUST NOT** send any [Server Data] packets on this connection afterwards. Servers are incapable of preventing disconnection.

~~~rs
struct DisconnectResponse {
  reply: u16;
  id: ChanID;
}
~~~

### `03` Disconnect Request
[Disconnect Request]: #03-disconnect-request
Represents the server choosing to terminate a connection. Clients **MUST** immediately reply with [Disconnect], making this less of a request and more of a requirement. Servers **MUST NOT** send any further [Server Data] packets on this channel after sending this packet.
~~~rs
struct DisconnectRequest {
  id: ChanID;
  payload: [u8];
}
~~~

### `E1` Failure
[Failure]: #e1-failure
Acts as a generic failure message. `error` is a numeric error type whose possibilities are decided by the `reply`ed-to packet. Errors with bit 15 set are implementation-defined; errors with bit 15 clear are reserved for the specification.

~~~rs
struct ConnectFail {
  reply: u16;
  id: ChanID;
  err_payload: [u8];
}
~~~

## Common Protocols

* `1117` [Echo](/common/1117.md)