# Wisp - A Lightweight Multiplexing Websocket Proxy Protocol

Version 2.0, draft 4 - written by [@ading2210](https://github.com/ading2210)

> [!WARNING]
> This version of the protocol is still a draft. Do not use it in production.

## About
Wisp is designed to be a low-overhead, easy to implement protocol for proxying multiple TCP/UDP sockets over a single websocket connection. Wisp is simpler and has better error handling abilities compared to alternatives such as penguin-rs.

## Packet Format
| Field Name  | Field Type | Notes                                         |
|-------------|------------|-----------------------------------------------|
| Packet Type | `uint8_t`  | The packet type, covered in the next section. |
| Stream ID   | `uint32_t` | Random stream ID assigned by the client.      |
| Payload     | `char[]`   | Payload takes up the rest of the packet.      |

Every packet must follow this format regardless of the type. Note that all data types are little-endian. Additionally, the stream ID of 0 is reserved for the initial handshake and must not be used elsewhere.

## Packet Types
Each packet type has a different format for the payload, which is detailed below.

### `0x01` - CONNECT
#### Payload Format
| Field Name           | Field Type | Notes                                                  |
|----------------------|------------|--------------------------------------------------------|
| Stream Type          | `uint8_t`  | Whether the new stream should use a TCP or UDP socket. |
| Destination Port     | `uint16_t` | Destination TCP/UDP port for the new stream.           |
| Destination Hostname | `char[]`   | Destination hostname, in a UTF-8 string.               |

#### Behavior
The client needs to send a CONNECT packet to the server to create a new stream under the same websocket. The stream ID chosen by the client at this point will be associated with this stream for all future messages. When the server receives this packet, it must validate this information, and if the payload is invalid, a CLOSE packet must be sent.

Once the payload has been validated, the server must immediately try to establish a TCP/UDP socket to the specified hostname and port. If this fails, the server must send a CLOSE packet with the reason. To reduce overall delay, the client can begin sending data before the any CONTINUE packet has been received from the server.

The stream type field determines whether the connection uses TCP or UDP. `0x01` in this field means TCP, and `0x02` means UDP. UDP support is optional for both the server and the client.

### `0x02` - DATA
#### Payload Format
| Field Name     | Field Type | Notes                                                      |
|----------------|------------|------------------------------------------------------------|
| Stream Payload | `char[]`   | The data which is sent to and from the destination server. |

#### Behavior
Any DATA packets sent from the client to the server must be proxied to the TCP/UDP socket associated with the stream ID of the packet. On the server, the received payload must be buffered before being sent to the TCP/UDP socket in order to handle congestion. The size of this send buffer is predetermined and must be the same for every stream. 

Any DATA packets sent from the server to the client must be interpreted as coming from the TCP/UDP socket associated with the stream ID of the packet.

For TCP streams, the server must buffer any packets received from the client in a FIFO queue, and it must keep a separate buffer for each TCP stream.

### `0x03` - CONTINUE
#### Payload Format
| Field Name       | Field Type | Notes                                                                    |
|------------------|------------|--------------------------------------------------------------------------|
| Buffer Remaining | `uint32_t` | The number of packets that the server can buffer for the current stream. |

#### Behavior
If the associated stream is a UDP socket, then CONTINUE packets must not be sent, and the client does not keep track of any buffer for the stream.

When the client receives a CONTINUE packet from the server, it must store the received buffer size. When sending a DATA packet, this value should be decremented by 1 on the client. Once the remaining buffer size reaches zero, the client cannot send any more DATA packets, until it receives another CONTINUE packet which resets this value.

The server must send another CONTINUE packet when it has received the same number of packets from the client as its own maximum buffer size. The server should regularly send CONTINUE packets before this point to ensure minimal delays when receiving data from the client. 

### `0x04` - CLOSE
#### Payload Format
| Field Name   | Field Type | Notes                                  |
|--------------|------------|----------------------------------------|
| Close Reason | `uint8_t`  | The reason for closing the connection. |

#### Behavior
Any CLOSE packets sent from either the server or the client must immediately close the associated stream and TCP socket. The close reason in the payload doesn't affect this behavior, but may provide extra information which is useful for debugging.

#### Client/Server Close Reasons
- `0x01` - Reason unspecified or unknown. Returning a more specific reason should be preferred.
- `0x02` - Voluntary stream closure, which would equate to one side resetting the connection.
- `0x03` - Unexpected stream closure due to a network error.
- `0x04` - Incompatible extensions. This will only be used during the initial handshake.

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

#### Close Reasons Specified by Extensions:
- `0xc0` - Authentication failed due to invalid username/password.
- `0xc1` - Authentication failed due to invalid signature.
- `0xc2` - Authentication required but the client did not provide credentials.

### `0x05` - INFO
#### Payload Format
| Field Name         | Field Type | Notes                                               |
|--------------------|------------|-----------------------------------------------------|
| Major Wisp Version | `uint8_t`  | The major version of the latest supported protocol. |
| Minor Wisp Version | `uint8_t`  | The minor version of the latest supported protocol. |
| Extension Data     | `char[]`   | Data about the supported protocol extensions.       |

#### Behavior
When a Wisp connection is first established, both the server and client must send an INFO packet describing the protocol extensions that it supports. The extension data is represented as an array of extension metadata entries, the format of which is indicated below. If an extension is missing from this packet, it is assumed to not be supported.

The version numbers must be set to the latest protocol version supported by both the client and server. This will match the [Semantic Versioning](https://semver.org/) format.

## Protocol Extensions

### Common Format
| Field Name         | Field Type | Notes                                                                        |
|--------------------|------------|------------------------------------------------------------------------------|
| Extension ID       | `uint8_t`  | The ID of the protocol extension.                                            |
| Payload Length     | `uint32_t` | The length of the payload for the extension metadata, in bytes.              |
| Extension Metadata | `char[]`   | A custom byte array which has information about the status of the extension. |

### `0x01` - UDP
The presence of this extension indicates whether or not the client or server implementation supports UDP. There is no payload.

### `0x02` - Password Authentication
This extension adds password-based authentication to Wisp. A payload is required for this feature. The presence of this extension indicates that the server allows username/password authentication.

Once the server receives the username and password sent by the client, it will check if these credentials match. If they do, the connection will proceed as normal, but if they do not, the connection will be closed by sending a CLOSE packet with a reason of `0xc0`, then closing the underlying websocket.

#### Server Message
| Field Name      | Field Type | Notes                                                  |
|-----------------|------------|--------------------------------------------------------|
| Required        | `uint8_t`  | A boolean that specifies if password auth is required. |

#### Client Message
| Field Name      | Field Type | Notes                                    |
|-----------------|------------|------------------------------------------|
| Username Length | `uint8_t`  | The length of the username string.       |
| Password Length | `uint16_t` | The length of the password string.       |
| Username String | `char[]`   | A UTF-8 encoded string for the username. |
| Password String | `char[]`   | A UTF-8 encoded string for the password. |

### `0x03` - Public/Private Key Authentication
This extension adds public/private key authentication to Wisp. A payload is required for this feature. The presence of this extension on the server message means that the server allows the usage of key authentication.

During the handshake, the server sends a list of supported signature algorithms and a challenge which consists of random bytes. The length of the challenge may vary, but it should be around 512 bits. 

When the client receives this, it will select an algorithm to use, and sign the provided challenge using its private key. The client then sends back the SHA-512 hash of the public key which corresponds to the private key used, as well as the signature. 

When the server receives the response from the client, it must verify the signature using the the public keys it has stored. If the verification is successful for one of the public keys allowed by the server, no more action is needed. If it fails for all of them, the connection must be closed immediately by sending a CLOSE packet with a reason of `0xc1`, then closing the underlying websocket.

Currently, the only supported signature algorithm is Ed25519, and this is represented by a bit mask of `0b00000001` in the payload.

#### Server Message
| Field Name           | Field Type | Notes                                                                                                 |
|----------------------|------------|-------------------------------------------------------------------------------------------------------|
| Required             | `uint8_t`  | A boolean that specifies if public/private key authentication is required.                            |
| Supported Algorithms | `uint8_t`  | A bit mask representing a list of signature algorithms supported by the server.                       |
| Challenge Data       | `char[]`   | The randomly generated challenge data. The length of this may vary and fills the rest of the payload. |

#### Client Message
| Field Name          | Field Type | Notes                                                                                                   |
|---------------------|------------|---------------------------------------------------------------------------------------------------------|
| Selected Algorithm  | `uint8_t`  | A bit mask representing the signature algorithm that was selected by the client.                        |
| Public Key Hash     | `char[64]` | A SHA-512 hash of the public key used. This is always 64 bytes long.                                    |
| Challenge Signature | `char[]`   | A cryptographic signature generated using the client's private key. This fills the rest of the payload. |

### `0x04` - Server MOTD
This extension allows the server to specify a MOTD (a welcome message) to the client. The client can then display this message to the user. The purpose of this is to provide a mechanism for the server to make the user aware of any limitations imposed on the client, such as rate limits and block lists, as well other info.

### Server Message
| Field Name  | Field Type | Notes                               |
|-------------|------------|-------------------------------------|
| MOTD String | `char[]`   | A UTF-8 string containing the MOTD. |

### Client Message
The client does not need to send a payload.

### Note on Authentication Behavior
On the server message for each extension for authentication, there is a field that indicates whether or not that particular auth method is required. If no authentication methods are required, or if the extensions for authentication are not present on the server, the client will assume authentication is optional. If either key or password auth is required, the client must prompt the user for their credentials. If both methods are indicated to be required, the client may choose which one to use. 

## HTTP Behavior
Since the entire protocol takes place over websockets, a few rules need to be in place to ensure that an HTTP connection can be upgraded successfully.

### Server Architecture
The server must consist of a HTTP and websocket server that conforms to the respective standards.

### Choosing a Websocket URL
To ensure compatibility with previous wsproxy implementations, the URL of the websocket should always end with a trailing forward slash (`/`) to prevent confusion with wsproxy endpoints. 

For example, a wsproxy endpoint might look like this: `ws://example.com/customprefix/host:port`

Meanwhile a Wisp endpoint would look like this: `ws://example.com/customprefix/`

It is up to the server implementation to interpret the prefix. It may be ignored, or used for gatekeeping in order to establish basic password-based authentication.

### Establishing a Websocket Connection
The client must establish the connection by performing a standard websocket handshake. The `Sec-WebSocket-Protocol` header does not need to be set.

### Handshake Steps
Immediately after a websocket connection is established, the server must send an INFO packet containing information about the server itself. This includes information about the latest supported Wisp version and a list of supported protocols. Immediately afterwards, the server must send a CONTINUE packet containing the initial buffer size for each stream. The stream ID for this packet must be set to 0. 

The client must wait for the CONTINUE packet to be received from the server for any communications to continue. If it receives the INFO packet first, it must use version 2 of the protocol. If it receives the CONTINUE packet first, it must use version 1 of the protocol. After it has received an INFO packet from the server, the client must then send an INFO packet to the server describing itself and the extensions it supports.

While the server waits for an INFO packet from the client, it must assume the client is using version 1 of the protocol. After it receives the INFO packet from the client, it will assume version 2 of the protocol is used, and it must validate that the extensions are compatible.

If there are any critical compatibility issues detected by either party during the handshake, the connection should be immediately closed with a CLOSE packet with reason `0x04` and a stream ID of 0. 

After these steps have been performed, regular communications can begin between the client and server. Only extensions that both sides support may be used.
