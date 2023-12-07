# RPL

Routing Protocol for Low-Power and lossy networks (RPL), defined in [RFC 6550](),
is a routing protocol for Low-Power and Lossy Networks (LLNs)
These networks are characterized by:
- low bandwidth;
- high packet loss rates;
- hgih latency.

RPL is a distance vector routing protocol that builds a
Destination Oriented Directed Acyclic Graph (DODAG) rooted at a single destination.
The DODAG is built using control messages, specifically ICMPv6 messages.
RPL supports *Point to Point (**P2P**)*, *Multipoint to Point (**M2P**)* and *Point to Multipoint (**P2M**)* traffic. 
**P2P** communication occurs between two nodes, **M2P** communication involves 
data being sent from multiple nodes to a central node, and **P2M** communication 
involves data being sent from a central node to multiple nodes.
RPL uses **upward** and **downward routes** to facilitate these different types of traffic.
Upward routes lead towards the root of the network,
while downward routes start at the root and lead towards other nodes in the network.

#### Destination Oriented Directed Acyclic Graph (DODAG)

RPL specifies how to construct a Directed Acyclic Graph (DAG) rooted at a single destination, 
known as a Destination-Oriented DAG (DODAG), using an objective function and a set of metrics and constraints. 
The objective function evaluates a combination of metrics and constraints to determine 
the _best_ path for packets to follow.

#### Rank

The Rank of a node is a measure of its distance from the root node.
The rank is determined using routing metrics, which include characteristics of both the links
(such as throughput, latency, link reliability, expected transmission count, link quality level)
and the nodes (such as energy state).
These metrics are used in objective functions to calculate the cost of a route between nodes.

#### Parents

The parents of a node are all its neighbors that are part of a possible route to the sink node.
A neighbor is a node that can be reached via single hop radio link.
The preferred parent of a node is the neighbor that is on the best route from the node to the sink,
depending on the objective function used.

#### RPL Instance

The RPL Instance is a group of DODAGs that have the same RPL Instance ID.
All DODAGs in the same RPL Instance share the same objective function.

#### DODAG ID

The DODAG ID is a globally unique identifier for the DODAG.
This is one of the routable IPv6 address of the root node of the DODAG.

#### Version number

The version number is a counter that is incremented by the root node whenever a new version of the DODAG is created.

## Mode of Operations

RPL supports 4 mode of operations (MOPs):
- MOP 0: No downward routes maintained
- MOP 1: Non-storing mode
- MOP 2: Storing mode with no multicast supports
- MOP 3: Storing mode with multicast support

In MOP0, only upward routes are maintained.
In MOP1, MOP2 and MOP3, both upward and downward routes are maintained.
In MOP1, only the root node maintains downward routes.
In MOP2 and MOP3, all nodes maintain downward routes.
In MOP3, multicast is supported.

## Control Messages

The DODAG is created with the assistance of control messages, specifically ICMPv6 messages.
The four types of control messages that RPL uses to construct the DODAG are:
- DODAG Information Solicitation (**DIS**).
- DODAG Information Object (**DIO**)
- Destination Advertisement Object (**DAO**)
- Destination Advertisement Object Acknowledgment (**DAO-ACK**)
