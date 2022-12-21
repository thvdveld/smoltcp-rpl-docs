# Routing Protocol for Low-Power and Lossy Networks

A Wireless Sensor Network (WSN) is a Low-Power and Lossy Network (LLN) 
and is composed of constrained devices, actuators and sensors.
These devices are connected via a sink node to the Internet and to an end user.
These networks can be composed of hundreds of these tiny battery powered devices
with short radio transmission range.
Thus, intermediate nodes act as relay nodes transmitting data towards the sink (root) node
using a multi-hop path.
Given the low-power and lossy radio links, the battery supplied nodes,
and the multi-hop mesh topologies, routing issues are very challenging for LLNs and WSNs.

An effective routing solution that overcomes some of the challenges posed by WSNs was
developed by the IETF *Routing over Low power and Lossy Networks* working group,
which proposed 
[**IPv6 Routing protocol for Low-power and Lossy Networks (RPL)**](https://datatracker.ietf.org/doc/html/rfc6550).

RPL provides support for *Point to Point (**P2P**)*, *Multipoint to Point (**M2P**)*
and *Point to Multipoint (**P2M**)* traffic.
**P2P** communication takes place between two nodes.
In **M2P** communication, the info is sent from multiple nodes towards a central node.
In **P2M** communication, the info is sent from a central node towards multiple nodes.
To ensure these types of traffic, RPL provisions **upward routes** and **downward routes**.
The upward routes are the ones that head toward the root.
The downward routes are the ones that start at the root and head towards the other
nodes in the network.

## Destination Oriented Directed Acyclic Graph (DODAG)
