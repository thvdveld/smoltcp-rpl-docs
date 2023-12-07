# Introduction

```admonish warning
This website is still under construction.
```

```admonish danger title="Experimental RPL implementation"
RPL support is not yet available in the main branch of smoltcp.
The current RPL implementation is available in the `rpl` branch of smoltcp.
Note that the RPL implementation is still experimental and not yet ready for production.
```

The [Routing Protocol for Low-power and lossy networks (RPL)](https://datatracker.ietf.org/doc/html/rfc6550)
is a routing protocol for low power and lossy networks (LLNs).
It is designed to be used in various application domains such as industrial monitoring,
building automation, connected homes, and smart cities.
RPL is a distance vector routing protocol that builds a Directed Acyclic Graph (DAG) connecting the nodes in the network.
The DAG is used to route packets in the network.
RPL is designed to be used in networks where the nodes are constrained in terms of power, memory, and processing resources.

The goal of this website is to better explain how RPL works and how to use it with [smoltcp](https://github.com/smoltcp-rs/smoltcp).
At the moment, the RPL implementation can be found in the [`rpl` branch](https://github.com/thvdveld/smoltcp/tree/rpl) of smoltcp.

## Implementations of RPL

Contiki-NG is an open-source operating system for the Internet of Things.
It is a fork of the Contiki operating system and supports a wide range of platforms.
Contiki-NG includes an implementation of RPL.
