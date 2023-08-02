# Optimizations


<!--## DODAG Configuration in DIO-->

<!--DIO messages MAY contain a DODAG configuration option (16 bytes).-->
<!--In smoltcp, we only add this option when a multicast DIO message is sent when the Trickle timer is in the first interval.-->
<!--A Trickle timer is in the first interval at startup or when it is reset.-->

<!--When a node receives a DIO without the DODAG Configuration option as a response on a multicast DIS,-->
<!--the node will send a unicast DIS.-->
<!--Responding on a unicast DIS is always a unicast DIO containing a DODAG Configuration option.-->

<!--It is important for a node to receive a DODAG Configuration option when it has never joined a network.-->
<!--This option contains information related to the objective function and trickle timer.-->

<!--If the objective function is different, this node can only join as a leaf and not act as a router in the network.-->

<!--Note that Contiki-NG always sends DIO messages containing a DODAG Configuration option.-->
<!--However, we think that this is a (tiny) waste of power.-->
<!--This is something that we need to measure.-->


## Trickle timer implementation

The Trickle timer that is implemented in smoltcp uses the [enhanced Trickle](https://d1wqtxts1xzle7.cloudfront.net/71402623/E-Trickle_Enhanced_Trickle_Algorithm_for20211005-2078-1ckh34a.pdf?1633439582=&response-content-disposition=inline%3B+filename%3DE_Trickle_Enhanced_Trickle_Algorithm_for.pdf&Expires=1690468547&Signature=GWAtYVYOGyXrGmy~PHBDmcjxtgjGv93RuCCTxDcW1x3gWIlGw2DIxXMXluHJhO5vcR8HR~4qW5zMUmYw0fcrYvvoWrbBOAVxWs5MVF3gr8rTFsenuuSdG9Gi8OQFHnHjG8-p7~0RfHlGU5hxednaKu-dt5ECfzhsCfbfeTTCRk4Zm~CjDW4eAimwRpxuGZ9SWoySnnbOCurAtijdcdqw~YVuJV5M7VunKXgDPjEyEnCEAuwpLurPMvg7sAOpJeOaM7Yz7qCvGAe-oG9Cr8xE805TCxCcWRkOlKQHWFq6r1bK3htwgESB6iumT7Y28IbFvCYQZ7gpFviIH13jBeXFNA__&Key-Pair-Id=APKAJLOHF5GGSLRBV4ZA) timer.

Note that Contiki-NG also implements this timer.
