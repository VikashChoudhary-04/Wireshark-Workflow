# Wireshark Packet Layers and Fields

## Objective

This file teaches you how to read a packet as a structured collection of protocol layers and fields.

The goal is not to memorize networking theory.

The goal is to answer practical questions such as:

```text
Who sent this packet?
Who received it?
Which protocol is being used?
Which service is involved?
What happened at this layer?
What evidence does this packet provide?
What information is missing?
```

The central model is:

```text
Packet
  ↓
Protocol layers
  ↓
Fields
  ↓
Values
  ↓
Meaning
  ↓
Evidence
```

By the end of this workflow, you should be able to move confidently between the packet list, packet details, and packet bytes.

---

## The Packet as Evidence

A captured packet is not simply a row in Wireshark.

It contains multiple layers of information.

A simplified packet may look like:

```text
Ethernet
    ↓
IPv4
    ↓
TCP
    ↓
Application Protocol
    ↓
Application Data
```

Each layer contributes information.

For example:

```text
Ethernet
    Source MAC
    Destination MAC

IPv4
    Source IP
    Destination IP
    TTL
    Protocol

TCP
    Source Port
    Destination Port
    Sequence Number
    Acknowledgment Number
    Flags
    Window

Application Protocol
    Request
    Response
    Application fields
```

The exact layers depend on the traffic.

---

## Why Packet Layers Matter

Suppose you see:

```text
192.168.1.10 → 192.168.1.20
```

That tells you something about IP communication.

But it does not tell you everything.

You still need to ask:

```text
Which transport protocol?
Which source port?
Which destination port?
Which application protocol?
What TCP state?
What application event?
```

A packet becomes useful when you interpret the relevant layers together.

---

## The Three Main Wireshark Panes

A normal Wireshark layout contains three important areas:

```text
Packet List
    ↓
Packet Details
    ↓
Packet Bytes
```

Each answers a different question.

### Packet List

The packet list provides a high-level view.

Typical columns include:

```text
No.
Time
Source
Destination
Protocol
Length
Info
```

Use it for:

```text
Finding patterns
Scanning large captures
Identifying protocol changes
Locating events
Comparing packet sequences
```

### Packet Details

The packet details pane exposes the decoded protocol structure.

Use it for:

```text
Field inspection
Protocol relationships
Flags
Addresses
Ports
Headers
Protocol-specific evidence
```

### Packet Bytes

The bytes pane shows the actual captured bytes.

Use it when you need to:

```text
Verify raw values
Understand encoding
Inspect headers
Investigate dissection problems
Confirm what was actually captured
```

These panes should be used together.

---

## Read from Outer Layers to Inner Layers

A useful default workflow is:

```text
Frame
  ↓
Link Layer
  ↓
Network Layer
  ↓
Transport Layer
  ↓
Application Layer
```

For common Ethernet/IP/TCP traffic:

```text
Frame
    ↓
Ethernet II
    ↓
Internet Protocol Version 4
    ↓
Transmission Control Protocol
    ↓
Application Protocol
```

Do not skip directly to the application layer.

The lower layers often explain what happened to the application traffic.

---

## Frame Layer

The frame represents the captured packet as Wireshark received it.

Common frame information includes:

```text
Packet number
Capture timestamp
Captured length
Original length
Encapsulation
Protocols present
```

Frame information can answer questions such as:

```text
When was this packet captured?
Which packet number is it?
How large was it?
Was the entire packet captured?
```

For example:

```text
frame.number
frame.time
frame.len
```

are useful display-filter fields.

---

## Captured Length vs Original Length

A packet may have:

```text
Captured length
```

and:

```text
Original length
```

These values are important when investigating truncated captures.

If the captured packet is shorter than the original packet, the packet may not contain all of the data that existed on the wire.

This creates an important forensic limitation:

```text
Missing bytes
    ↓
Missing protocol information
    ↓
Incomplete interpretation
```

Never assume that an incomplete capture contains the complete packet.

---

## Link Layer

On Ethernet networks, the link-layer header is commonly:

```text
Ethernet II
```

Important fields include:

```text
Destination MAC address
Source MAC address
EtherType
```

Example:

```text
Destination: aa:bb:cc:dd:ee:ff
Source: 11:22:33:44:55:66
Type: IPv4
```

The link layer answers questions such as:

```text
Which MAC address sent this frame?
Which MAC address should receive it?
What network-layer protocol follows?
```

---

## MAC Addresses

MAC addresses identify link-layer interfaces within the relevant network context.

For example:

```text
Source MAC:
11:22:33:44:55:66

Destination MAC:
aa:bb:cc:dd:ee:ff
```

Do not automatically treat a MAC address as equivalent to an IP address.

They belong to different layers.

A packet can have:

```text
Source MAC ≠ Source IP
```

and that can be completely normal.

---

## EtherType

EtherType helps identify what protocol follows the Ethernet header.

Common examples include:

```text
IPv4
IPv6
ARP
```

The important reasoning pattern is:

```text
Ethernet
    ↓
EtherType
    ↓
Next protocol
```

This allows Wireshark to continue decoding the packet.

---

## ARP as a Layer Relationship

ARP is different from ordinary IPv4 transport traffic.

An ARP exchange can answer:

```text
Which MAC address owns this IPv4 address?
```

A simplified workflow is:

```text
ARP Request
    ↓
Who has IP X?
    ↓
ARP Reply
    ↓
IP X is at MAC Y
```

When troubleshooting connectivity, ARP can therefore matter before TCP or application traffic is even considered.

---

## Network Layer

The network layer commonly contains IPv4 or IPv6.

For IPv4, important fields include:

```text
Source address
Destination address
TTL
Protocol
Total length
Identification
Flags
Fragment offset
Header checksum
```

Not every field will be equally important in every investigation.

Focus on the fields that answer the current question.

---

## IPv4 Source and Destination

The most commonly used IPv4 fields are:

```text
ip.src
ip.dst
```

Example:

```text
Source:
192.168.1.10

Destination:
192.168.1.20
```

This establishes packet direction at the IP layer.

But remember:

```text
Packet direction
≠
Application intent
```

You still need the transport and application layers.

---

## IPv4 Protocol Field

IPv4 identifies the next protocol through its protocol field.

For example:

```text
IPv4
    ↓
TCP
```

or:

```text
IPv4
    ↓
UDP
```

or:

```text
IPv4
    ↓
ICMP
```

This creates the relationship:

```text
IP
 ↓
Transport/control protocol
```

Understanding this relationship makes packet navigation much easier.

---

## IPv6

IPv6 provides the same broad networking role as IPv4 but uses a different addressing and header structure.

Important fields include:

```text
Source address
Destination address
Traffic class
Flow label
Next header
Hop limit
```

Do not assume that an investigation involving IPv4 automatically covers IPv6.

When analyzing a network, check whether both are present.

---

## IPv6 Next Header

IPv6 uses the Next Header mechanism to identify what follows.

For example:

```text
IPv6
  ↓
TCP
```

or:

```text
IPv6
  ↓
UDP
```

IPv6 may also contain extension headers.

A practical analysis workflow is:

```text
IPv6
  ↓
Check source/destination
  ↓
Check Next Header
  ↓
Inspect extension headers if present
  ↓
Continue to transport/application layer
```

---

## TTL and Hop Limit

IPv4 uses:

```text
TTL
```

IPv6 uses:

```text
Hop Limit
```

These values limit how many routing hops a packet can traverse.

They can also provide useful investigative context.

For example, unexpected changes in observed TTL values may help when comparing traffic paths or operating systems.

However:

```text
TTL value alone
```

is not sufficient to identify an operating system or network path with certainty.

Treat it as supporting evidence.

---

## Fragmentation

Large IP packets can be fragmented.

IPv4 provides fields related to fragmentation such as:

```text
Identification
Flags
Fragment offset
```

Fragmentation matters because the application payload may not be contained in one packet.

The workflow becomes:

```text
Packet
    ↓
Fragmented?
    ↓
Identify related fragments
    ↓
Reassembly
    ↓
Inspect higher-layer protocol
```

Do not assume that every packet containing an IP header also contains a complete TCP or application message.

---

## Transport Layer

The most common transport protocols you will analyze are:

```text
TCP
UDP
```

They provide different communication behavior.

### TCP

TCP is:

```text
Connection-oriented
Stateful
Sequence-aware
Acknowledgment-based
```

It provides fields useful for analyzing:

```text
Connection establishment
Data transfer
Retransmissions
Acknowledgments
Flow control
Connection termination
Resets
```

### UDP

UDP is:

```text
Connectionless
Lightweight
Message-oriented
```

It does not provide the same TCP connection state.

The analysis approach must therefore change.

---

## TCP Header

Important TCP fields include:

```text
Source port
Destination port
Sequence number
Acknowledgment number
Header length
Flags
Window
Checksum
Urgent pointer
Options
```

Not every field is equally important in every investigation.

A common troubleshooting focus is:

```text
Ports
Flags
Sequence numbers
Acknowledgments
Window
Options
Timing
```

---

## TCP Source and Destination Ports

TCP ports identify endpoints of the transport connection.

Example:

```text
Source port:
51542

Destination port:
443
```

This often represents:

```text
Client ephemeral port
        ↓
Server service port
```

But do not assume the role from the port number alone.

Use packet direction and connection context.

---

## TCP Flags

TCP flags communicate important connection-state information.

Common flags include:

```text
SYN
ACK
FIN
RST
PSH
URG
ECE
CWR
```

For basic connection analysis, focus first on:

```text
SYN
ACK
FIN
RST
```

A simplified sequence is:

```text
Client → SYN
Server → SYN-ACK
Client → ACK
```

This represents the normal beginning of a TCP connection.

---

## TCP Flags as Evidence

A TCP RST packet can indicate that a connection was reset.

But:

```text
RST
```

does not automatically mean:

```text
Attack
```

or:

```text
Network failure
```

Possible explanations depend on context.

Always inspect:

```text
Previous packets
Direction
Timing
Application behavior
Connection state
```

The flag is evidence, not a complete conclusion.

---

## TCP Sequence Numbers

TCP sequence numbers help track the ordering of data.

They allow Wireshark to analyze:

```text
Expected sequence
Received sequence
Retransmission
Out-of-order delivery
Missing segments
```

When analyzing a TCP stream, do not look only at packet numbers.

Packet number:

```text
Wireshark capture order
```

Sequence number:

```text
TCP data-stream position
```

These are different concepts.

---

## TCP Acknowledgment Numbers

Acknowledgment numbers indicate what data has been received from the peer.

Together with sequence numbers they allow you to reason about:

```text
Data delivery
Acknowledgments
Missing data
Retransmissions
Flow behavior
```

A useful conceptual model is:

```text
Sender
  ↓
Sequence number
  ↓
Receiver
  ↓
Acknowledgment
  ↓
Sender
```

---

## TCP Window

The TCP receive window communicates how much data the receiver is currently prepared to accept.

A very small or zero window can affect throughput.

When investigating application slowness, inspect:

```text
Window
Timing
Acknowledgments
Retransmissions
Application behavior
```

Do not attribute slowness to the window without correlating it with the rest of the connection.

---

## TCP Options

TCP options can provide additional information.

Common options include:

```text
Maximum Segment Size
Window Scale
Selective Acknowledgment
Timestamps
```

These options can influence how a TCP connection behaves.

For example:

```text
Window Scale
```

can affect the interpretation of the effective receive window.

Treat TCP options as part of the connection's negotiated behavior.

---

## UDP

UDP packets generally contain:

```text
Source port
Destination port
Length
Checksum
Payload
```

There is no TCP-style:

```text
SYN
ACK
FIN
RST
Sequence number
```

Therefore, do not apply TCP connection-state reasoning directly to UDP.

For UDP analysis, focus more on:

```text
Source
Destination
Ports
Packet frequency
Timing
Payload/protocol
Request/response patterns
```

---

## Application Layer

Above TCP or UDP, Wireshark may decode an application protocol.

Examples include:

```text
DNS
DHCP
HTTP
TLS
SSH
FTP
SMTP
SMB
LDAP
Kerberos
SNMP
NTP
```

The application layer is where the packet often answers the most user-visible question.

For example:

```text
DNS
    Which name was requested?

HTTP
    Which resource was requested?

TLS
    Which server was indicated during negotiation?

DHCP
    Which address configuration was requested?

NTP
    Which time service was contacted?
```

---

## Protocol Dissection

Wireshark examines packet bytes and attempts to decode them into protocol fields.

This process is called protocol dissection.

A typical result might look like:

```text
Ethernet II
Internet Protocol Version 4
Transmission Control Protocol
TLS
```

Each layer exposes fields that Wireshark understands.

However, dissection is not infallible.

Problems can occur because of:

```text
Malformed packets
Truncated captures
Unsupported protocols
Incorrect decoding
Encrypted payloads
Non-standard ports
Encapsulation
Capture artifacts
```

When the decoded structure looks wrong, investigate the packet bytes and capture context.

---

## Dissection Is an Interpretation Layer

Remember:

```text
Captured bytes
        ↓
Wireshark dissection
        ↓
Displayed fields
```

The fields shown by Wireshark are an interpretation of the captured bytes.

This is extremely useful.

But when evidence is disputed or unexpected, you may need to inspect the raw bytes.

---

## Packet Bytes

The packet bytes pane displays the raw captured data.

You may see:

```text
Hexadecimal bytes
ASCII representation
```

For example:

```text
45 00 00 34 ...
```

The exact representation depends on the packet.

The bytes pane is especially useful when:

```text
A field is not decoded as expected.
You need to verify raw data.
A protocol is partially dissected.
You suspect truncation.
You need to understand offsets.
```

---

## Bytes and Fields Are Connected

When you select a field in the packet details pane, Wireshark can highlight the corresponding bytes.

This creates a valuable relationship:

```text
Field
  ↕
Raw bytes
```

For example:

```text
TCP destination port
        ↕
Corresponding bytes
```

Learning to move between these views builds deeper packet understanding.

---

## Packet Encapsulation

Not every capture has the same outer structure.

You may encounter:

```text
Ethernet
Linux cooked capture
Loopback
802.11 wireless
VLAN
GRE
VXLAN
VPN/tunnel encapsulation
```

Therefore, do not assume that every packet starts with:

```text
Ethernet II
```

The capture's link-layer type determines how the packet begins.

---

## VLAN Tags

VLAN traffic may introduce additional Ethernet-related information.

A simplified structure may look like:

```text
Ethernet
    ↓
802.1Q VLAN
    ↓
IPv4
    ↓
TCP
    ↓
Application
```

Important VLAN information may include:

```text
VLAN ID
Priority
Tagging information
```

When analyzing segmented networks, VLAN information can help explain where traffic belongs.

---

## Tunneling

A packet may contain another protocol inside an outer protocol.

For example:

```text
Outer packet
    ↓
Tunnel
    ↓
Inner packet
    ↓
Transport
    ↓
Application
```

This means the packet can contain multiple logical networking contexts.

When you encounter tunneling:

```text
Identify outer layer
    ↓
Identify tunnel protocol
    ↓
Inspect inner packet
    ↓
Continue analysis
```

Do not stop analysis at the outer header.

---

## Encapsulation vs Encryption

These concepts are different.

### Encapsulation

One protocol is carried inside another.

Example:

```text
Outer IP
    ↓
Tunnel
    ↓
Inner IP
```

The inner protocol may still be visible.

### Encryption

The contents are protected from direct inspection.

Example:

```text
TLS
    ↓
Encrypted application data
```

Metadata may remain visible even when application content is encrypted.

Understanding this distinction prevents incorrect assumptions about visibility.

---

## Protocol Layers and Display Filters

Packet structure directly supports filtering.

Examples:

```text
ip.src
```

comes from the IP layer.

```text
tcp.dstport
```

comes from TCP.

```text
dns.qry.name
```

comes from DNS.

This relationship can be represented as:

```text
Question
    ↓
Protocol layer
    ↓
Field
    ↓
Filter
```

For example:

```text
Question:
Which DNS names were queried?

Layer:
DNS

Field:
dns.qry.name
```

---

## Packet Fields as Investigative Evidence

Every field should be interpreted according to its meaning.

Examples:

```text
Source IP
    → packet source at IP layer

Destination IP
    → packet destination at IP layer

TCP destination port
    → transport-layer destination endpoint

TCP RST
    → TCP reset flag is present

DNS query name
    → DNS query-name field contains a value
```

Do not turn a field into a stronger claim than the field actually supports.

---

## Do Not Confuse Observation with Interpretation

Consider:

```text
tcp.flags.reset == 1
```

Observation:

```text
A TCP packet with the RST flag is present.
```

Interpretation:

```text
The connection may have been terminated/reset.
```

Conclusion:

```text
The application failed because the server rejected the connection.
```

These are increasingly stronger claims.

The capture must provide enough evidence for each step.

---

## Field Values Need Context

Consider:

```text
tcp.dstport == 443
```

Observation:

```text
TCP destination port is 443.
```

Possible interpretation:

```text
This may be HTTPS/TLS-related traffic.
```

But a stronger statement:

```text
The application is definitely HTTPS.
```

requires protocol evidence.

Always distinguish:

```text
Port
Protocol
Application
```

---

## Read the Packet Hierarchically

When analyzing an unfamiliar packet, use:

```text
1. Frame
2. Link layer
3. Network layer
4. Transport layer
5. Application layer
6. Relevant fields
7. Raw bytes if necessary
```

At every layer ask:

```text
What does this layer tell me?
Is it relevant to my question?
Does it explain something above or below it?
```

---

## Packet Reading Workflow

Use this workflow for every unfamiliar packet:

```text
Select packet
    ↓
Read packet-list summary
    ↓
Expand protocol layers
    ↓
Identify source/destination
    ↓
Identify transport protocol
    ↓
Identify ports
    ↓
Identify application protocol
    ↓
Inspect important fields
    ↓
Check timing and context
    ↓
Inspect raw bytes if required
    ↓
Correlate with neighboring packets
```

This should eventually become automatic.

---

## Practical Exercise 1 — Read One Packet

Choose any TCP packet.

Identify:

```text
Frame:
Packet number
Timestamp
Length

Ethernet:
Source MAC
Destination MAC

IP:
Source IP
Destination IP
TTL

TCP:
Source port
Destination port
Flags
Sequence number
Acknowledgment number
Window

Application:
Protocol
Relevant fields
```

Do not move on until you can locate each item in the packet details pane.

---

## Practical Exercise 2 — Compare Two Packets

Choose:

```text
A TCP SYN packet
A TCP SYN-ACK packet
```

Compare:

```text
Source
Destination
Ports
Flags
Sequence number
Acknowledgment number
Window
Timing
```

Explain how the two packets represent different stages of the TCP handshake.

---

## Practical Exercise 3 — Compare TCP and UDP

Find:

```text
One TCP packet
One UDP packet
```

Compare their fields.

Identify:

```text
Which fields exist in TCP but not UDP?
Which fields are shared?
How does the analysis workflow differ?
```

---

## Practical Exercise 4 — Read an IPv4 Packet

Choose an IPv4 packet and identify:

```text
Source
Destination
TTL
Protocol
Length
Identification
Fragmentation fields
```

Then determine which transport protocol follows IPv4.

---

## Practical Exercise 5 — Read an IPv6 Packet

If IPv6 traffic exists in the capture, identify:

```text
Source
Destination
Hop Limit
Next Header
Extension headers if present
Transport protocol
```

Compare the structure with IPv4.

---

## Practical Exercise 6 — Read an ARP Exchange

Find an ARP request and reply.

Identify:

```text
Sender MAC
Sender IP
Target MAC
Target IP
Operation
```

Explain:

```text
What question did the request ask?
What information did the reply provide?
```

---

## Practical Exercise 7 — Follow a Field to the Bytes

Select a field in the packet details pane.

Then locate the highlighted bytes.

Repeat for:

```text
IP source address
IP destination address
TCP destination port
TCP flags
```

The goal is to understand:

```text
Decoded field
    ↕
Raw packet bytes
```

---

## Practical Exercise 8 — Identify Encapsulation

Find a packet with more than the basic:

```text
Ethernet
IP
TCP
```

structure.

Identify the additional protocol layer.

Then explain:

```text
What is the outer protocol?
What is the inner protocol?
Why is the extra layer present?
```

---

## Practical Exercise 9 — Detect Truncation

Find a packet where captured length and original length differ, if the capture contains one.

Identify:

```text
Captured length
Original length
```

Then answer:

```text
What information might be missing?
Could the missing bytes affect protocol analysis?
```

---

## Practical Exercise 10 — Layer-to-Question Mapping

For each question, identify the primary layer:

```text
Which interface sent the frame?
Which host sent the packet?
Which service received it?
Which TCP event occurred?
Which DNS name was queried?
Which HTTP method was used?
Which TLS metadata is visible?
```

Map each question to:

```text
Layer
Field
Expected evidence
```

---

## Common Packet-Reading Mistakes

Avoid these mistakes.

### Mistake 1 — Looking Only at the Packet List

The packet list is a summary.

Important evidence may only be visible in packet details.

### Mistake 2 — Looking Only at the Application Layer

Lower-layer problems can explain application failures.

### Mistake 3 — Treating Ports as Protocol Proof

Ports are useful indicators but are not absolute proof of application identity.

### Mistake 4 — Treating One Packet as the Entire Story

Connections are sequences.

### Mistake 5 — Ignoring Direction

Source and destination matter.

### Mistake 6 — Ignoring Timing

Timing can distinguish normal behavior from delayed or repeated behavior.

### Mistake 7 — Ignoring Capture Limitations

A missing packet may be caused by:

```text
Capture location
Capture duration
Packet loss
Filtering
Truncation
Asymmetric traffic
```

### Mistake 8 — Trusting Every Decoded Field Without Question

When something appears inconsistent, verify the packet bytes and surrounding context.

---

## Packet Layers and Troubleshooting

When troubleshooting a failure, move upward through the stack.

Example:

```text
Link
 ↓
Network
 ↓
Transport
 ↓
Application
```

Ask:

```text
Is the frame visible?
Is the IP communication occurring?
Is the transport connection established?
Is the application exchange occurring?
Where does expected behavior stop?
```

This produces a structured troubleshooting workflow.

---

## Packet Layers and Security Analysis

The same model works for security investigations.

For example:

```text
Ethernet
    ↓
Identify local communication

IP
    ↓
Identify hosts

TCP/UDP
    ↓
Identify services

Application
    ↓
Identify behavior

Timing and correlation
    ↓
Understand sequence
```

A suspicious application event should therefore be correlated with its lower-layer communication.

---

## Packet Layers and Forensics

For forensic analysis, record evidence by layer.

Example:

```text
Frame:
Packet 4821 at 14:32:11.234

IP:
192.168.1.10 → 192.168.1.50

TCP:
51542 → 443
SYN

TLS:
ClientHello
```

This creates a reproducible evidence trail.

The objective is not to record everything.

Record the fields necessary to support the finding.

---

## Professional Packet-Reading Standard

You should eventually be able to select an unfamiliar packet and quickly answer:

```text
What is this?
Who sent it?
Who received it?
Which layer am I looking at?
What protocol comes next?
Which service is involved?
What event does this packet represent?
What does the packet prove?
What does it not prove?
Which packets should I inspect next?
```

This is the foundation for deeper protocol analysis.

---

## Completion Criteria

You are ready to continue when you can independently:

* Navigate from frame information to application fields.
* Explain the major layers of a packet.
* Distinguish link, network, transport, and application information.
* Identify source and destination addresses.
* Identify source and destination ports.
* Read basic TCP flags.
* Explain sequence and acknowledgment numbers at a conceptual level.
* Distinguish TCP and UDP analysis.
* Read basic IPv4 and IPv6 fields.
* Recognize ARP traffic.
* Recognize fragmentation and encapsulation.
* Distinguish encapsulation from encryption.
* Move between decoded fields and raw packet bytes.
* Identify capture limitations.
* Separate packet observations from interpretations.
* Use packet layers to guide display-filter construction.
* Use packet structure to troubleshoot connectivity and application problems.

Once this becomes comfortable, protocol analysis can move from simply reading packets to understanding how complete network conversations behave.
