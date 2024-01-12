# Wisp - A Lightweight Multiplexing Websocket Proxy Protocol

Draft 1 - written by [@ading2210](https://github.com/ading2210)

## About
Wisp is designed to be a low-overhead, easy to implement protocol for proxying multiple TCP sockets over a single websocket connection. As it is only intended to carry TCP traffic, it is a lot simpler than alternatives such as penguin-rs.

## Packet format
| Field Name  | Field Type | Notes                                         |
|-------------|------------|-----------------------------------------------|
| Packet Type | `uint8_t`  | The packet type, covered in the next section. |
| Stream ID   | `uint32_t` | Random stream ID assigned by the client.      |
| Payload     | `char[]`   | Payload takes up the rest of the packet.      |

Every packet must follow this format regardless of the type. Note that all data types are little-endian. 

## Packet Types
Each packet type has a different format for the payload, which is detailed below.

### `0x01` - CONNECT
#### Payload Format
| Field Name           | Field Type | Notes                                    |
|----------------------|------------|------------------------------------------|
| Destination Port     | `uint16_t` | Destination TCP port for the new stream. |
| Destination Hostname | `char[]`   | Destination hostname, in a UTF-8 string. |

#### Behavior
The client needs to send a CONNECT packet to the server to create a new stream under the same websocket. The stream ID chosen by the client at this point will be associated with this stream for all future messages. If creating the stream fails during this process, the server needs to immediately send a CLOSE packet with a relevant reason.

On receiving a CONNECT packet, the server must try to establish a TCP socket to the specified hostname and port. If this fails, the server must send a CLOSE packet with the reason.

### `0x02` - DATA
#### Payload Format
| Field Name     | Field Type | Notes                                                      |
|----------------|------------|------------------------------------------------------------|
| Stream Payload | `char[]`   | The data which is sent to and from the destination server. |

#### Behavior
Any DATA packets sent from the client to the server must be proxied to the TCP socket associated with the stream ID of the packet.

Any DATA packets sent from the server to the client must be interpreted as coming from the TCP socket associated with the stream ID of the packet.

### `0x03` - CLOSE
#### Payload format
| Field Name   | Field Type | Notes                                  |
|--------------|------------|----------------------------------------|
| Close Reason | `uint8_t`  | The reason for closing the connection. |

#### Behavior
Any CLOSE packets sent from either the server or the client must immediately close the associated stream and TCP socket. The close reason in the payload doesn't affect this behavior, but may provide extra information which is useful for debugging.

#### Client/Server Close Reasons
- `0x01` - Reason unspecified or unknown. Returning a more specific reason should be preferred.
- `0x02` - Voluntary stream closure, which would equate to one side resetting the connection.
- `0x03` - Unexpected stream closure due to a network error.

#### Server Only Close Reasons
- `0x41` - Stream creation failed due to invalid information. This could be sent if the destination was a reserved address or the port is invalid. 
- `0x42` - Stream creation failed due to an unreachable destination host. This could be sent if the destination is an domain which does not resolve to anything. 
- `0x43` - Stream creation timed out due to the destination server not responding.
- `0x44` - Stream creation failed due to the destination server refusing the connection.
- `0x47` - TCP data transfer timed out.
- `0x48` - Stream destination address/domain is intentionally blocked by the proxy server.
- `0x49` - Connection throttled by the server.

#### Client Only Close Reasons
- `0x81` - The client has encountered an unexpected error and is unable to receive any more data. 

## HTTP Behavior

Since the entire protocol takes place over websockets, a few rules need to be in place to ensure that an HTTP connection can be upgraded successfully.

### Server Architecture
The server must consist of a HTTP and websocket server that conforms to the respective standards.

### Choosing a Websocket URL
To ensure compatibility with previous wsproxy implementations, the URL of the websocket should always end with a trailing forward slash (`/`) to prevent confusion with wsproxy endpoints. 

For example, a wsproxy endpoint might look like this: `ws://example.com/customprefix/host:port`

Meanwhile a Wisp endpoint would look like this: `ws://example.com/customprefix/`

It is up to the server implementation to interpret the prefix. It may be ignored, or used for gatekeeping in order to establish password-based authentication.

### Establishing a Websocket Connection
The client must establish the connection by performing a standard websocket handshake. The `Sec-WebSocket-Protocol` header must be set to `wisp-v1`. If this protocol header is mismatched between the server and the client, the connection cannot proceed.

### Querying Server Information
The sever may optionally implement an information query feature, in which an HTTP GET request to the same aforementioned websocket URL will return some information about the server. The format of the returned data needs to be a JSON string. 

The fields in this endpoint are to be decided, but they should include info such as some statistics about server load, the server maintainers, and the protocol version.