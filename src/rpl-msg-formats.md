# Control Message Formats

<script> document.createElement("warn"); </script>
<script> document.createElement("success"); </script>

<style>
:root {
    --red:   rgb(198, 0, 26);
    --green: rgb(0, 118, 85);
}

warn {
    color: var(--red);
    font-weight: bold;
}

success {
    color: var(--green);
    font-weight: bold;
}
</style>

## DODAG Information Solicitation (DIS)

Sent by nodes that are not part of a RPL network to request information
about existing DODAGs in range of the node.
It is broadcasted, however, it can be unicast if it is for probing.
Using the [Solicited Information](#solicited-information) option, the DODAGs are filtered,
i.e., filtering on Version number, RPL Instance ID and/or DODAG ID.

#### Packet structure

```txt
 0                   1                   2
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     Flags     |   Reserved    |   Option(s)...
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- **Flags**: unused field, must be set to zero.
- **Reserved**: unused field, must be set to zero.
- **Option(s)**: [Pad1](#pad1), [PadN](#padn), [Solicited Information](#solicited-information).

**Scope**: all-RPL-nodes multicast address, or link-local unicast address

#### Sending a DIS

1. The node **is** a ROOT node:
    - A ROOT node should never send a DIS, unless it is [probing].
      When probing, the DIS message is a unicast message.
2. The node **is** a LEAF node:
    - Never sends a DIS.
3. The node **is not** a LEAF node:
    - The node **is not** part of a DODAG:
        - Transmit a multicast DIS message.
          Usually this is only sent after 5 seconds and then every 60 seconds.
          The DIS message MAY contain a [Solicited Information](#solicited-information) option,
          which is used for filtering DODAGs.
    - The node **is** part of a DODAG:
        - The node should not send a DIS, unless it is [probing].
          When probing, the DIS message is a unicast message.

#### Receiving a DIS

1. The node **is** a ROOT node:
    - Check if the [Solicited Information](#solicited-information) option is present.
      This option has predicates that we need to match on.
    - After matching the predicates:
        - **Multicast** DIS: reset the [Trickle] timer.
          When the timer expires, a <warn>multicast</warn> [DIO](#dodag-information-object-dio) message is transmitted.
        - **Unicast** DIS: **do not** reset the [Trickle] timer, but respond with a <warn>unicast</warn> [DIO](#dodag-information-object-dio) message,
          which MUST contain a [DODAG Configuration](#dodag-configuration) option.
2. The node **is** a LEAF node:
    - Ignore the DIS message.
3. The node **is not** a LEAF node:
    - The node **is** part of a DODAG: handle the message like the ROOT node.
    - The node **is not** part of a DODAG: <warn>ignore</warn> the message.

---

## DODAG Information Object (DIO)

Sent by nodes that are part of a RPL network.
It is broadcasted, however, it can be unicast if it is for probing.
They send this message periodicaly, based on the Trickle timer.
They are also sent as a response on a [DIS](#dodag-information-solicitation-dis) message.
The message contains information about the DODAG it is part of.
More information is usually contained in a [DODAG Configuration](#dodag-configuration) option.

#### Packet structure

```txt
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| RPLInstanceID |Version Number |             Rank              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|G|0| MOP | Prf |     DTSN      |     Flags     |   Reserved    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
+                                                               +
|                                                               |
+                            DODAGID                            +
|                                                               |
+                                                               +
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   Option(s)...
+-+-+-+-+-+-+-+-+
```

- **RplInstanceId**: the RPL instance ID.
- **Version Number** the version of the RPL isntance, specifies an iteration ([Section 8.2]()).
- **Rank**: shows a relative distance from the advertising node to the root ([Section 8.2]()).
- **G (Grounded)**: indicates if the scope of the application is met or not (application specific and in our case always 0).
- **0**: unused field, must be set to zero.
- **MOP (Mode Of Operation)**: the mode of operation of the DODAG.
- **Prf (Preference)**: indicates how preferable the root node is (0 is least preferred) ([Section 8.2]()).
- **DTSN (DAO Trigger Sequence Number)**: used to maintain downward routes ([Section 9]()).
- **Flags**: unused field, must be set to zero.
- **Reserved**: unused field, must be set to zero.
- **DODAGID**: the identifier of the root, which is the IPv6 address of the root.
- **Options(s)**: [Pad1](#pad1), [PadN](#padn), [DAG Metric Container](#dag-metric-container), [Route Information](#route-information), [DODAG Configuration](#dodag-configuration), [Prefix Information](#prefix-information).

**Scope**: all-RPL-nodes multicast address, or link-local unicast address

#### Sending a DIO

1. The node is a ROOT node:
    - Based on the Trickle timer a <warn>multicast</warn> DIO message is transmitted,
      containing a [DODAG Configuration](#dodag-configuration) option.
      Note that the standard does not require this.
      However, the option contains useful information about the Trickle timer and the
      objective function.

      If nodes in range, not connected to a DODAG, are interested in this option
      (they require to know the objective function that is used),
      then they will send a <warn>unicast</warn> [DIS](#dodag-information-solicitation-dis) message.
      As a response, a <warn>unicast</warn> DIO message is sent, which MUST contain a 
      [DODAG Configuration](#dodag-configuration) option.
      If the number of interested nodes is high, then the overhead of the exchange of 
      <warn>unicast</warn> messages might be significant.

      The following values are set by the user of the library:
        - RPL Instance ID
        - Version number
        - Grounded (G)
        - Mode of Operation (MOP)
        - Preference (Prf)
        - DODAG ID

      The Rank is equal to the ROOT Rank value.
      The DTSN is a sequence counter starting with the default value, 
      [incremented] when new route information is required. 


2. The node **is** a LEAF node:
    - A LEAF node does not transmit a DIO [Section 8.5].

3. The node **is not** a LEAF node:
    - The node **is** part of a DODAG: 
      behave as the ROOT node, however, the DIO information was obtained when receiving a DIO message
      from a node with a lower Rank.
      The Rank was computed when selecting a parent.
      The DTSN is a sequence counter starting with the default value,
      [incremented] when new route information is required. 

    - The node **is not** part of a DODAG: the node does not send a DIO.

#### Receiving a DIO

1. The node is a ROOT node:
    - The DIO is ignored.

2. The node **is** part of a DODAG:
    - Ignore if the Rank is higher than ours.
    - Add sender to the parent list and check for new preferred parent.
        - Selected a new preferred parent:
            - MOP 1: Send DAO message to the ROOT node. The parent address is in the DAO [Transit Information](#transit-information) option.
            - MOP 2-3: 
                - Send No-path DAO message to old preferred parent.
                - Send DAO message to the new preferred parent (which is forwarded to the ROOT node).
                  The destination needs to be the preferred parent, since this contains the 
                  information about who the parent is.
                  There is no [Transit Information](#transit-information) option.

3. The node **is not** part of a DODAG:
    - The node **is** a leaf:
        - Accept the DIO and select sender as a parent.
        - <success>The node is now part of a DODAG</success>.
    - The node **is not** a leaf:
        - A [DODAG Configuration](#dodag-configuration) option **is** present:
            - The node accepts the DIO, iff the mode of operation and objective function is the same.
              <warn>The mode of operation and objective function(s) are set by the developer</warn>.
              The node then joins the network:
                - Copy the following fields (<warn>a non-root node cannot modify these</warn>):
                    - Grounded (G)
                    - Mode of Operation (MOP)
                    - Preference (Prf)
                    - Version number
                    - RPL Instance ID
                    - DODAG ID
                - Select the sender of the DIO as a parent and send DAOs accordingly (see step 2).
                - Calculate a Rank using the Objective Function.
                - Reset the Trickle timer to make sure that this node now also sends DIOs.
                - <success>The node is now part of a DODAG</success>.
        - A [DODAG Configuration](#dodag-configuration) option **is not** present:
            - Transmit a <warn>unicast</warn> [DIS](#dodag-information-soliciation-dis) message to gain information about the used objective function.

---

## Destination Advertisement Object (DAO)

Sent by nodes that have selected a parent.
For Mode of Operation 1 (non-storing), the message is sent to the DODAG root.
For Mode of Operation 2 and 3, the message is sent to the preferred parent.
It is sent to propagate routing information.

#### Packet structure

```txt
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| RPLInstanceID |K|D|   Flags   |   Reserved    | DAOSequence   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
+                                                               +
|                                                               |
+                            DODAGID*                           +
|                                                               |
+                                                               +
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   Option(s)...
+-+-+-+-+-+-+-+-+
```

- **RplInstanceId**: the RPL instance ID.
- **K**: if set, an acknowledgement is requested.
- **D**: if set, the DODAGID is present in the message.
- **Flags**: unused field, must be set to zero.
- **Reserved**: unused field, must be set to zero.
- **DAOSequence**: indicates the freshness of the information.
    It is incremented for each unique DAO message, echoed in the DAO-ACK.
- **DODAGID**: the identifier of the root, which is the IPv6 address of the root.
- **Options**: [Pad1](#pad1), [PadN](#padn), [RPL Target](#rpl-target), [Transit Information](#transit-information), [RPL Target Descriptor](#rpl-target-descriptor).

**Scope**: in **MOP 1**, global or unique-local addresses are used for the source and
  destination address.
  In other modes, the addresses are link-local addresses.

#### Sending a DAO

1. The node is a ROOT node:
    - The ROOT node should **not** send a DAO.

2. The node **is** part of a DODAG:
    - First parent selection:
        - **MOP 1**: send a DAO to the <warn>DODAG root</warn>,
          containing a [Transit Information](#transit-information) option and a [RPL Target](#rpl-target) option.
          The Transit Information option copntians the parent and the route lifetime.
          The RPL Target option contains the original sender IPv6 prefix **?** address of the DAO message.

        - **MOP 2 - 3**: send a DAO to the <warn>preferred</warn> parent,
          containing a [Transit Information](#transit-information) option and a [RPL Target](#rpl-target) option.
          The Transit Information option only contains the route lifetime and not the parent address.
          The parent address is already in the IPv6 destination address.
          The RPL Target option contains the original sender IPv6 prefix **?** address of the DAO message.

    - Switched parent:
        - **MOP 1**: send a DAO to the <warn>DODAG root</warn>,
          containing a [Transit Information](#transit-information) option and a [RPL Target](#rpl-target) option.
          The Transit Information option copntians the parent and the route lifetime.
          The RPL Target option contains the original sender IPv6 prefix **?** address of the DAO message.

        - **MOP 2 - 3**: 
            - Send a No-Path DAO to the old parent,
              containing a [Transit Information](#transit-information) option and a [RPL Target](#rpl-target) option.
              The Transit Information option has a <warn>path lifetime of 0x00</warn>.
            - Send a DAO to the <warn>preferred</warn> parent,
              containing a [Transit Information](#transit-information) option and a [RPL Target](#rpl-target) option.
              The Transit Information option only contains the route lifetime and not the parent address.
              The parent address is already in the IPv6 destination address.
              The RPL Target option contains the original sender IPv6 prefix **?** address of the DAO message.

    - Maintanance of the route:
        - **MOP 1**: the ROOT node increments the DTSN in the DIO messages.
          When a node receives a DIO from the its parent with a new DTSN,
          it should transmit a DAO to the ROOT, as previously explained.
          <warn>In smoltcp, the ROOT never increments the DTSN, and each node just keeps track of a DAO expiration timer,
          just like **MOP 2 - 3**</warn>.

        - **MOP 2 - 3**: each non-leaf and non-ROOT node should keep track of a DAO expiration timer.
          When this timer expires, the node needs to transmit a DAO to its preferred parent,
          as previously explained.
          The DAO retransmition should be one or two minutes before the expriation of the route.
    
    Every time a DAO message is transmitted, the DAO Sequence number is incremented.
    <warn>smoltcp always enables the acknowledgement request flag. The lifetime of the route is 30 minutes. Missing 
    a DAO is significant</warn>.

3. The node **is not** part of a DODADG:
    - The node should **not** send a DAO since it has not selected a parent.

#### Receiving a DAO

1. The node is a ROOT node:
    - Store the route in the routing table.

2. The node **is** a LEAF node:
    - Should never receive a DAO, so ignore the message.

3. The node **is not** a LEAF node:
    - **MOP 1**: all DAO messages are forwared to the default route,
      untill it reaches the destination (which should be the ROOT).
    - **MOP 2**:
        - Store the information in the routing table.
        - If the information is new, create a new DAO message which is sent to the preferred parent.
          Information is new when:
            - It has a newer Path Sequence number.
            - It has additional Path Control bits.
            - It is a No-Path DAO message.
    
    - Node is the destination address and ACK is requested:
        - Transmit a DAO-ACK to the source of the DAO, with the correct status flags.

---

## DAO Acknowledgement (DAO-ACK)

The DAO-ACK is a response on a DAO that requested an acknowledgement.
They are unicast messages.

#### Packet structure

```txt
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| RPLInstanceID |D|  Reserved   |  DAOSequence  |    Status     |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
+                                                               +
|                                                               |
+                            DODAGID*                           +
|                                                               |
+                                                               +
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   Option(s)...
+-+-+-+-+-+-+-+-+
```

- **RplInstanceId**: the RPL instance ID.
- **D**: if set, the DODAGID is present in the message.
- **Reserved**: unused field, must be set to zero.
- **DAOSequence**: correlates to the DAO message.
- **Status**: indicates the completion.
    - Value of 0: the parent accepts to be the parent.
    - Value of 1-127: the parent suggests to find another parent.
    - Value of 127-255: the parent rejects to be the parent.
- **DODAGID**: the identifier of the root, which is the IPv6 address of the root.
- **Options**: [Pad1](#pad1), [PadN](#padn).

**Scope**: in **MOP 1**, global or unique-local addresses are used for the source and
  destination address.
  In other modes, the addresses are link-local addresses.

#### Sending a DAO-ACK

When the `K` flag is set to 1 in the [DAO](#destination-advertisement-object-dao) message, respond with a DAO-ACK with the correct
status.


#### Receiving a DAO-ACK

1. The node is the destination: mark the DAO as acknowledged.
    - Status 0: the parent is selected.
    - Status 1 - 127: remove the parent from the parent set if the parent set has more than 1 parent entry.
      This is not a hard requirement.
    - Status 127 - 255: remove the parent from the parent set.

2. The node is not the destination: forward the message if the node is not a LEAF node.

---

## Options


### Pad1

Used for option alignement.

```txt
 0
 0 1 2 3 4 5 6 7
+-+-+-+-+-+-+-+-+
|   Type = 0x00 |
+-+-+-+-+-+-+-+-+
```

### PadN

Used for option alignement.

```txt
 0                   1                   2
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+- - - - - - - -
|   Type = 0x01 | Option Length | 0x00 Padding...
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+- - - - - - - -

```

### DAG Metric Container

Contains metrics about the network.
There can be more than one DAG Metric Container option.
More information about metrics are in [RFC6551: Routing Metrics Used for Path Calculation in Low-Power and Lossy Networks](https://datatracker.ietf.org/doc/html/rfc6551)
```txt
 0                   1                   2
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+- - - - - - - -
|   Type = 0x02 | Option Length | Metric Data
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+- - - - - - - -
```

### Route Information

<warn>smoltcp does not yet use Route Information Options</warn>.
More information can be found in [Section 6.7.5](https://datatracker.ietf.org/doc/html/rfc6550#section-6.7.5).

```txt
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   Type = 0x03 | Option Length | Prefix Length |Resvd|Prf|Resvd|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Route Lifetime                         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
.                   Prefix (Variable Length)                    .
.                                                               .
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### DODAG Configuration

This option contains information about the DODAG.
This information usually does not change within a DODAG and it is not
necassary to include it in each [DIO].
The ROOT is the only node that sets this information.

```txt
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   Type = 0x04 |Opt Length = 14| Flags |A| PCS | DIOIntDoubl.  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  DIOIntMin.   |   DIORedun.   |        MaxRankIncrease        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|      MinHopRankIncrease       |              OCP              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   Reserved    | Def. Lifetime |      Lifetime Unit            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

```

- **Flags**: unused, set to zero, ignored.
- **A**: 0 if there is no security.
- **PCS (Path Control Size)**: number of bits to be allocated to the path control field.
- **DIOIntDoubl.**: variable used by the Trickle timer.
- **DIOIntMin.**: variable used by the Trickle timer.
- **DIORedun.**: variable used by the Trikle timer.
- **MaxRankIncrease**: variable used by the objective function.
- **MinHopRankIncrease**: variable used by the objective function.
- **OCP (Objective Code Point)**: identifies what objective function is used by the DODAG.
- **Reserved**: unused, set to zero, ignored.
- **Def. Lifetime**: lifetime that is used as default for all RPL routes in the routing tables.
- **Lifetime Unit**: lifetime unit in seconds that is used for all RPL routes in the routing tables.

### RPL Target

This option contains information about the child.
The message is only used in combination wiath a Transit Information option.
The target prefix field contains the global or unique-local IPv6 address.

```txt
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   Type = 0x05 | Option Length |     Flags     | Prefix Length |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
+                                                               +
|                Target Prefix (Variable Length)                |
.                                                               .
.                                                               .
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

```

- **Flags**: unused, set to zero, ignored.
- **Prefix Length**: number of valid leading bits of the prefix.
- **Target Prefix**: contains the prefix or the full IPv6 address of the child node.

### Transit Information

This option contains information about the lifetime of a route, the parent used in the route
or to invalidate a route.

```txt

 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   Type = 0x06 | Option Length |E|    Flags    | Path Control  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Path Sequence | Path Lifetime |                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                               +
|                                                               |
+                                                               +
|                                                               |
+                        Parent Address*                        +
|                                                               |
+                               +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- **E (External)**: indicates that the parent router redistributes external targets into the RPL network.
  An External Target is a Target that has been learned through an alternate protocol.
- **Flags**: unused, set to zero, ignored.
- **Path Control**: limits the number of DAO parents to which a DAO message advertises connectivity and communicate
  preference among the parents.
- **Path Sequence**: increased whenever information in this option is newer information.
- **Path Lifetime**: the length of time in Lifetime Units that the prefix is valid for route determination.
- **Parent Address**: 
    - **MOP 1**: the address of the parent.
    - **MOP 2 - 3**: field is not present.

### Solicited Information

This option is used when sending a DIS to filter potential DODAGs.
When receiving such an option, a node will check if it matches the predicates.

```txt

 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   Type = 0x07 |Opt Length = 19| RPLInstanceID |V|I|D|  Flags  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
+                                                               +
|                                                               |
+                            DODAGID                            +
|                                                               |
+                                                               +
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version Number |
+-+-+-+-+-+-+-+-+
```

- **RPLInstanceID**: The RPL Instance ID predicate.
- **V**: Filter using the Version number.
- **I**: Filter using the RPL Instance ID.
- **D**: Filter using the DODAG ID.
- **Flags**: unused, set to zero, ignored.
- **DODAGID**: The DODAG ID predicate.
- **Version Number**: The Version Number predicate.

### Prefix Information

This option is used by a router to indicate nodes in the DODAG what 
prefix to use.

```txt
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   Type = 0x08 |Opt Length = 30| Prefix Length |L|A|R|Reserved1|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                         Valid Lifetime                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                       Preferred Lifetime                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           Reserved2                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
+                                                               +
|                                                               |
+                            Prefix                             +
|                                                               |
+                                                               +
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- **Prefix Length**: the number of leading bits in the Prefix field that are valid.
- **L**: indicates that this prefix can be used for on-link determination.
- **A**: indicates that this prefix can be used for stateless address configuration.
- **R**: indicaties that the prefix field contains a complete IPv6 address assigned to the sending router.
- **Reserved1**: unused, set to zero, ignored.
- **Valid Lifetime**: time in seconds that the prefix is valid for the purpose of on-link determination.
- **Preferred Lifetime**: time in seconds that the addresses generated with the prefix with SLAAC are valid.
  This should not be higher than the valid lifetime.
- **Reserved2**: unused, set to zero, ignored.
- **Prefix**: an IPv6 address or a prefix of an IPv6 address.

### RPL Target Descriptor

This option is used to tag a target.
When used, it is always associated with a RPL Target option.
It must be copied by other nodes in the network and not modified.
<warn>smoltcp does not use this option</warn>.

```txt
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   Type = 0x09 |Opt Length = 4 |           Descriptor
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       Descriptor (cont.)       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

```

---

## IPv6 Hop-by-Hop RPL option

---

## IPv6 Source routing header

---

## Incrementing the DTSN

**MOP 1**: If the DIO from the parent of a node has increased DTSN, then the node itself has to increment its DTSN.

**MOP 2 - 3**: If the DIO from the parent of a node has increased DTSN, it is not

## Incrementing the Path Sequence Number
