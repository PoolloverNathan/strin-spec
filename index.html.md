# STRIN
<!-- todo -->
## C→S Packets
### 01 Connect
[C→S Connect]: #01-connect
Connects to a given channel by ID. The upper 32 bits of the ID define the channel type, as with all channel IDs. This will be followed by [S→C Connect Response].
~~~c
struct Connect {
    chanid id;
    byte[] payload;
}
~~~
## S→C Packets
### 01 Connect Response
[S→C Connect Response]: #01-connect-response
Completes a connection handshake. If `error` is 0, data may be sent on the channel.
~~~c
struct ConnectResponse {
    chanid id;
    byte[] payload;
}
~~~