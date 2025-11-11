# Wisp Protocol
Wisp is designed to be a low-overhead, easy to implement protocol for proxying multiple TCP sockets over a single websocket connection.

For the protocol specifications, see [`protocol.md`](https://github.com/MercuryWorkshop/wisp-protocol/blob/main/protocol.md).

## Server Implementations:
- **[wisp-server-python](https://github.com/MercuryWorkshop/wisp-server-python)** - by [@ading2210](https://github.com/ading2210)
- **[wisp-server-node](https://github.com/MercuryWorkshop/wisp-server-node)** - by [@ProgrammerIn-wonderland](https://github.com/ProgrammerIn-wonderland)
- [wisp-js](https://github.com/MercuryWorkshop/wisp-client-js/tree/rewrite) - by [@ading2210](https://github.com/ading2210)
- [epoxy-server](https://github.com/MercuryWorkshop/epoxy-tls/tree/multiplexed/server) (Rust) - by [@r58Playz](https://github.com/r58Playz)
- [Woeful](https://github.com/MercuryWorkshop/woeful) (C++) - by [@FoxMoss](https://github.com/MercuryWorkshop/woeful)

## Client Implementations:
- **[wisp-client-js](https://github.com/MercuryWorkshop/wisp-client-js)** - by [@ading2210](https://github.com/ading2210)
- [wisp-mux](https://crates.io/crates/wisp-mux) (Rust) - by [@r58Playz](https://github.com/r58Playz)

## Software Using Wisp:
- [WispMark](https://github.com/MercuryWorkshop/wispmark) - A benchmarking tool for Wisp protocol implementations.
- [Mittens](https://github.com/scaratech/mittens) - Programmable middleware that works with any Wisp implementation.
- [libcurl.js](https://github.com/ading2210/libcurl.js) - A port of libcurl to WebAssembly, for proxying HTTPS requests from the browser with full TLS encryption
- [epoxy-tls](https://github.com/MercuryWorkshop/epoxy-tls) - An encrypted proxy for browser javascript, written in Rust.
- [Ultraviolet](https://github.com/titaniumnetwork-dev/Ultraviolet) - A sophisticated web proxy, which uses either libcurl.js or epoxy. 
- [Scramjet](https://github.com/MercuryWorkshop/scramjet) - An experimental web proxy that aims to be the successor to Ultraviolet.
- [Whisper](https://github.com/MercuryWorkshop/Whisper) - A client for Wisp that exposes the connection over a TUN device.
- [v86](https://github.com/copy/v86) - x86 PC emulator and x86-to-wasm JIT, running in the browser
- [Puter](https://github.com/HeyPuter/puter/) - An advanced, open-source internet operating system designed to be feature-rich, exceptionally fast, and highly extensible

## Copyright:
This repository is licensed under the [Creative Commons Attribution 4.0 International](https://github.com/MercuryWorkshop/wisp-protocol/blob/main/LICENSE) license. The implementations listed above use different licenses.

The Wisp protocol specifications were written by [@ading2210](https://github.com/ading2210).

<img src="https://mirrors.creativecommons.org/presskit/buttons/88x31/png/by.png" alt="creative commons banner" height="64px"/>
