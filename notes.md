# Number Systems
## Value Conversions
Hex | Dec | Bin
--- | --- | ---
0|0|0000
1|1|0001
2|2|0010
3|3|0011
4|4|0100
5|5|0101
6|6|0110
7|7|0111
8|8|1000
9|9|1001
A|10|1010
B|11|1011
C|12|1100
D|13|1101
E|14|1110
F|15|1111

# eve-ng commands
## Troubleshooting
- Kill all proc w/ name pattern.  Helps when GUI cannot stop a node.

```pkill -f linux_l2-adventerprisek9-ms.nov3_2015_high_iron.bin```

# Ethernet
## Foundation Topics
### RJ-45 Pinouts and Wiring
- Most 10BASE-T and 100BASE-TX use two twisted pairs.  One for TX, one for RX.

Two EIE/TIA cabling specifications are: T568A and T568B.

PC network interface card (NIC) transmits on pair 1,2 and receives on pair 3,6; switch ports do the opposite.

**Type of Cable:**
- Straight-through: T568A or T568B on both ends.
- Crossover: T568A on one end, and T568B on other end.
  - When the two devices on the ends of the cable both transmit using the same pins, a crossover cable is required.
  - Crossover cables can also be used between a pair of PCs, swapping the transmit pair on one end (1,2) with the receive pins at the other end (3,6).

**Auto-MDIX** (automatic medium-dependent interface crossover): detects the wrong cable and causes the switch to swap the pair it uses for transmitting and receiving, which solves the cabling problem.

*This feature is not supported on all Cisco switch models*

### Autonegotiation, Speed, and Duplex
By default, each Cisco switch port uses Ethernet autonegotiation to determine the speed and duplex setting (half or full).
To disable autonegotiation on a Cisco switch port, you simply need to statically configure the speed and the duplex settings.
* Can set speed with sub-interface command
```
Switch(config)# interface fastethernet 5/4
 Switch(config-if)# speed speed {auto | 10 | 100 | 1000}
 ```
* Can set duplex with sub-interface command

*Note: you cannot set duplex until the speed is manually configured*
```
Switch(config)# interface fastethernet 5/4
 Switch(config-if)# duplex duplex {auto | half | full}
 ```

CDP exchanges duplex information and can spot an issue.
```
%CDP-4-DUPLEX_MISMATCH: duplex mismatch discovered on FastEthernet0/13                   
(not full duplex), with Switch1 FastEthernet0/13 (full duplex).
```

Interface counters may indicate duplex mismatch between switch and device.
```
switch1# sh int fa0/2
! Lines omitted for brevity   
54 output errors, 5 collisions, 0 interface resets
0 babbles, 54 late collision, 59 deferred
```
### Type of Ethernet

|Type of Ethernet| General Descripton |
|-|-|
|10BASE5| Thicknet; used coaxial|
|10BASE2| Thinnet; used coaxial|
|10BASE-T| First type of Ethernet using twisted pair cabling|
|DIX Ethernet v2| Layer1 and Layer2 specs for original Ethernet, from DIX, typically called DIX v2|
|IEEE 802.3| Called MAC because of the name IEEE committee (Media Access Control); orig. Layer 1 and 2 specs, standardized using DIX v2 as a basis|
|IEEE 802.2| Called LLC becuase of the name of the IEEE committee (Logical Link Control); Layer 2 spec for headers common to multiple IEEE LAN specs.|
| IEEE 802.3u| IEEE standard for Fast Ethernet (100Mbps) over copper and optical ; typically called FastE|
| IEEE 802.3z| Gigabit Ethernet over optical cabling; typically called GigE|
| IEEE 802.3ab| Gigabit Ethernet over copper cabling|

Switches can dynamically detect the speed using couple methods:
1. Fast Link Pulses (FLP) of the autonegotiation process.
2. If autonegotiation is disabled on either end of the cable, the switch detects the speed anyway based on the incoming electrical signal

For a good reference for more information on the actual FLPs used by autonegotiation, refer to the Fast Ethernet web page of the University of New Hampshire Research Computing Center’s InterOperability Laboratory, at www.iol.unh.edu/services/testing/fe/training/.

Switches can only detect duplex settings through autonegotiation:
- If both ends have autonegotiation enabled, the duplex is negotiated
- If either device has autonegotiation disable, then devices w/o configured duplex will goto default setting.
  - Default duplex on Cisco switches is half duplex (HDX)

Full duplex (FDX) is only available when collisions cannot occur on attached cable.
- Collision-free links are only guaranteed when a shared hub is **NOT** in use.

### CSMA/CD
- The original Ethernet spec. expected collisions to occur.
- Media was shared, creating a literal electrical bus.
  - Any signal induced onto wire could collide with another induced signal from another device.
- When two or more Ethernet frames overlap on the transmission medium at the same instant in time, a collision occurs; the collision results in bit errors and lost frames.

CSMA/CD minimizes the number of collisions, but when they occur, CSMA/CD defines how the sending stations can recognize the collisions and retransmit the frame.

1. A device with a frame to send listens until the Ethernet is not busy (in other words, the device cannot sense a carrier signal on the Ethernet segment).

2. When the Ethernet is not busy, the sender begins sending the frame.

3. The sender listens to make sure that no collision occurred.

4. If there was a collision, all stations that sent a frame send a jamming signal to ensure that all stations recognize the collision.

5. After the jamming is complete, each sender of one of the original collided frames randomizes a timer and waits that long before resending. (Other stations that did not create the collision do not have to wait to send.)

6. After all timers expire, the original senders can begin again with Step 1.

### Collision Domains and Switch Buffering

_collision domain is_ a set of devices that can send frames that collide with frames sent by another device in that same set of devices.

By definition of the term, **Ethernet hubs**

-  Operate solely at Ethernet Layer 1

-  Repeat (regenerate) electrical signals to improve cabling distances

-  Forward signals received on a port out all other ports (no buffering)

NICs operating in *HDX* mode use *loopback circuitry* when transmitting a frame. This circuitry loops the transmitted frame back to the receive side of the NIC so that when the NIC receives a frame over the cable, the combined looped-back signal and received signal allows the NIC to notice that a collision has occurred.

**Ethernet switches** greatly reduce the number of possible collisions, both through frame buffering and through their more complete Layer 2 logic.

- Switches have the same cabling and signal regeneration benefits as hubs
- sometimes reducing or even eliminating collisions by buffering frames
- When switches receive multiple frames on different switch ports, they store the frames in memory buffers to prevent collisions.
- When a switch port connects through cable to a single other non-hub device, no collisions can possibly occur.
- Because collisions cannot occur, such segments can use *full-duplex* logic.

## Ethernet Layer 2: Framing and Addressing

**Frame:** refers to the bits and bytes that include the Layer 2 header and trailer, along with the data encapsulated by that header and trailer

**Packet:** most often used to describe the Layer 3 header and data, without a Layer 2 header or trailer.


|Preamble|SFD|Dest. Address|Src. Address|Length|DSAP|SSAP|Control|OUI|TYPE|Data|FCS|
|:------:|:-:|:-----------:|:----------:|:----:|:--:|:--:|:-----:|:-:|:--:|:--:|:-:|
|7B|1B|6B|6B|2B|1B|1B|1-2B|3B|2B|Variable|4B|

*802.3 Layer 2 Frame*

- **Preamble**: Provides synchronization and signal transitions to allow proper clocking of transmitted signal.

- **SFD (Start of Frame Delimiter)**: Provide byte-level synchronization and to mark a new incoming frame.

- **Dest./Src. Address:** 48-bit MAC addresses.

- **Length** Describes the length, in bytes, of data following the Length field, up to the Either trailer.  Allows receiver to predict the end of the received frame.

### Types of Ethernet Addresses
Unicast: Single LAN interface.  The I/G bit, the least significant bit in the most significant byte, is set to 0.
- IEEE intends for unicast addresses to be unique in the universe

Broadcast: All devices address.  Always hex value FFFFFFFFFFFF.

Multicast: Implies some subset of devices on the LAB.  By definition, the I/G bit is set to 1.
- Some devices may receive the frame, but discard it if not anticipating it.
  - Non-joined mcast PCs can Rx an mcast frame for many reasons:
    1. No switch support of mcast
    2. No IGMP snooping enabled on the VLAN.
    3. since there is a 32:1 overlap of IP Multicast addresses to Ethernet MAC addresses, any multicast address in the [224-239].0.0.x and [224-239].128.0.x ranges should NOT be considered.

### Ethernet Address Formats

**OUI:** Organizationally Unique Identifier (OUI).  The IEEE expects each manufacturer to use its OUI for the first 3 bytes of the MAC assigned to any Ethernet product created by that vendor.

**MAC Byte Significance**

|Most Significant Byte | | | | | Least Significant Byte |
|-|-|-|-|-|-|
| 1st Byte  |  2nd Byte  |  3rd Byte  |  4th Byte  | 5th Byte  |  6th Byte  |
|OUI | -> | -> | Vendor-Assigned | -> | -> |



**1st Byte Below**
| | | | | | | U/L Bit | I/G Bit |
|-|-|-|-|-|-|-|-|
|bit|bit|bit|bit|bit|bit|bit|bit| 
|Most significant Bit| | | | | | | Least significant Bit |

IEEE documentation lists Ethernet addresses with the most significant byte on the left. However, inside each byte, the leftmost bit is the most significant bit, and the rightmost bit is the least significant bit.
- Two most significant bits in an Ethernet address:
  - Image The Individual/Group (I/G) bit
    - Binary 0 means the address is unicast.  Binary 1 means the address is multicast or broadcast.
    - Wireshark filter for any broadcast or multicast. ```(eth.dst[0] & 1)```
    - Ethernet multicast addresses used by IP multicast implementations always start with 0x01005E. Hex 01 (the first byte of the address) converts to binary 00000001, with the least significant bit being 1, confirming the use of the I/G bit.
  - Image The Universal/Local (U/L) bit
    - Binary 0 means address was vendor assigned.  Binary 1 means address was administratively assigned, overriding the vendor-assigned address.

### Protocol Types and the 802.3 Length Field
**Type Field:** allows receiver of Ethernet frame to determine type of data.  Ex.  IP, IPX, AppleTalk, etc.

- DIX and the revised IEEE framing use the Type field, also called the Protocol Type field.
- The originally defined IEEE framing uses those same 2 bytes as a Length field
- Ethernet Type field values begin at 1536, and the length of the Data field in an IEEE frame is limited to decimal 1500 or less. That way, an Ethernet NIC can easily determine whether the frame follows the DIX or original IEEE format.
- The original IEEE frame used a 1-byte Protocol Type field (DSAP) for the 802.2 LLC standard type field.

### Switching and Bridging Logic

Switches learn MAC addresses, and the port to associate with them, by reading the source MAC address of received frames. 

| Type of Address | Switch Action |
| - | - |
| Known unicast | Forwards frame out the single interface associated with the dest. address.
| Unknown unicast | Flood frame out all interfaces, except the int on which the frame was Rx.| 
| Broadcast | Floods frame identically to unknown unicast|
| Multicast | Floods frame identically to unknown unicast, unless multicast optimizations are configured.|

```
Switch1# show mac-address-table dynamic
          Mac Address Table
------------------------------------------

Vlan    Mac Address       Type       Ports
----    -----------       ----       -----
   1    000f.2343.87cd    DYNAMIC    Fa0/13                                            
   1    0200.3333.3333    DYNAMIC    Fa0/3
   1    0200.4444.4444    DYNAMIC    Fa0/13
   ```
   ```
   switch4# show mac-address-table aging-time
Vlan    Aging Time
----    ----------
   1     300                                  
   ```

## Switching Internal Processing Methods

| Switching Methods | Description |
|-|-|
| Store-and-forward | The switch fully receives all bits in the frame (store) before forwarding the frame (forward).  This allows the switch to check the frame check sequence (FCS) before forwarding the frame, thus ensuring that errored frames are not forwarded.|
| Cut-through | The switch performs the address table lookup as soon as the Dest. Add. field in the header is received.  The first bits in the frame can be sent out the outbound port before the final bits in the incoming frame are received.  This does not allow the switch to discard frames that fails the FCS check, but the forwarding action is faster, resulting in lower latency. |
| Fragment-free | This performs like cut-through switching, but the switch waits for 64 bytes to be received before forwarding the first bytes of the outgoing frame.  According to Ethernet specifications, collisions should be detected during the first 64 bytes of the frame, so frames that are in error because of a collision will not be forwarded.|

## SPAN, RSPAN, and ERSPAN

_SPAN_ is Switch Port Analyzer
- SPAN sessions can be sourced from a port or ports, or from a VLAN.
- monitoring traffic for compliance reasons, for data collection purposes, or to support a particular application.
- destination port for a SPAN session can be on the local switch, as in SPAN operation

_RSPAN_ is Remote Switch Port Analyzer
- a specific VLAN must be configured across the entire switching path from the source port or VLAN to the RSPAN destination port.
  - This requires that the RSPAN VLAN be included in any trunks in that path, too.

_ERSPAN_ is Encapsulated Remote Switch Port Analyzer
- Encapped in GRE tunnel and sent to an IP
- IP Protocol number 47, which is GRE. 47 in HEX is 2F, so the capture filter for this is ```ip proto 0x2f```

### Core Concepts of SPAN, RSPAN, and ERSPAN

SPAN
- Source that consists of at least one port or at least one VLAN on a switch.
- On the same switch, you configure a destination port.

RSPAN
- you create the same source type—at least one port or at least one VLAN.
- destination for this session is the RSPAN VLAN, rather than a single port on the switch
- At the switch that contains an RSPAN destination port, the RSPAN VLAN data is delivered to the RSPAN port.

ERSPAN
- creates a generic routing encapsulation (GRE) tunnel for all captured traffic and allows it to be extended across Layer 3 domains
- operational enhancement brought to us by IOS-XE and can be found in current platforms like the ASR 1000, but keep in mind that ERSPAN is also supported by the Catalyst 6500, 7600, as well as the Nexus platforms.
- Viable monitoring sources include Fast Ethernet, Gigabit Ethernet, and Port-Channel interfaces.

Rules:
- Regardless of the type of SPAN we are running, a SPAN source port can be any type of port—a routed port, a physical switch port, an access port, a trunk port, an EtherChannel port (either one physical port or the entire port-channel interface), and so on.
- SPAN source VLAN, all active ports in that VLAN are monitored. A
- A port configured as a SPAN destination cannot be part of a SPAN source VLAN.

### SPAN Restrictions and Conditions
**RESTRICTIONS**
- When you configure a destination port, its original configuration is overwritten. If the SPAN configuration is removed, the original configuration on that port is restored.
- When you configure a destination port, the port is removed from any EtherChannel bundle if it were part of one. If it were a routed port, the SPAN destination configuration overrides the routed port configuration.
- Destination ports do not support port security, 802.1x authentication, or private VLANs. In general, SPAN/RSPAN and 802.1x are incompatible.
- Destination ports do not support any Layer 2 protocols, including CDP, Spanning Tree, VTP, DTP, and so on.

**CONDITIONS**
 - The source can be either one or more ports or a VLAN, but not a mix of these.
- Image Up to 64 SPAN destination ports can be configured on a switch.
- Image Switched or routed ports can be configured as SPAN source ports or SPAN destination ports.
- Image Be careful to avoid overloading the SPAN destination port. A 100-Mbps source port can easily overload a 10-Mbps destination port; it’s even easier to overload a 100-Mbps destination port when the source is a VLAN.
- Image Within a single SPAN session, you cannot deliver traffic to a destination port when it is sourced by a mix of SPAN, RSPAN, or ERSPAN source ports or VLANs. This restriction comes into play when you want to mirror traffic to both a local port on a switch (in SPAN) and a remote port on another switch (in RSPAN or ERSPAN mode).
- Image A SPAN destination port cannot be a source port, and a source port cannot be a destination port.
- Image Only one SPAN/RSPAN/ERSPAN session can send traffic to a single destination port.
-Image A SPAN destination port ceases to act as a normal switch port. That is, it passes only SPAN-related traffic.
- Image It’s possible to configure a trunk port as the source of a SPAN or RSPAN session. In this case, all VLANs on the trunk are monitored by default; the filter vlan command option can be configured to limit the VLANs being monitored in this situation.
- Image Traffic that is routed from another VLAN to a source VLAN cannot be monitored with SPAN. An easy way to understand this concept is that only traffic that enters or exits the switch in a source port or VLAN is forwarded in a SPAN session. In other words, if the traffic comes from another source within the switch (by routing from another VLAN, for example), that traffic isn’t forwarded through SPAN.

### SPAN Types of Traffic
PAN, RSPAN, and ERSPAN support three types of traffic: transmitted, received, and both. 
- By default, SPAN is enabled for traffic both entering and exiting the source port or VLAN.
- **For Receive (RX) SPAN**, the goal is to deliver all traffic received to the SPAN destination. As a result, each frame to be transported across a SPAN connection is copied and sent before any modification (for example, VACL or ACL filtering, QoS modification, or even ingress or egress policing).
- **For Transmit (TX) SPAN**, all relevant filtering or modification by ACLs, VACLs, QoS, or policing actions are taken before the switch forwards the traffic to the SPAN/RSPAN destination. As a result, not all transmit traffic necessarily makes it to a SPAN destination. Also, the frames that are delivered do not necessarily match the original frames exactly, depending on policies applied before they are forwarded to the SPAN destination.
- A special case applies to certain types of Layer 2 frames. SPAN/RSPAN **usually ignores CDP, spanning-tree BPDUs, VTP, DTP, and PAgP frames**. However, these traffic types can be forwarded along with the normal SPAN traffic if the encapsulation replicate command is configured.

### Basic SPAN Configuration
```
MDF-ROC1# configure terminal
MDF-ROC1(config)# monitor session 1 source interface fa0/12
MDF-ROC1(config)# monitor session 1 destination interface fa0/24
```
### Complex SPAN Configuration
Configure a switch to send the following traffic to interface fa0/24, preserving the encapsulation from the sources:

- Received on interface fa0/18
- Sent on interface fa0/9
- Sent and received on interface fa0/19 (which is a trunk)

We also filter (remove) VLANs 1, 2, 3, and 229 from the traffic coming from the fa0/19 trunk port.
```
MDF-ROC3# config term
MDF-ROC3(config)# monitor session 11 source interface fa0/18 rx
MDF-ROC3(config)# monitor session 11 source interface fa0/9 tx
MDF-ROC3(config)# monitor session 11 source interface fa0/19
MDF-ROC3(config)# monitor session 11 filter vlan 1 - 3 , 229
MDF-ROC3(config)# monitor session 11 destination interface fa0/24 encapsulation replicate
```
### RSPAN Configuration
we configure two switches, IDF-1 and IDF-2, to send traffic to RSPAN VLAN 199, which is delivered to port fa0/24 on switch MDF-SYR9 as follows:
- From IDF-1, all traffic received on VLANs 66–68
- From IDF-2, all traffic received on VLAN 9
- From IDF-2, all traffic sent and received on VLAN 11

Note that all three switches use a different session ID, which is permissible in RSPAN. The only limitation on session numbering is that the session number must be 1 to 66.
```
IDF-1# config term
IDF-1(config)# vlan 199
IDF-1(config-vlan)# remote span
IDF-1(config-vlan)# exit
IDF-1(config)# monitor session 3 source vlan 66 – 68 rx
IDF-1(config)# monitor session 3 destination remote vlan 199
```
```
IDF-2# config term
IDF-2(config)# vlan 199
IDF-2(config-vlan)# remote span
IDF-2(config-vlan)# exit
IDF-2(config)# monitor session 23 source vlan 9 rx
IDF-2(config)# monitor session 23 source vlan 11
IDF-2(config)# monitor session 23 destination remote vlan 199
```
```
MDF-SYR9# config term
MDF-SYR9(config)# vlan 199
MDF-SYR9(config-vlan)# remote span
MDF-SYR9(config-vlan)# exit
MDF-SYR9(config)# monitor session 63 source remote vlan 199
MDF-SYR9(config)# monitor session 63 destination interface fa0/24
MDF-SYR9(config)# end
```
### ERSPAN Configuration

Configure ASR 1002 to capture received traffic and send to it to Catalyst 6509 Gig2/2/1. This traffic will simply be captured, encapsulated in GRE by ASR 1002 natively, and routed over to the Catalyst 6509. A sniffing station on the 6500 attached to GE2/2/1 will see the complete Ethernet frame (L2 to L7) information.
```
ASR1002(config)# monitor session 1 type erspan-source
ASR1002(config-mon-erspan-src)# source interface gig0/1/0 rx
ASR1002(config-mon-erspan-src)# no shutdown
ASR1002(config-mon-erspan-src)# destination
ASR1002(config-mon-erspan-src-dst)# erspan-id 101
ASR1002(config-mon-erspan-src-dst)# ip address 10.1.1.1
ASR1002(config-mon-erspan-src-dst)# origin ip address 172.16.1.1
```
```
!Now for the configuration of the Catalyst 6500
SW6509(config)# monitor session 2 type erspan-destination
SW6509(config-mon-erspan-dst)# destination interface gigabitEthernet2/2/1
SW6509(config-mon-erspan-dst)# no shutdown
SW6509(config-mon-erspan-dst)# source
SW6509(config-mon-erspan-dst-src)# erspan-id 101
SW6509(config-mon-erspan-dst-src)# ip address 10.1.1.1
```
```
ASR1002# show monitor session 1
Session 1
---------
Type                   : ERSPAN Source Session
Status                 : Admin Enabled
Source Ports           :
   RX Only           : Gi0/1/0
Destination IP Address : 10.1.1.1
MTU                   : 1464
Destination ERSPAN ID : 101
Origin IP Address    : 172.16.1.1
```

## Virtual Switch System (VSS)
* This feature exists in **Cisco Catalyst 6500 and 4500 Series** switches running IOS-XE
* Simplifies the network by reducing the number of network elements, and this eliminates or masks the complexity of managing redundant switches and links
* simplifies the network by reducing the number of network elements
* eliminates or masks the complexity of managing redundant switches and links
* VSS manages the redundant links in such a fashion that they will be seen by external devices as a single Port-channel.
* simplifies network configuration and operation by reducing the total number of Layer 3 routing neighbors
* simultaneously providing a loop-free Layer 2 topology.

### Virtual Switching System
* Fundamental reason for employing a VSS is to logically combine a pair of switches into a single network element,
* Switch in the access layer connects to both switches of the VSS using one logical port channel because the VSS is perceived as being a single switch to external devices
* Switches aren't independent, though physically different.
  * must be understood that from an operation and control plane level, they are acting as one unit.
  * enables a loop-free Layer 2 network topology.
* simplifies the Layer 3 network topology by reducing the number of routing peers in the network, thus extending network simplification into the data plane itself.

### VSS Active and VSS Standby Switch
* Each individual switch will first contend for specific operational roles inside the VSS process itself and then behave according to those roles.
* Each time that we create or restart a VSS, the peer switches will negotiate their roles. 
* one device will become the VSS active switch, and the other will become the VSS standby.
* **VSS active switch** controls the VSS, running the Layer 2 and Layer 3 control protocols for the switching modules on both switches.
  * Provides management functions for the VSS, such as module online insertion and removal (OIR) and the console interface.
* **VSS standby switch** sends all control traffic to the VSS active switch for processing.

### Virtual Switch Link (VSL)
Special link that carries control and data traffic between the two switches of a VSS
* The VSL is typically implemented as an EtherChannel
  * Can support up to eight links incorporated into the bundle
* Optimized to provide _control and management plane traffic_ higher priority than data traffic in an effort to ensure that control and management messages are never discarded.
* Data traffic is load balanced among the VSL links by either the default or the configured EtherChannel load-balancing algorithm.

### Multichassis EtherChannel (MEC)
Multichassis EtherChannel (MEC), which is an EtherChannel whose member ports can be distributed across the member switches in a VSS.
* A VSS can support a maximum of **256 EtherChannels**. This limit applies to the total number of regular EtherChannels and MECs.
* A special kind of Port-channel that can exist not between two physical devices, but between multiple chassis.
* Layer 2 protocols operate on the EtherChannel as a single logical entity. 
  * This extends to protocols like Spanning Tree Protocol, which would normally serve to block redundant links between devices in an effort to prevent the formation of switching loops.
* Extends hardware/device failover not capable in normal EtherChannel.
* Non-VSS switches connected to a VSS view the MEC as a standard EtherChannel
  * Non-VSS switches can connect in a dual-homed manner
* Traffic traversing the MEC can be load balanced locally within a VSS member switch much like that of standard EtherChannels.
* Cisco MEC supports dynamic EtherChannel protocols:
  * industry-standard Link Aggregation Control Protocol (LACP)
  * the Cisco-proprietary Port Aggregation Protocol (PAgP)
  * static EtherChannel configuration.

### Basic VSS Configuration
1. create the same *virtual switch domain* on both sides of the VSS (between 1 and 255).
2. configure one switch to be switch number 1 and the other switch to be switch number 2.

```
SW1(config)# switch virtual domain 10
Domain ID 10 config will take effect only
after the exec command 'switch convert mode virtual' is issued
SW1(config-vs-domain)# switch 1
SW1(config-vs-domain)# exit

SW2(config)# switch virtual domain 10
Domain ID 10 config will take effect only
after the exec command 'switch convert mode virtual' is issued
SW2(config-vs-domain)# switch 2
SW2(config-vs-domain)# exit
```
3. Create the VSL, which requires a unique port channel on each switch.

**Note:** If the VSS standby switch VSL port channel number has been configured previously for another use, the VSS will come up in route processor redundancy mode. To avoid this situation, check that both port channel numbers are available on both of the peer switches.

```
SW1(config)# int port-channel 5
SW1(config-if)# switchport
SW1(config-if)# switch virtual link 1
SW1(config-if)# no shut
SW1(config-if)# exit
*Jan 24 05:19:57.092: %SPANTREE-6-PORTDEL_ALL_VLANS: Port-channel5 deleted from all Vlans

SW2(config)# int port-channel 10
SW2(config-if)# switchport
SW2(config-if)# switch virtual link 2
SW2(config-if)# no shut
SW2(config-if)# exit
SW2(config)#
*Jan 24 05:14:17.273: %SPANTREE-6-PORTDEL_ALL_VLANS: Port-channel10 deleted from all Vlans
```
4. Add VSL physical member ports to the appropriate Port-channel.

**Note:** After the interfaces are put into a VSL Port-channel with the channel-group command, the interfaces go into “notconnect” status. Interface status will show “up,” but the line protocol will be “down.” The interface will be in up/down (not connect) status until the switch is rebooted.

```
SW1(config)# int range gig7/3 - 4
SW1(config-if-range)# switchport mode trunk
SW1(config-if-range)# channel-group 5 mode on
WARNING: Interface GigabitEthernet7/3 placed in restricted config mode. All extraneous configs removed!
WARNING: Interface GigabitEthernet7/4 placed in restricted config mode. All extraneous configs removed!
SW1(config-if-range)# exit

SW2(config)# int range gig4/45 - 46
SW2(config-if-range)# switchport mode trunk
SW2(config-if-range)# channel-group 10 mode on
WARNING: Interface GigabitEthernet4/45 placed in restricted config mode. All extraneous configs removed!
WARNING: Interface GigabitEthernet4/46 placed in restricted config mode. All extraneous configs removed!
SW2(config-if-range)# exit
```
5. Complete the switch conversion process by implementing the switch convert mode virtual command on Switch 1

```
SW1# switch convert mode virtual

This command will convert all interface names
to naming convention "interface-type switch-number/slot/port",
save the running config to startup-config and

reload the switch.
Do you want to proceed? [yes/no]: yes                                                                                  
Converting interface names
Building configuration...
Compressed configuration from 6551 bytes to 2893 bytes[OK]
Saving converted configuration to bootflash: ...
Destination filename [startup-config.converted_vs-20130124-062921]?
Please stand by while rebooting the system...
Restarting system.

Rommon (G) Signature verification PASSED
Rommon (P) Signature verification PASSED
FPGA   (P) Signature verification PASSED

Similarly you need to enter the "switch convert mode virtual" command on Switch 2
  for converting to Virtual Switch Mode.
```
```
SW2# switch convert mode virtual

This command will convert all interface names
to naming convention "interface-type switch-number/slot/port",
save the running config to startup-config and
reload the switch.
Do you want to proceed? [yes/no]: yes                                                                                  
Converting interface names
Building configuration...
Compressed configuration from 6027 bytes to 2774 bytes[OK]
Saving converted configuration to bootflash: ...
Destination filename [startup-config.converted_vs-20130124-052526]?
Please stand by while rebooting the system...
Restarting system.

Rommon (G) Signature verification PASSED
Rommon (P) Signature verification PASSED
FPGA   (P) Signature verification PASSED



************************************************************
*                                                          *
* Welcome to Rom Monitor for   WS-X45-SUP7-E System.       *
* Copyright (c) 2008-2012 by Cisco Systems, Inc.           *
* All rights reserved.                                     *
*                                                          *
************************************************************
```

After confirmation is completed on each switch, the running configuration will be saved as the startup configuration and the switch will reboot. After the reboot, the switch will be in virtual switch mode.

### VSS Verification Procedures

```# show switch virtual``` will show *switch domain number*, and *the switch number* and *role* for each of the switches

```
SW1# sh switch virtual

Executing the command on VSS member switch role = VSS Active, id = 1

Switch mode                  : Virtual Switch
Virtual switch domain number : 10
Local switch number          : 1
Local switch operational role: Virtual Switch Active
Peer switch number           : 2
Peer switch operational role : Virtual Switch Standby

Executing the command on VSS member switch role = VSS Standby, id = 2

Switch mode                  : Virtual Switch
Virtual switch domain number : 10
Local switch number          : 2
Local switch operational role: Virtual Switch Standby
Peer switch number           : 1
Peer switch operational role : Virtual Switch Active
```
One of the most important operational requirements of a VSS is to ensure that one switch in the cluster is the active switch and the other is the standby.

Console of the Standby Switch:
```
SW2-standby>
Standby console disabled
```

```# show switch virtual role``` shows many other configuration variables for each switch in the Domain.

```
SW1# sh switch virtual role
Executing the command on VSS member switch role = VSS Active, id = 1
RRP information for Instance 1
--------------------------------------------------------------------
Valid Flags   Peer     Preferred Reserved
               Count     Peer       Peer
--------------------------------------------------------------------
TRUE   V       1           1         1
Switch Switch Status Preempt       Priority Role     Local   Remote
       Number         Oper(Conf)   Oper(Conf)         SID     SID
--------------------------------------------------------------------
LOCAL   1     UP     FALSE(N )     100(100) ACTIVE   0       0
REMOTE  2     UP     FALSE(N )     100(100) STANDBY 6834   6152

Peer 0 represents the local switch

Flags : V - Valid
In dual-active recovery mode: No

Executing the command on VSS member switch role = VSS Standby, id = 2

RRP information for Instance 2

--------------------------------------------------------------------
Valid Flags   Peer     Preferred Reserved
               Count     Peer       Peer

--------------------------------------------------------------------
TRUE    V       1           1         1

Switch Switch Status Preempt       Priority Role     Local   Remote
       Number         Oper(Conf)   Oper(Conf)         SID     SID
--------------------------------------------------------------------
LOCAL   2     UP     FALSE(N )     100(100) STANDBY  0       0
REMOTE  1     UP     FALSE(N )     100(100) ACTIVE   6152   6834

Peer 0 represents the local switch

Flags : V - Valid
In dual-active recovery mode: No
```

```# show switch virtual link``` shows information on the VSL

```
SW1# sh switch virtual link

Executing the command on VSS member switch role = VSS Active, id = 1

VSL Status : UP
VSL Uptime : 3 minutes
VSL Control Link : Gi1/7/4

Executing the command on VSS member switch role = VSS Standby, id = 2

VSL Status : UP
VSL Uptime : 3 minutes
VSL Control Link : Gi2/4/45
```

```# show switch virtual link port-channel``` verify information about the VSL port channel configuration

```
SW1# sh switch virtual link port-channel

Executing the command on VSS member switch role = VSS Active, id = 1

Flags: D - down       P - bundled in port-channel
       I - stand-alone s - suspended
       H - Hot-standby (LACP only)
       R - Layer3     S - Layer2
       U - in use     N - not in use, no aggregation
       f - failed to allocate aggregator

       M - not in use, no aggregation due to minimum links not met
       m - not in use, port not aggregated due to minimum links not met
       u - unsuitable for bundling
       d - default port

       w - waiting to be aggregated

Group Port-channel Protocol   Ports
------+-------------+-----------+-------------------
5     Po5(SU)          -       Gi1/7/3(P) Gi1/7/4(P)
10    Po10(SU)         -       Gi2/4/45(P) Gi2/4/46(P)

Executing the command on VSS member switch role = VSS Standby, id = 2

Flags: D - down       P - bundled in port-channel
       I - stand-alone s - suspended
       H - Hot-standby (LACP only)
       R - Layer3     S - Layer2
       U - in use     N - not in use, no aggregation
       f - failed to allocate aggregator

       M - not in use, no aggregation due to minimum links not met
       m - not in use, port not aggregated due to minimum links not met
       u - unsuitable for bundling
       d - default port

       w - waiting to be aggregated

Group Port-channel Protocol   Ports
------+-------------+-----------+-------------------
5     Po5(SU)          -       Gi1/7/3(P) Gi1/7/4(P)
10    Po10(SU)         -       Gi2/4/45(P) Gi2/4/46(P)

SW1#
```

## IOS-XE
* IOS was monolithic.  Not able to keep up with software demands.
  * Evolved in NX-OS, IOS-XR, and IOS-XE.
* IOS-XE was designed for routers, switches, and appliances.
* seamlessly integrates a generic approach to network management into every function, borrowing heavily from the equally reliable POSIX operating system
* all the advantages of traditional IOS and its unparalleled history for delivering functionality for business-critical applications.
  * while retaining the same look and feel of IOS, but doing it while ensuring enhanced “future-proof” functionality.
* IOS-XE runs a modern Linux operating system that employs a single daemon.
  * running IOS and these other applications as separate processes, it becomes apparent that we can now leverage symmetrical multiprocessing.
  * should one of these isolated processes fail, it will not affect the kernel. 
  * garner the benefits of load balancing across multiple-core CPUs by binding processes to different cores.
  * operational environment where it is possible to support multi-threading and multicore CPUs
* IOS-XE separates the control plane from the forwarding plane, ensures a level of management and control that could not possibly exist in the context of the traditional monolithic IOS.
* application designers will have the ability to build drivers for new data plane ASICs and have them interoperate with sets of standard APIs.
  * these APIs that will then create control plane and data plane processing separation.

As a result of the modular architecture that we have attributed to IOS-XE, we see both a logical and a physical isolation of the control and data planes themselves. This at first might seem to be a subtle difference, but in fact, this has far-reaching benefits. Now we have physical separation of the three planes through modular blades that are installed into the chassis, each having dedicated hardware resources. IOS-XE also maintains logical separation as an abstraction layer. We get even more capabilities to reduce failure domains within the routing system, as well as the capability to isolate operation loads between planes. An example of this would be heavy stress caused by forwarding massive amounts of traffic in the data plane, which would have no impact on the control plane running on the same chassis. This is made possible by the fact that IOS-XE runs a separate driver instance for each bay or blade slot in the chassis; therefore, one drive failing will have no impact on the other bays or the chassis as a whole. The outcome is that all the other processes will continue to forward traffic. In addition to this, we can actually patch individual drivers without bringing down the entire chassis.

This separation is achieved through the **Forwarding and Feature Manager (FFM)** and the **Forwarding Engine Driver (FED)**.

The FFM provides a set of APIs used to manage the control plane processes. The resulting outcome is that the FFM programs the data plane through the FED and maintains all forwarding states for the system. It is the FED that allows the drivers to affect the data plane, and it is provided by the platform.



# IPv6
## Modified EUI-64 Addressing
- Host assigns itself an address.
- MAC address is converted into a 64-bit identifier, called a Modified EUI-64.  Then appended to a 64 bit network prefix learned by other means.
- Eliminates need for static or DHCP.

### How it's made
1. Take 48-bit MAC address
2. Split into two 24-bit parts.
- first being the OUI (organizationally unique identifier) 
- the other being NIC-specific
- 64 - 48 bits = 16 bits remaining
3. The 16-bit value of FFFE is then inserted between these two 24-bit groups

*IEEE has chosen FFFE as a reserved value that can only appear in EUI-64 generated from the EUI-48 MAC address.*

4. Finally, the seventh bit from the left, or the universal/local (U/L) bit, needs to be inverted. 
- Determines if officially assigned or locally generated
- If it is 0, the address is locally administered
- If it is 1, the address is globally unique
- In the original OUI portion, the globally unique addresses assigned by the IEEE have their U/L bit always set to 0, whereas locally created addresses have it set to 1

*The meaning of the U/L bit in the Modified EUI-64 is inverted when compared to the meaning assigned by IEEE. This is the reason for calling this address the Modified EUI-64. Therefore, when the bit is inverted, it maintains its original value.*

### Sample Conversion
 OUI | NIC 
 -- | --
 00 21 2F | B5 6E 10
 0000 0000 | <- original bits
 0000 0010 | <- inverted 7th bit
 02 21 2F  | B5 6E 10

**Resulting Modified EUI-64 address**

*Modified OUI* + *FFFE* + *NIC*

*24-bits* + *16-bits* + *24-bits*

 **221:2FFF:FEB5:6E10**

 ### Implementing Modified EUI-64
View Burned in address of interface
 ```
 R1# show interface FastEthernet0/1 | include bia

  Hardware is Gt96k FE, address is 0017.942f.10f1 (bia 0017.942f.10f1)
  ```

Enable EUI-64 on interface
 ```
R1(config)# interface FastEthernet0/1
R1(config-if)# ipv6 address 2001::/64 eui-64
```
View the EUI-64 address
```
R1# show ipv6 interface FastEthernet0/1 | include EUI

    2001::217:94FF:FE2F:10F1, subnet is 2001::/64 [EUI/TEN]
```

## IPv6 Neighbor Discovery

Neighbor Discovery protocol is in fact an umbrella term for many interrelated sub-protocols and mechanisms whose main responsibilities include the following:

- Resolution of IPv6 addresses into MAC addresses of neighboring hosts
- Duplicate address detection
- Router discovery
- Stateless address auto-configuration
- Host redirection

_The common denominator for all these operations is the ICMPv6 protocol_

1. ICMPv6 performs legacy ICMP tasks (ping, TTL expiration, packet too big, unknown dest, communication prohibited, etc.)
2. ICMPv6 has also taken on the role of Address Resolution Protocol (ARP) as well as some other protocols.

_Neighbor Discovery functions added to ICMPv6 are related to facilitating communication between directly connected neighbors_

1. **Router Advertisement**
- Sent by routers out each IPv6-enabled interface
- Sent periodically, though infrequently
- Set immediately as a response to a **Router Solicitation** message
- Informs attached hosts of router presence
- May contain vital information allowing hosts to automatically configure their IPv6 stack: the IPv6 prefix of the particular network, the first-hop MTU, the router’s MAC address (allowing hosts to learn about the router’s MAC address right away), and even the list of DNS domains and DNS servers
- Using the information in a received Router Advertisement, a host can automatically configure its IPv6 stack and obtain IPv6 connectivity without the need for any dedicated DHCP service on a network

2. **Router Solicitation**
- Sent by node when interface is enabled
- Used to get **Router Advertisements** from local link routers.  Beats waiting for scheduled advertisements.

3. **Redirect:** Redirect messages are used by routers to tell hosts that a better on-link router exists for a given destination address.

4. **Neighbor Solicitation** (three main purposes)
- Discover the link layer address of a neighbor as part of the MAC address resolution process. This process replaces the use of ARP requests and replies in IPv4
- The second purpose is to determine the reachability of a neighbor
- The last purpose is to detect the presence of duplicate IPv6 addresses.

5. **Neighbor Advertisement**
- Sent in response to Neighbor Solicitations
- Sent by a neighbor to announce a change in its link layer address
- Upon receipt of a Neighbor Advertisement, a node will update its neighbor cache, which contains mappings between IPv6 and link layer addresses of neighbors.

6. **Host auto-configuration**
- When host is first connected it must acquire info necessary for host to speak to local link and the entire network (link-local and global addresses)
- Link-local IPv6 can be automatic generation, or set manually
- To prevent duplicate addresses the **Duplicate Address Detection** procedure is invoked.

7. **Duplicate Address Detection**
- Auto-configured host sends **Neighbor Solicitation** to all node interfaces on same link using _solicited node multicast_
- Source of the multicast packet is unspecified address (::)
- Body of packet contains nodes complete link-local address, which is checked for dupes.
- If no response in given timer period, then node going through auto-configure can keep the address.
- Otherwise, the node with duplicate address will respond with a **Neighbor Advertisement**.  This advertisement is sent to the _all-nodes multicast address_ FF02::1.  Message includes it's link-local address and a flag to indicate if sending node is a router
- All nodes on the same link will therefore be forced to update their neighbor caches if their entries are not up to date.
- A duplicate address on the local link may require the manual configuration of the new host interface link-local address

8. **Router Discovery**
- Performed by host after it has unique link-local address.
- Host uses this packet to find routers on local link and also prefix lists used on the local link.
- **Router Solicitation** to all routers on the local link using the _all-router multicast_ address FF02::2
- In doing so, the host provides all such routers with its newly created local link address and its corresponding link layer address, so that all routers may update their records. All routers then respond in turn with a Router Advertisement
- The **Router Advertisement** info received is used by host in auto-configuration:
  - A router's link layer address
  - A router’s lifetime (that is, how long a host is able to keep using this router until subsequent advertisements update this value)
  - Flags used to determine the process by which the host’s global unicast address is created
  - Periodic timer values used in the Address Resolution and Neighbor Unreachability Detection procedures
  - Global prefixes that should be cached in the host’s prefix list
- At this point, the host can use its own MAC address to create a Modified EUI-64 interface identifier and append it to the prefixes acquired from the Router Advertisement message, generating a set of unique global IPv6 addresses, one for each global prefix
- In the end, the host has all information to achieve full IPv6 connectivity: its own global address and the address of its gateway. This process is called stateless address auto-configuration, or SLAAC.

## Implementing auto-configuration
Let’s look at how to configure IPv6 auto-configuration between R1 and R2. We will configure R2 such that it obtains its IPv6 address from R1. First, we need to make our configuration on R1

```
R1# conf t

Enter configuration commands, one per line.  End with CNTL/Z.

R1(config)# ipv6 unicast-routing

R1(config)# int FastEthernet0/0

R1(config-if)# ipv6 address FE80::1 link-local

R1(config-if)# ipv6 address 2001:12::1/64

R1(config-if)# no shut
```
Now we need to tell R2 to obtain its interface IPv6 address via auto-configuration

```
R2# conf t

Enter configuration commands, one per line.  End with CNTL/Z.

R2(config)# ipv6 unicast-routing

R2(config)# interface FastEthernet0/0

R2(config-if)# ipv6 enable

R2(config-if)# ipv6 address autoconfig

R2(config-if)# no shut

R2(config-if)# end
```

_Sidenote: telling SLAAC to install default route looks like_
```
R2(config-if)# ipv6 address autoconfig default
```
Before we do anything else, we need to check whether R2 sees any routers on the Ethernet segment connected to R1

```
R2# show ipv6 routers

Router FE80::1 on FastEthernet0/0, last update 1 min

  Hops 64, Lifetime 1800 sec, AddrFlag=0, OtherFlag=0, MTU=1500

  HomeAgentFlag=0, Preference=Medium

  Reachable time 0 msec, Retransmit time 0 msec

  Prefix 2001:12::/64 onlink autoconfig

    Valid lifetime 2592000, preferred lifetime 604800
```

Note that R2 sees R1 as the router on this segment (ID FE80::1), and that the prefix 2001:12::/64 will be issued as the auto-configurable network address. Lastly, observe that the lifetime of this auto-configuration will be 30 days. Now let’s look at the address that was auto-configured on R2

```
R2# show ipv6 interface FastEthernet0/0

FastEthernet0/0 is up, line protocol is up

  IPv6 is enabled, link-local address is FE80::20F:8FFF:FE4A:1060

  No Virtual link-local address(es):

  Global unicast address(es):

    2001:12::20F:8FFF:FE4A:1060, subnet is 2001:12::/64 [EUI/CAL/PRE]

      valid lifetime 2591899 preferred lifetime 604699

  Joined group address(es):

    FF02::1

    FF02::2

    FF02::1:FF4A:1060

  MTU is 1500 bytes

  ICMP error messages limited to one every 100 milliseconds

  ICMP redirects are enabled

  ICMP unreachables are sent

  ND DAD is enabled, number of DAD attempts: 1

  ND reachable time is 30000 milliseconds

  ND advertised reachable time is 0 milliseconds

  ND advertised retransmit interval is 0 milliseconds

  ND router advertisements are sent every 200 seconds

  ND router advertisements live for 1800 seconds

  ND advertised default router preference is Medium

  Hosts use stateless autoconfig for addresses.
```

Observe that R2 has created a host portion for the IPv6 address based on the Modified EUI-64 scheme for both the global and link local addresses. Also notice that R2 joined three multicast groups: FF02::1 as an IPv6-enabled node, FF02::2 as an IPv6-enabled router, and FF02::1:FF4A:1060 as a solicited-node multicast address that corresponds to both the link-local address and the global unicast address.

## Stateless address auto-configuration (SLAAC) Proccess

1. A host sends a Router Solicitation (RS) message.

2. A router with IPv6 unicast routing enabled will reply with a Router Advertisement (RA) message.

3. The host takes the IPv6 prefix from the Router Advertisement message and combines it with the 64-bit Modified EUI-64 address to create a global unicast address.

4. The host also uses the source IPv6 address of the Router Advertisement message as its default gateway. This address would be a link-local address.

5. Duplicate Address Detection is performed by IPv6 clients to ensure the uniqueness of the new IPv6 address.

## IPv6 Layer 2 Multicast
- L3 mcast packet encapsulated into L2 Ethernet fram.
- For IPv6, The group MAC address is computed using the prefix of 33-33 in hexadecimal, concatenated with the lowermost four bytes of the multicast IPv6 address.

```
show ipv6 interface FastEthernet0/0 | include FF


  IPv6 is enabled, link-local address is FE80::200:11FF:FE11:1111
```

Multicast Address | MAC Address | Purpose
--- | --- | ---
FF02::1 | 33-33-00-00-00-01 | All nodes
FF02::2 | 33-33-00-00-00-02 | All routers
FF02::1:FF00:1 | 33-33-FF-00-00-01 | SNM based on the Global unicast IPv6 address
FF02::1:FF11:1111 | 33-33-FF-11-11-11 | SNM based on the Link Local IPv6 address

_"SNM" stands for Solicited Node Multicast_


# VLANS
- Normal-range: VLANs 1–1005
- Extended: 1006-4095
 - 802.1q was first to support 12-bit VLAN ID field for Extended range (2^12 = 4096).
 - Following that, Cisco changed ISL to use 12 of it's reserved 15 bits to support Extended Range VLANs.
- VLANs have an operational state: active (default), or suspended.
 - A suspended VLAN is hibernating. It exists, but my not pass traffic.  Access ports in this VLAN drop all frames.
 

# Private VLAN
- There is a Primary VLAN, and the private VLANs are secondary to it.
- Each primary VLAN can only have one Isolated VLAN. Having multiple makes no sense.
- There may be many community VLANs per primary.
 - Ports in the same community may communicate.
- Isolated ports may not speak to anything accept a Promiscuous port.
- Promiscous ports are used for link to the gateway.
- A device on promiscious port can communicate with all secondary VLANs associated with primary and vice versa.
- If a frame from a secondary VLAN is passed through a trunk port, it's tagged using the corresponding primary VLAN.

# DTP (Dynamic Trunking Protocol)
- Cisco technology
- It is used for auto trunk negotiation.
- Depending on the Catalyst generation, the switchport defaults differ.
 - Catalyst 2950 and 3550 models default ```switchport mode dynamic desirable```
 - Catalyst models, such as 2960, 3560 or 3750, default ```switchport mode dynamic auto```
- If one of the sides is desirable, then a trunk is negotiated.
- When negotiated, then trunk interface bounces.
- If a switch supports 802.1q and ISL (Inter-Switch Link) encapsulation, then ISL is used.
- A trunks encapsulation can be changed on the fly without bouncing the port.
- CDP can be used to detect native VLAN mismatch.
- Switches will only negotiate a link if the VTP domain name matches, or one switch has no VTP domain name configured (NULL).
 - This prevents accidental overwrites of VLANs.  You wouldn't want switches in different VTP domains to auto trunk to eachother.
 
 ```
 May 24 10:15:29.796: %DTP-5-DOMAINMISMATCH: Unable to perform trunk negotiation on port Et0/1 because of VTP domain mismatch.
```

 - Routers do not support DTP, so you must manually configure them to support trunking.
 - The TOS/TAS/TNS stand for Trunk Operating/Administrative/Negotiation Status! The TOT/TAT/TNT stand for Trunk Operating/Administrative/Negotiation Type
 
 # ISL and 802.1q Trunking
 - ISL adds a new 26-byte header, plus a new trailer (to allow for the new FCS value), encapsulating the entire original frame.
 - 802.1Q inserts a 4-byte header, called a tag, into the original frame (right after the Source Address field). The original frame’s addresses are left intact.
 - You can configure 802.1Q native VLANs under a subinterface or under the physical interface on a router. Use the ```encapsulation dot1q vlan-id native``` subcommand.
 - Alternately, if not configured on a subinterface, the router assumes that the native VLAN is associated with the physical interface.
 
 # Jumbo Frames
 - ```system mtu 1504``` Applies to 100Mbps interfaces
 - ```system mtu jumbo 1504``` Applies to 1Gbps and 10Gbps interfaces
 
 # VLAN Tagging on WAN
 - Today, several emerging alternatives exist for the passage of VLAN traffic across a WAN, including 802.1Q-in-Q, its standardized version 802.1ad called Provider Bridges, another standard 802.1ah called Provider Backbone Bridges, Layer2 Tunneling Protocol (L2TPv3), Ethernet over MPLS (EoMPLS), and VLAN Private LAN Services (VPLS). While these topics are more applicable to the CCIE Service Provider certification, you should at least know the concept of 802.1 Q-in-Q tunneling.
 - Also known as Q-in-Q on Catalyst switches, 802.1Q-in-Q allows an SP to preserve 802.1Q VLAN tags across a WAN service. By doing so, VLANs actually span multiple geographically dispersed sites.
  - The ingress SP switch takes the 802.1Q frame, and then tags each frame entering the interface with an additional 802.1Q header, called the S-tag (the original customer tags are called C-tags and are not modified nor processed).
- On Catalyst switches, the Q-in-Q is supported on 3550 and higher platforms

# VTP
- In any VTP version, messages Transmitted and accepted on trunk ports only.
- Access ports do not send nor accept VTP messages.

# VTPv1 and VTPv2
- VTPv1 is default active version on enterprise IOS-based switches.
- VTPv1 and VTPv2 only support Normal range VLANs.
 - Can be configured in vlan database or config mode.  Stored in vlan.dat in Flash.
- Avoid using VLANs 1006–1024 for compatibility with CatOS-based switches.
- Defaults to VTPv1. No password, no domain name.  Doesn't send updates.

## Migrating from VTPv1/2 to 3
- Cannot change a Client's version without first going to Transparent mode.

```
IOU2(config)#vtp version 2
Cannot modify version in VTP client mode unless the system is in VTP version 3
IOU2(config)#vtp mode transparent 
Setting device to VTP Transparent mode for VLANS.
IOU2(config)#vtp version 2        
IOU2(config)#
```


# VTPv3
- VTPv3 introduces ```vtp mode off``` to disable globally, or per-interface basis with ```no vtp``` on the interface.
- VTPv3 if in any mode (server, client, transparent, or off) the normal and extended-range VLANs are stored in vlan.dat.  If tranparent, or off mode, then VLANs are also present in the running-config.
- VTPv3 supports Extended range VLANs (1006-4095).
- Supports many Servers, but only one Primary server.
 - Promote a Server to Primary status with an EXEC ```vtp primary```.
 - The state of Primary Server is not stored in config.  It's a runtime state.
- VTPv3 no longer lets you reset the configuraton revision number back to 0 by setting the switch to Transparent mode and back.
  - In VTPv3, if you want to reset the config rev. # you must change the domain name, or configure a password.
- Cooperation between VTPv3 and VTPv1-only switches is not supported.
- TPv3 switch detects an older switch running VTPv1 or VTPv2 on its port, it will revert to VTPv2 operation on that port, forcing the older switch to operate in VTPv2 mode.

# PPPoE
- PPPoE virtualizes Ethernet into multiple point-to-point sessions between client hosts and an access concentrator, turning the broadcast Ethernet into a point-to-multipoint environment.
- The PPPoE client feature permits a Cisco IOS router, rather than an endpoint host, to serve as the client in a network.
  - Lets entire LANs connect to Internet, because router is inline and can NAT.
- In a DSL environment, PPP interface IP addresses are derived from an upstream DHCP server using IP Configuration Protocol (IPCP), a subprotocol of PPP. Therefore, IP address negotiation must be enabled on the router’s dialer interface. This is done using the ip address negotiated command in the dialer interface configuration.
- MTU should be 1492.  PPPoE introduces an 8-byte overhead (2 bytes for the PPP header and 6 bytes for PPPoE).
- TCP sessions should negotiate Maximum Segment Size down to 1452 bytes, allowing for 40 bytes in TCP and IP headers and 8 bytes in the PPPoE, totaling 1500 bytes that must fit into an ordinary Ethernet frame.
- The key difference between ISDN BRI configuration and PPPoE is the pppoe-client dial-pool-number command.
