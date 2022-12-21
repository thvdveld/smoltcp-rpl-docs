# Mode of Operation 0: no downward routes maintained

First, we have the network that just woke up:

![Empty RPL network](assets/rpl0.webp)

Then, one node decided that he wants to join a RPL network.
For that it sends a *DODAG Information Solicitation (DIS)* message.
This message is sent to the *all-RPL-node* IPv6 multicast address (`ff02::a1`).
This means that all nodes in the radio range of the sending node will receive it.

![Empty RPL network](assets/rpl1.webp)

Only the root node has information about the RPL DODAG structure.
Therefore, it is the only one that can respond with a *DODAG Information Object (DIO)* packet.
It is also using the `ff02::a1` multicast address.

![Empty RPL network](assets/rpl2.webp)

The nodes that receive the DIO message and did not yet joined a RPL network
will join this network.
They do that by selecting the root node (the sender of the DIO packet) as their parent.

![Empty RPL network](assets/rpl3.webp)

Because the nodes have selected a parent, they can start sending DIO packets as well.

![Empty RPL network](assets/rpl4.webp)

Some UDP:

![Empty RPL network](assets/rpl5.webp)
