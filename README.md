# Wisp Protocol
Wisp is designed to be a low-overhead, easy to implement protocol for proxying multiple TCP sockets over a single websocket connection.

For the protocol specifications, see [`protocol.md`](https://github.com/MercuryWorkshop/wisp-protocol/blob/main/protocol.md).

## Server Implementations:
- **[wisp-server-python](https://github.com/MercuryWorkshop/wisp-server-python)** - by [@ading2210](https://github.com/ading2210)
- **[wisp-server-node](https://github.com/MercuryWorkshop/wisp-server-node)** - by [@ProgrammerIn-wonderland](https://github.com/ProgrammerIn-wonderland)
- [epoxy-server](https://github.com/MercuryWorkshop/epoxy-tls/tree/multiplexed/server) (Rust) - by [@r58Playz](https://github.com/r58Playz)
- [WispServerCpp](https://github.com/FoxMoss/WispServerCpp) - by [@FoxMoss](https://github.com/FoxMoss)

## Client Implementations:
- **[wisp-client-js](https://github.com/MercuryWorkshop/wisp-client-js)** - by [@ading2210](https://github.com/ading2210)
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
