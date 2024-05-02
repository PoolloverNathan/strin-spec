# STRIN
STRIN is a protocol for managing arbitrarily many typed connection streams to a server. It runs on top of [WebSocket], 
## Common
~~~c
struct chanid {
    u32  typePart;
    u128 idPart: 96;
}
~~~
## C→S Packets
### 00 Client Data
[Client Data]: #00-client-data
Represents data being sent to the server on a connection created with [Connect]. STRIN does not interpret the payload as anything other than raw bytes — only the channel type may interpret this data.
~~~c
struct ClientData {
    byte[] payload;
}
~~~
### 01 Connect
[Connect]: #01-connect
Connects to a given channel by ID. The upper 32 bits of the ID define the channel type, as with all channel IDs. This will be followed by [Connect Response].
~~~c
struct Connect {
    chanid id;
    byte[] payload;
}
~~~
## S→C Packets
### 00 Server Data
[Server Data]: #00-server-data
The equivalent of [Client Data] but traveling the opposite direction.
~~~c
struct ServerData {
    byte[] payload;
}
~~~
### 01 Connect Response
[Connect Response]: #01-connect-response
Completes a connection handshake. If `error` is 0, `handle` is the ID of a connection to the channel, and `payload` is a response to [Connect]'s payload. Otherwise, `errpayload` is additional information about `error` (if any).
~~~c
struct ConnectResponse {
    chanid id;
    u16 error;
    union {
        struct {
            u128 handle;
            byte[] payload;
        };
        byte[] errpayload;
    };
}
~~~