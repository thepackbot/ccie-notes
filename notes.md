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
* Can set speed with subinterface command
```
Switch(config)# interface fastethernet 5/4
 Switch(config-if)# speed speed {auto | 10 | 100 | 1000}
 ```
* Can set duplex with subinterface command

*Note: you cannot set duplex until the speed is manually configured*
```
Switch(config)# interface fastethernet 5/4
 Switch(config-if)# duplex duplex {auto | half | full}
 ```



Switches can dynamically detect the speed using couple methods:
1. Fast Link Pulses (FLP) of the autonegotiation process.
2. If autonegotiation is disabled on either end of the cable, the switch detects the speed anyway based on the incoming electrical signal

Switches can only detect duplex settings through autonegotiation:
- If both ends have autonegotiation enabled, the duplex is negotiated
- If either device has autonegotiation disable, then devices w/o configured duplex will goto default setting.
  - Default duplex on Cisco switches is half duplex (HDX)

Full duplex (FDX) is only available when collisions cannot occur on attached cable.
- Collision-free links are only guaranteed when a shared hub is **NOT** in use.

### CSMA/CD
- The orinal Ethernet spec. expected collisions to occur.
- Media was shared, creating a literal eletrical bus.
  - Any signal inuced onto wire could collide with another induced signal from another device.
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
- When a switch port connects through cable to a single other nonhub device, no collisions can possibly occur.
- Because collisions cannot occur, such segments can use *full-duplex* logic.



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
- Determines if officialy assigned or locally generated
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

Neighbor Discovery protocol is in fact an umbrella term for many interrelated subprotocols and mechanisms whose main responsibilities include the following:

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
- Used to get **Router Advertisments** from local link routers.  Beats waiting for scheduled advertisements.

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