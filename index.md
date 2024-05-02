# STRIN
STRIN is a protocol for managing arbitrarily many typed connection streams to a server. It runs on top of [WebSocket](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API), interpreting each complete frame as a packet:
~~~rs
struct Packet {
    type: u8;
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
## Common
~~~rs
struct ChanID {
    typePart: u32;
    idPart: u96;
}
~~~
## C→S Packets
### `00` Client Data
[Client Data]: #00-client-data
Represents data being sent to the server on a connection created with [Connect]. STRIN does not interpret the payload as anything other than raw bytes — only the channel type may interpret this data.
~~~rs
struct ClientData {
    payload: [u8];
}
~~~
### `01` Connect
[Connect]: #01-connect
Connects to a given channel by ID. The upper 32 bits of the ID define the channel type, as with all channel IDs. This will be followed by [Connect Response].
~~~rs
struct Connect {
    id: chanid;
    payload: [u8];
}
~~~
## S→C Packets
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
Completes a connection handshake. If `error` is 0, `handle` is the ID of a connection to the channel, and `payload` is a response to [Connect]'s payload. Otherwise, `errpayload` is additional information about `error` (if any).
~~~rs
struct ConnectResponse {
    reply: u16;
    id: ChanID;
    error: u16;
    if error == 0 {
        handle: u128;
        payload: [u8];
    } else {
        errpayload: [u8];
    }
}
~~~