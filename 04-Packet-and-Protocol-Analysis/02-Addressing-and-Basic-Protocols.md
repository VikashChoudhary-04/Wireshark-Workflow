# Wireshark Addressing and Basic Protocols

## Objective

This file builds practical packet-analysis knowledge around addressing and the basic protocols that appear frequently in network captures.

The goal is not to memorize protocol specifications.

The goal is to answer questions such as:

```text
Who is communicating?
How are the hosts addressing each other?
Is the communication local or routed?
How is an IP address mapped to a link-layer address?
Which protocol is responsible for the observed behavior?
What evidence does the packet provide?
```

The central workflow is:

```text
Address
    ↓
Protocol
    ↓
Packet fields
    ↓
Communication pattern
    ↓
Interpretation
```

The protocols covered here are:

```text
Ethernet
ARP
IPv4
IPv6
ICMP
ICMPv6
DHCP
```

TCP and UDP are covered in greater depth in the next protocol-analysis file.

---

## Addressing Across Layers

A network packet can contain several types of addressing information.

A common packet may contain:

```text
Ethernet
    Source MAC
    Destination MAC

IPv4
    Source IP
    Destination IP

TCP/UDP
    Source Port
    Destination Port
```

These identifiers belong to different layers.

Think of them as:

```text
MAC address
    ↓
Local link delivery

IP address
    ↓
Network-layer delivery

Port
    ↓
Transport endpoint
```

Do not treat them as interchangeable.

---

## MAC Addresses

A MAC address identifies a network interface at the link layer.

Example:

```text
Source MAC:
00:11:22:33:44:55

Destination MAC:
aa:bb:cc:dd:ee:ff
```

MAC addresses are especially important when analyzing:

```text
Local Ethernet communication
ARP
Switching behavior
Broadcasts
VLANs
Layer-2 troubleshooting
```

A MAC address does not directly tell you which application is communicating.

---

## IP Addresses

An IP address identifies a network-layer endpoint.

IPv4 example:

```text
192.168.1.10
```

IPv6 example:

```text
2001:db8::10
```

IP addresses are used for:

```text
Host identification
Routing
Packet direction
Conversation analysis
```

In Wireshark, common IPv4 fields are:

```text
ip.src
ip.dst
```

IPv6 fields are:

```text
ipv6.src
ipv6.dst
```

---

## Ports

Ports belong to the transport layer.

Example:

```text
192.168.1.10:51542
        ↓
192.168.1.20:443
```

Here:

```text
192.168.1.10
    → source IP

51542
    → source port

192.168.1.20
    → destination IP

443
    → destination port
```

This creates a useful endpoint model:

```text
Source:
IP + Port

Destination:
IP + Port
```

A transport conversation is therefore more specific than an IP conversation.

---

## Broadcast Addresses

Some traffic is intended for multiple hosts on a local network.

IPv4 broadcast examples include:

```text
255.255.255.255
```

and subnet-directed broadcasts where applicable.

Ethernet also has a broadcast MAC address:

```text
ff:ff:ff:ff:ff:ff
```

Broadcast traffic is important for protocols such as:

```text
ARP
DHCP
```

When you see broadcast traffic, ask:

```text
Why does every host need to receive this?
Is this expected for the protocol?
What information is being requested?
```

---

## Multicast

Multicast allows traffic to be delivered to a group of receivers.

You may encounter multicast in:

```text
IPv4
IPv6
DNS-related systems
Routing protocols
Service discovery
Streaming and other applications
```

Do not automatically treat multicast as suspicious.

Determine:

```text
Which multicast address?
Which protocol?
Which hosts participate?
What is the protocol purpose?
```

---

## Unicast

Unicast is communication between specific endpoints.

Example:

```text
192.168.1.10
      ↓
192.168.1.20
```

Most ordinary client-server traffic is unicast.

A practical classification is:

```text
Unicast
Broadcast
Multicast
```

When analyzing a capture, knowing which communication model is present can immediately narrow the investigation.

---

## Ethernet

Ethernet is a common link-layer technology.

A typical Ethernet frame may contain:

```text
Ethernet II
    Destination MAC
    Source MAC
    Type
```

followed by a network-layer protocol such as:

```text
IPv4
IPv6
ARP
```

A simplified structure is:

```text
Ethernet
    ↓
Network protocol
    ↓
Transport protocol
    ↓
Application protocol
```

---

## Ethernet Type

The Ethernet Type field indicates the payload protocol.

Common examples include:

```text
IPv4
IPv6
ARP
```

This allows Wireshark to identify the next protocol layer.

When examining an unfamiliar Ethernet frame:

```text
1. Check source MAC.
2. Check destination MAC.
3. Check Type.
4. Follow the next protocol.
```

---

## ARP

Address Resolution Protocol maps an IPv4 address to a link-layer address on a local network.

A simplified ARP request asks:

```text
Who has 192.168.1.1?
```

The corresponding reply may communicate:

```text
192.168.1.1 is at aa:bb:cc:dd:ee:ff
```

The workflow is:

```text
IPv4 address
    ↓
ARP request
    ↓
MAC address
    ↓
Ethernet communication
```

ARP is therefore an important bridge between network-layer addressing and local link-layer delivery.

---

## ARP Request

An ARP request is typically broadcast on the local network.

Conceptually:

```text
Sender:
192.168.1.10 / MAC-A

Question:
Who has 192.168.1.1?

Target:
192.168.1.1
```

When analyzing an ARP request, identify:

```text
Sender IP
Sender MAC
Target IP
Target MAC if present
Operation
```

The request helps determine what address resolution the host needs.

---

## ARP Reply

An ARP reply provides the requested mapping.

Conceptually:

```text
192.168.1.1
    ↓
aa:bb:cc:dd:ee:ff
```

The reply allows the sender to construct Ethernet frames for the destination.

When troubleshooting local connectivity, compare:

```text
ARP request
    ↓
ARP reply
    ↓
Subsequent IP traffic
```

If the expected ARP resolution does not occur, investigate that before assuming the problem is TCP or the application.

---

## Gratuitous ARP

A host may send ARP information without a normal request/reply exchange.

Such traffic can be used for legitimate purposes such as:

```text
Address conflict detection
Updating neighboring ARP caches
Failover
Interface changes
Redundancy mechanisms
```

Do not label gratuitous ARP as malicious solely because it is unsolicited.

Investigate:

```text
Who sent it?
Which address is announced?
How often?
What happened afterward?
```

---

## ARP and Duplicate IP Detection

ARP traffic can help investigate potential address conflicts.

Suppose two systems appear to claim the same IPv4 address.

Investigate:

```text
ARP sender IP
ARP sender MAC
Observed IP-to-MAC mappings
Timing
Frequency
Subsequent connectivity behavior
```

The goal is to determine whether the same IP appears associated with multiple MAC addresses.

A single observation should be correlated with the surrounding capture.

---

## ARP Filter Workflow

A useful starting filter is:

```text
arp
```

Then inspect:

```text
ARP requests
ARP replies
Sender addresses
Target addresses
Timing
```

For a particular IP:

```text
arp && arp.dst.proto_ipv4 == 192.168.1.1
```

or use the packet details and autocomplete to identify the exact field available in the capture.

Field discovery should take priority over guessing.

---

## IPv4

IPv4 is one of the most common network-layer protocols.

Important fields include:

```text
Source
Destination
TTL
Protocol
Length
Identification
Flags
Fragment offset
```

For most basic investigations, begin with:

```text
Source
Destination
Protocol
TTL
Fragmentation
```

Then inspect additional fields when the question requires them.

---

## IPv4 Addressing

Common private IPv4 ranges include:

```text
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

When analyzing a capture, private addresses often indicate internal network communication.

But address ranges alone do not establish:

```text
Trust
Ownership
Security status
User identity
```

They simply describe address space.

---

## IPv4 Routing Perspective

Suppose:

```text
Client:
192.168.1.10

Server:
192.168.1.20
```

The client may communicate directly at the local network level.

If the server is outside the local subnet, the client normally sends traffic toward a router/default gateway.

This distinction matters when interpreting MAC addresses.

For routed communication:

```text
Source IP:
Original client

Destination IP:
Remote server

Destination MAC:
Often the local next-hop interface rather than the remote server
```

This is an important concept.

The destination IP and destination MAC do not necessarily identify the same physical system.

---

## IPv4 TTL

TTL limits how many routing hops an IPv4 packet can traverse.

Each router that forwards the packet normally decrements the TTL.

In Wireshark:

```text
ip.ttl
```

can be inspected or filtered.

TTL can help with:

```text
Comparing packets
Understanding routing behavior
Investigating unexpected paths
Supporting path-related analysis
```

Do not use one TTL value as definitive proof of an operating system or exact network path.

---

## IPv4 Fragmentation

IPv4 can fragment packets.

Important fields include:

```text
Identification
Flags
Fragment offset
```

When fragmentation occurs:

```text
Original packet
    ↓
Multiple fragments
    ↓
Reassembly
    ↓
Higher-layer data
```

Investigate fragmentation when:

```text
Application data appears incomplete
Packets have unusual offsets
Transport headers are missing from later fragments
Reassembly behavior matters
```

---

## IPv6

IPv6 provides a larger address space and a different packet-header structure.

Important fields include:

```text
Source
Destination
Traffic Class
Flow Label
Next Header
Hop Limit
```

Common Wireshark fields include:

```text
ipv6.src
ipv6.dst
ipv6.hlim
```

When analyzing IPv6:

```text
1. Identify source.
2. Identify destination.
3. Identify Next Header.
4. Inspect extension headers.
5. Continue to transport/application protocol.
```

---

## IPv6 Address Types

IPv6 addresses can serve different purposes.

You may encounter:

```text
Global unicast
Link-local
Multicast
Loopback
Unique local addresses
```

A link-local address commonly begins with:

```text
fe80::
```

IPv6 multicast addresses commonly begin with:

```text
ff00::
```

Do not interpret an IPv6 address type as suspicious by itself.

Understand what role the address serves.

---

## IPv6 Neighbor Discovery

IPv6 does not use ARP.

Neighbor Discovery Protocol, carried through ICMPv6, provides related functions such as:

```text
Neighbor discovery
Router discovery
Address resolution
Neighbor reachability
```

This creates an important comparison:

```text
IPv4:
ARP

IPv6:
ICMPv6 Neighbor Discovery
```

When analyzing IPv6 local-network behavior, do not search for ARP and assume it should exist.

---

## ICMP

Internet Control Message Protocol is used for network-layer control and diagnostic messaging.

Common ICMP traffic includes:

```text
Echo request
Echo reply
Destination unreachable
Time exceeded
```

A common troubleshooting workflow is:

```text
ICMP request
    ↓
ICMP response
    ↓
Timing
    ↓
Reachability interpretation
```

ICMP is not simply "ping."

It supports several control and error functions.

---

## ICMP Echo Request

A typical ping begins with an ICMP Echo Request.

Conceptually:

```text
Client
    ↓
Echo Request
    ↓
Server
```

In Wireshark, you can filter:

```text
icmp
```

and inspect the ICMP type/code.

The key evidence includes:

```text
Source
Destination
Type
Code
Identifier
Sequence
Timestamp
```

---

## ICMP Echo Reply

A successful ping normally produces:

```text
Echo Request
    ↓
Echo Reply
```

When analyzing reachability, compare:

```text
Request timestamp
Reply timestamp
```

to understand round-trip timing.

A missing reply requires context.

Possible explanations include:

```text
Host unreachable
Firewall policy
ICMP filtering
Routing issue
Capture visibility problem
```

Do not immediately conclude that the host is offline.

---

## ICMP Destination Unreachable

An ICMP Destination Unreachable message indicates a network-layer error condition.

The exact meaning depends on the ICMP type/code.

When you see one:

```text
1. Identify source.
2. Identify destination.
3. Identify ICMP type/code.
4. Inspect the packet that triggered the message.
5. Correlate timing.
```

The embedded information can help identify the original communication.

---

## ICMP Time Exceeded

ICMP Time Exceeded messages can occur when TTL expires.

This is important for understanding:

```text
Routing loops
Traceroute-style behavior
Unexpected paths
Hop limits
```

A practical workflow is:

```text
ICMP Time Exceeded
    ↓
Identify quoted original packet
    ↓
Identify source/destination
    ↓
Inspect TTL
    ↓
Correlate with surrounding traffic
```

---

## ICMP Filtering Workflow

Start:

```text
icmp
```

Then narrow to an event:

```text
icmp.type
```

Use packet details and autocomplete to identify the exact type/code values relevant to the capture.

For troubleshooting, correlate:

```text
Request
Response/error
Timing
Original traffic
```

---

## ICMPv6

ICMPv6 performs both control/error functions and important IPv6 neighbor/router discovery functions.

Examples include:

```text
Neighbor Solicitation
Neighbor Advertisement
Router Solicitation
Router Advertisement
Echo Request
Echo Reply
```

A broad filter is:

```text
icmpv6
```

Then inspect:

```text
Type
Code
Source
Destination
Options
Referenced addresses
```

---

## Neighbor Solicitation

Neighbor Solicitation helps IPv6 hosts discover or verify neighbors.

Conceptually:

```text
Host A
    ↓
Who owns this IPv6 address?
    ↓
Neighbor Solicitation
```

When analyzing it, inspect:

```text
Source IPv6
Target IPv6
ICMPv6 type
Options
Link-layer information
```

This is part of normal IPv6 operation.

---

## Neighbor Advertisement

A Neighbor Advertisement provides information in response to neighbor discovery or may be sent proactively.

A simplified relationship is:

```text
Neighbor Solicitation
        ↓
Neighbor Advertisement
```

Compare:

```text
Target address
Source address
Timing
Options
```

when troubleshooting IPv6 connectivity.

---

## Router Solicitation and Advertisement

IPv6 hosts can discover routers using ICMPv6.

The sequence may be:

```text
Router Solicitation
        ↓
Router Advertisement
```

Router Advertisements can provide network configuration information.

When analyzing them, inspect:

```text
Router information
Prefix information
Lifetime values
Options
```

This can help explain how a host obtained or learned network configuration.

---

## DHCP

Dynamic Host Configuration Protocol is used to provide IPv4 network configuration.

A common DHCP exchange is:

```text
Discover
    ↓
Offer
    ↓
Request
    ↓
ACK
```

The exact packet sequence can vary.

DHCP commonly provides information such as:

```text
IP address
Subnet information
Default gateway
DNS server
Lease information
```

---

## DHCP Discover

A client without an IPv4 configuration may send a DHCP Discover.

Conceptually:

```text
Client
    ↓
I need network configuration.
    ↓
DHCP Discover
```

The packet is commonly broadcast because the client may not yet know which DHCP server to contact.

---

## DHCP Offer

A DHCP server may respond with an Offer.

The offer can contain proposed configuration information.

When analyzing:

```text
DHCP Discover
    ↓
DHCP Offer
```

compare:

```text
Client identity
Offered address
Server identity
Timing
Options
```

---

## DHCP Request

The client can request the offered configuration.

The sequence may therefore become:

```text
Discover
    ↓
Offer
    ↓
Request
```

This is an important point where you can determine:

```text
Which address the client requested
Which DHCP server was selected
```

---

## DHCP ACK

The server may confirm the configuration with an acknowledgment.

Typical flow:

```text
Discover
    ↓
Offer
    ↓
Request
    ↓
ACK
```

If the expected sequence is incomplete, investigate:

```text
Which packet is missing?
Which server responded?
Was another server involved?
Were there retries?
```

---

## DHCP Troubleshooting Workflow

Question:

```text
Why did a client fail to obtain an IPv4 configuration?
```

Start with:

```text
dhcp
```

or the appropriate DHCP protocol display shown by Wireshark.

Then identify:

```text
Discover
Offer
Request
ACK
```

Build the investigation:

```text
Client sends Discover
        ↓
Does Offer return?
        ↓
Does client Request?
        ↓
Does ACK return?
        ↓
What configuration was provided?
```

If the exchange stops, investigate the missing stage.

---

## Addressing Workflow

When analyzing any new packet, identify:

```text
1. Link-layer source.
2. Link-layer destination.
3. Network-layer source.
4. Network-layer destination.
5. Transport source port.
6. Transport destination port.
7. Application protocol.
```

Then ask:

```text
What does each address represent?
```

For example:

```text
MAC:
Local link endpoint

IP:
Network-layer endpoint

Port:
Transport endpoint

Application:
Protocol-specific communication
```

---

## Local vs Routed Communication

A useful troubleshooting question is:

```text
Is this communication local or routed?
```

Inspect:

```text
Source IP
Destination IP
Source MAC
Destination MAC
Gateway-related traffic
ARP/Neighbor Discovery
```

For local communication, the destination MAC may correspond to the destination host.

For routed communication, the destination MAC may correspond to the next-hop router while the destination IP remains the remote host.

This distinction is critical when analyzing packet paths.

---

## Addressing and Wireshark Filters

Common starting filters include:

```text
arp
```

```text
ip
```

```text
ipv6
```

```text
icmp
```

```text
icmpv6
```

For a specific IPv4 host:

```text
ip.addr == 192.168.1.10
```

For a source:

```text
ip.src == 192.168.1.10
```

For a destination:

```text
ip.dst == 192.168.1.10
```

For IPv6:

```text
ipv6.addr == 2001:db8::10
```

When uncertain about field syntax, use Wireshark autocomplete and packet context.

---

## Protocol Identification Workflow

Do not identify a protocol only from a port number.

Use this workflow:

```text
Port
    ↓
Packet dissection
    ↓
Protocol tree
    ↓
Protocol-specific fields
    ↓
Application behavior
```

For example:

```text
TCP port 443
```

suggests a possible web/TLS service.

Then inspect whether Wireshark actually identifies:

```text
TLS
```

and inspect the handshake.

---

## Protocol Correlation

Protocols often explain each other.

Examples:

```text
ARP
    ↓
Ethernet communication
    ↓
IPv4
    ↓
TCP
    ↓
TLS
```

Or:

```text
DHCP
    ↓
IPv4 configuration
    ↓
DNS
    ↓
TCP/UDP application traffic
```

Or:

```text
IPv6 Neighbor Discovery
    ↓
IPv6 communication
    ↓
TCP/UDP
    ↓
Application
```

A strong analyst follows these relationships instead of analyzing each protocol in isolation.

---

## Practical Exercise 1 — Address Mapping

Find an ARP request and answer:

```text
Who sent it?
Which IPv4 address was requested?
Which MAC address was associated with the sender?
What was the destination MAC?
Was a reply observed?
```

Then identify the corresponding IP traffic, if present.

---

## Practical Exercise 2 — Local vs Routed

Find an IPv4 conversation.

Determine:

```text
Source IP
Destination IP
Source MAC
Destination MAC
```

Then reason:

```text
Does the destination MAC represent the destination host?
Or does it appear to represent a next hop?
```

Use the network context rather than guessing.

---

## Practical Exercise 3 — ARP Troubleshooting

Find a host that performs ARP resolution.

Determine:

```text
ARP request
ARP reply
Timing
IP-to-MAC mapping
Subsequent communication
```

Then answer:

```text
Did address resolution appear successful?
```

---

## Practical Exercise 4 — Ping Analysis

Find an ICMP Echo Request.

Locate the corresponding Echo Reply.

Record:

```text
Request packet number
Reply packet number
Source
Destination
Sequence
Timing
```

Then determine the approximate request/reply delay from packet timestamps.

---

## Practical Exercise 5 — Missing ICMP Response

Find or create a situation where an ICMP Echo Request has no visible reply.

Investigate:

```text
Was the request transmitted?
Was the destination reachable through another protocol?
Was an ICMP error returned?
Could filtering or capture visibility explain the absence?
```

Do not conclude that the host is offline without supporting evidence.

---

## Practical Exercise 6 — DHCP Exchange

Find a DHCP exchange and identify:

```text
Discover
Offer
Request
ACK
```

Record:

```text
Client identity
Offered/requested address
DHCP server
Relevant configuration
Timing
```

Explain where the client obtains its IPv4 configuration.

---

## Practical Exercise 7 — IPv6 Neighbor Discovery

If IPv6 is available, locate:

```text
Neighbor Solicitation
Neighbor Advertisement
```

Identify:

```text
Source
Target
Type
Options
Timing
```

Explain the role of the exchange.

---

## Practical Exercise 8 — IPv4 and IPv6 Comparison

Find one IPv4 conversation and one IPv6 conversation.

Compare:

```text
Address format
Network-layer fields
Next-protocol identification
Neighbor discovery
Hop control
Transport protocol
Application protocol
```

Write down the practical differences you observe in Wireshark.

---

## Practical Exercise 9 — Protocol Chain

Choose one complete communication flow.

Write its protocol chain:

```text
Ethernet
    ↓
________
    ↓
________
    ↓
________
```

Then identify the most important field at each layer.

---

## Practical Exercise 10 — Addressing Evidence

Choose one packet and create an evidence record:

```text
Packet number:
Timestamp:
Source MAC:
Destination MAC:
Source IP:
Destination IP:
Source port:
Destination port:
Application protocol:
Important field:
Interpretation:
Uncertainty:
```

The goal is to turn packet inspection into reproducible analysis.

---

## Common Mistakes

### Mistake 1 — Treating MAC and IP as the Same Identifier

They represent different layers.

### Mistake 2 — Assuming Destination MAC Equals Destination IP Host

In routed communication, the destination MAC may identify the next hop.

### Mistake 3 — Treating Ports as Definitive Protocol Identity

A port suggests a service; packet dissection provides stronger evidence.

### Mistake 4 — Assuming ARP Exists for IPv6

IPv6 uses Neighbor Discovery through ICMPv6 instead.

### Mistake 5 — Treating ICMP as Only Ping

ICMP also carries network control and error information.

### Mistake 6 — Treating DHCP as a Single Packet

DHCP commonly involves a multi-step exchange.

### Mistake 7 — Ignoring Broadcast and Multicast

These are legitimate communication models used by many protocols.

### Mistake 8 — Interpreting One Address Without Direction

Always consider source and destination.

---

## Addressing Investigation Checklist

When analyzing network communication, ask:

```text
[ ] What is the source MAC?
[ ] What is the destination MAC?
[ ] What is the source IP?
[ ] What is the destination IP?
[ ] Is the traffic unicast, broadcast, or multicast?
[ ] Is the communication local or routed?
[ ] What transport protocol is used?
[ ] What are the source and destination ports?
[ ] What application protocol is present?
[ ] Is ARP involved?
[ ] Is IPv6 Neighbor Discovery involved?
[ ] Is ICMP/ICMPv6 involved?
[ ] Is DHCP involved?
[ ] What does the addressing evidence actually prove?
```

---

## Professional Addressing Workflow

Use:

```text
Link Layer
    ↓
MAC addresses
    ↓
Network Layer
    ↓
IP addresses
    ↓
Routing/locality
    ↓
Transport Layer
    ↓
Ports
    ↓
Application Layer
    ↓
Protocol behavior
```

When troubleshooting, move through the layers systematically.

When investigating security events, use the same layers to establish:

```text
Who
    ↓
Communicated with whom
    ↓
Using what protocol
    ↓
Over which service
    ↓
At what time
    ↓
With what observable behavior
```

---

## Completion Criteria

You are ready to continue when you can independently:

* Distinguish MAC addresses, IP addresses, and ports.
* Identify unicast, broadcast, and multicast traffic.
* Read Ethernet source and destination addresses.
* Explain the role of EtherType.
* Analyze ARP requests and replies.
* Understand basic IPv4 addressing.
* Understand basic IPv6 addressing.
* Recognize IPv4 fragmentation fields.
* Understand the practical role of TTL and Hop Limit.
* Recognize ICMP Echo traffic.
* Analyze ICMP error messages at a basic level.
* Recognize ICMPv6 Neighbor Discovery.
* Analyze basic DHCP exchanges.
* Distinguish local communication from routed communication.
* Understand why destination MAC and destination IP can represent different systems.
* Use addressing information to construct display filters.
* Correlate lower-layer addressing with transport and application behavior.
* Document addressing evidence without overstating what it proves.

Once these concepts are comfortable, you can move deeper into TCP and UDP analysis, where packet sequence, connection state, timing, retransmission, and flow control become central to troubleshooting and investigation.
