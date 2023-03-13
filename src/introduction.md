# Introduction

This website provides comprehensive documentation on how to use [smoltcp](https://github.com/smoltcp-rs/smoltcp).
We also discuss how to use the [Routing Protocol for Low-power and lossy networks (RPL)](https://datatracker.ietf.org/doc/html/rfc6550)
that recently gained support in smoltcp.

# Smoltcp

[smoltcp](https://github.com/smoltcp-rs/smoltcp) is a standalone, event-driven TCP/IP stack that is designed for bare-metal, real-time systems.
Its design goals are simplicity and robustness.
Its design anti-goals include complicated compile-time computations,
such as macro or type tricks, even at cost of performance degradation.
**smoltcp** is missing many widely deployed features,
usually because no one implemented them yet.

# RPL

The RPL protocol ([RFC6550](https://datatracker.ietf.org/doc/html/rfc6550))
is designed for networks that usually consist of low power devices and
where the network is generally susceptible to packet loss.
Read [here](./rpl_introduction.md) if you want to know more about how the protocol works.
