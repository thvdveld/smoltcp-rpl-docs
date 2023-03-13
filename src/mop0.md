# Mode of Operation 0: no downward routes maintained

First, there are constrained devices (nodes) that just woke up. One of this constrained devices
is assigned to be the root:

![Empty RPL network](assets/rpl_mop_0/rpl0.svg)

Then, one node decided that he wants to join a RPL network.
For that it sends a *DODAG Information Solicitation (DIS)* message.
This message is sent to the *all-RPL-node* IPv6 multicast address (`ff02::a1`).
This means that all nodes in the radio range of the sending node will receive it.

In the image below, node 1 is the one that decided to join a RPL network. The root node (R) 
and nodes 2 and 3 are in its radio range. 

![Empty RPL network](assets/rpl_mop_0/rpl1.svg)

In the begining, only the root node has information about the RPL DODAG.
Therefore, it is the only one that can respond with a *DODAG Information Object (DIO)* packet.
The root node is also using the `ff02::a1` multicast address. Thus, the *DIO* message will be received
by all the nodes in its radio range, including node 1.

![Empty RPL network](assets/rpl_mop_0/rpl2.svg)

The nodes that received the *DIO* message and did not yet join a RPL network
will join this network.
They do that by selecting the root node (the sender of the *DIO* packet) as their parent. 
The nodes that join the network will also set their rank. The rank is computed based on the information received 
in the *DIO* message and link/node metrics with the help of the objective function.

![Empty RPL network](assets/rpl_mop_0/rpl3.svg)

Because the nodes have selected a parent, they can start sending *DIO* packets as well. As stated before the *DIO* messages
are sent via the *all-RPL-node* IPv6 multicast address. This means that all the nodes in the radio range of a node will receive
the *DIO* message. When node 1 is sending the *DIO* message, the root, its current parent, will also receive it. 
The root node will ignore this message. This is because all nodes in a DODAG must ignore *DIO* messages from nodes 
with a higher rank, which guarantees optimal route selection and avoids loops.


![Empty RPL network](assets/rpl_mop_0/rpl4.svg)

## Maintenance of a RPL network ##

The *DIO* messages will be sent by nodes even after the DODAG is formed. The goal of sending *DIO* messages for the whole 
life of a RPL network is to maintain or upgrade the formed DODAG. However, the constant generation of control messages 
can consume a lot of energy. To preserve resources, the sending of control messages is minimized and only done when necessary,
through the use of the **Trickle Timer algorithm**.

The **Trickle Timer algorithm** monitors the consistency of packet exchange in the network. If the pattern is consistent 
and free of redundant or outdated data, the Trickle Timer decreases the rate of sending *DIO* messages exponentially.
However, if there are any inconsistencies in the network, the next *DIO* message is rescheduled and sent at the shortest 
possible time interval. In other words, this algorithm ensures that *DIOs* are aggressively advertised when the 
network is unstable and advertised at a slower pace when it is stable.

There are four parameters that control the functioning of the Trickle Timer:

- \\(I_{min}\\), which represents the minimum time interval between two *DIO* messages

- \\(I_{max}\\), which represents the maximum time interval between two *DIO* message

- \\(k\\), which represents the redundancy constant (the number of redundant control messages)

- \\(I\\), which represents the size of the current time-interval

In the beginning, \\(I\\) is set to a random value between \\(I_{min}\\) and \\(2 \cdot I_{min}\\).
*DIO* messages are sent when \\(I\\) expires and if the
counter (\\(c\\)) that keeps track of the consistent received messages is smaller than \\(k\\). 
If the network is stable, \\(I\\) is doubled until it reaches \\(I_{max}\\).
However, if inconsistencies are detected in the network, 
\\(I\\) is reset to a value between \\(I_{min}\\) and \\(2 \cdot I_{min}\\).
This ensures efficient use of resources while still being able to ensure the maintainance of the network.

## M2P communication ##

In **MOP 0**, the network is configured such that only upward routes are established. This means that 
any node in the network can transmit messages to the root node or any other node on its path towards the root, 
but not to other nodes in the network. The default route, represented by the parent of the node, is used to
transmit data packets in an upward direction. For example, node 5 can transmit messages to the root node 
or to nodes 2 and 1 (which are on the path towards the root) but cannot send messages to any other nodes within the network.

![Empty RPL network](assets/rpl_mop_0/rpl5.svg)
