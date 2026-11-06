# Wisp Protocol
Wisp is designed to be a low-overhead, easy to implement protocol for proxying multiple TCP sockets over a single websocket connection.

For the protocol specifications, see [`protocol.md`](https://github.com/MercuryWorkshop/wisp-protocol/blob/v1/protocol.md).

For more info, including an updated list of software using Wisp, see the [Wisp V2 README](https://github.com/MercuryWorkshop/wisp-protocol/blob/v2/README.md).

## Major Versions:
There are two major versions of Wisp:
- [Wisp V1](https://github.com/MercuryWorkshop/wisp-protocol/blob/v1/protocol.md) - The simpler version which is easier to implement, but without advanced features such as authentication.
- [Wisp V2](https://github.com/MercuryWorkshop/wisp-protocol/blob/v2/protocol.md) - Adds a handshake step which allows for adding protocol extensions, password and key authentication, MOTD messages, and more. 

# Server Implementations:
- **[wisp-server-python](https://github.com/MercuryWorkshop/wisp-server-python)** - by [@ading2210](https://github.com/ading2210)
- **[wisp-js/server](https://github.com/MercuryWorkshop/wisp-js)** - by [@ading2210](https://github.com/ading2210)
- [epoxy-server](https://github.com/MercuryWorkshop/epoxy-tls/tree/multiplexed/server) (Rust) - by [@r58Playz](https://github.com/r58Playz)
- [Woeful](https://github.com/MercuryWorkshop/Woeful) (C++) - by [@FoxMoss](https://github.com/FoxMoss)

## Client Implementations:
- **[wisp-js/client](https://github.com/MercuryWorkshop/wisp-js)** - by [@ading2210](https://github.com/ading2210)
- [wisp-mux](https://crates.io/crates/wisp-mux) (Rust) - by [@r58Playz](https://github.com/r58Playz)

## Software Using Wisp:
- [libcurl.js](https://github.com/ading2210/libcurl.js) - A port of libcurl to WebAssembly, for proxying HTTPS requests from the browser with full TLS encryption
- [epoxy-tls](https://github.com/MercuryWorkshop/epoxy-tls) - An encrypted proxy for browser javascript, written in Rust.
- [Ultraviolet](https://github.com/titaniumnetwork-dev/Ultraviolet/) - A sophisticated web proxy, which uses either libcurl.js or epoxy. 
- [Whisper](https://github.com/MercuryWorkshop/Whisper) - A client for Wisp that exposes the connection over a TUN device.

## Copyright:
This repository is licensed under the [Creative Commons Attribution 4.0 International](https://github.com/MercuryWorkshop/wisp-protocol/blob/main/LICENSE) license. The implementations listed above use different licenses.

The Wisp protocol specifications were written by [@ading2210](https://github.com/ading2210).

<img src="https://mirrors.creativecommons.org/presskit/buttons/88x31/png/by.png" alt="creative commons banner" height="64px"/>
