# Packet Inspection

## Objective

The goal of this file is to develop the ability to inspect a packet systematically and extract useful evidence from it.

Packet inspection is more than reading fields.

A professional analyst should be able to move through a packet from:

```text
Packet
  ↓
Frame metadata
  ↓
Link-layer information
  ↓
Network-layer information
  ↓
Transport-layer information
  ↓
Application-layer information
  ↓
Raw bytes
  ↓
Evidence
```

The objective is to answer questions such as:

* Who sent this packet?
* Who received it?
* When did it happen?
* Which protocol is involved?
* Which connection does it belong to?
* What does each important field tell me?
* Does the packet support or contradict my hypothesis?
* What information is visible?
* What information is unavailable because of encryption, truncation, or capture limitations?
* What packet should I inspect next?

## When Packet Inspection Is Useful

Inspect individual packets when aggregate information is not enough.

Typical situations include:

* Investigating a failed connection
* Understanding a TCP handshake
* Examining a DNS request or response
* Determining why a server returned an error
* Checking retransmissions
* Investigating an unusual packet
* Verifying a suspected protocol behavior
* Understanding an application request
* Examining encrypted traffic metadata
* Correlating packets belonging to the same conversation
* Validating an observation before documenting it

Do not inspect every packet manually.

Start broad, identify something interesting, then inspect the relevant packets in detail.

## The Packet Inspection Model

Use this sequence:

```text
1. Identify the packet
2. Establish its context
3. Read the packet details
4. Identify important fields
5. Correlate fields with packet bytes
6. Compare with related packets
7. Extract evidence
8. Decide what to inspect next
```

A packet should rarely be interpreted in isolation.

For example, a TCP `RST` packet is meaningful, but its significance depends on:

* What connection it belongs to
* Which endpoint sent it
* What happened immediately before it
* Whether a handshake completed
* Whether application data was exchanged
* Whether similar resets occurred elsewhere

## The Three Levels of Packet Inspection

Wireshark provides three primary views for understanding a packet.

```text
Packet List
    ↓
Packet Details
    ↓
Packet Bytes
```

Each answers a different question.

| View           | Main purpose                             |
| -------------- | ---------------------------------------- |
| Packet List    | Which packet am I looking at?            |
| Packet Details | What does Wireshark understand about it? |
| Packet Bytes   | What bytes are actually present?         |

Use all three when necessary.

## Packet List

The packet list gives the high-level context.

Common columns include:

* Packet number
* Time
* Source
* Destination
* Protocol
* Length
* Info

The packet list is useful for quickly identifying:

* Communication direction
* Protocol
* Packet ordering
* Repeated behavior
* Errors
* Handshakes
* Requests and responses
* Potential anomalies

Do not treat the `Info` column as the complete meaning of the packet.

It is a summary generated from the packet's decoded contents.

When something matters, inspect the packet details.

## Packet Number

The packet number identifies the packet's position in the capture.

For example:

```text
1
2
3
4
5
...
```

Packet numbers are useful for:

* Referencing evidence
* Returning to a packet
* Comparing nearby packets
* Documenting observations
* Understanding packet sequence within the capture

A packet number is not the same thing as a protocol sequence number.

Keep these concepts separate.

```text
Packet number
    = position in the capture

TCP sequence number
    = position of TCP data within a TCP stream
```

## Time

The time column tells you when Wireshark recorded the packet relative to the selected timestamp display.

Time can help answer:

* When did the event occur?
* How long did a response take?
* Did packets arrive in the expected order?
* Was there a delay between request and response?
* Did multiple events occur within the same time window?

Be careful when interpreting timestamps.

Capture timestamps depend on the capture environment and timestamp configuration.

Do not automatically assume that the timestamp represents a perfectly synchronized real-world clock.

For performance analysis, always compare timestamps between related packets rather than relying only on absolute time.

## Source and Destination

Source and destination identify the direction of communication at the layer being displayed.

For example:

```text
Client → Server
Server → Client
```

At the IP layer, you may see IP addresses.

At other protocol layers, you may encounter:

* MAC addresses
* Hostnames
* Ports
* Protocol-specific identifiers

Always ask:

```text
Which layer am I looking at?
What does this address identify?
```

Do not confuse:

* MAC address
* IP address
* Port
* Hostname
* URL
* Application identity

They represent different things.

## Protocol

The Protocol column provides a high-level indication of what Wireshark has identified.

Examples include:

```text
ARP
IPv4
IPv6
TCP
UDP
DNS
HTTP
TLS
ICMP
DHCP
SSH
```

Protocol identification is useful for orientation, but it is not enough for analysis.

A packet identified as TCP does not tell you:

* Whether it is part of a successful connection
* Whether it contains application data
* Whether it is a retransmission
* Whether the connection was reset
* Whether the server responded correctly

Inspect the decoded fields and surrounding packets.

## Info Column

The Info column provides a concise interpretation of packet contents.

It may show information such as:

```text
SYN
SYN, ACK
ACK
GET /...
200 OK
Client Hello
Server Hello
Standard query
Standard query response
```

Use this as a starting point.

Do not treat it as your final conclusion.

The correct workflow is:

```text
Info column
    ↓
Packet details
    ↓
Relevant fields
    ↓
Related packets
    ↓
Interpretation
```

## Opening the Packet Details

Select a packet in the packet list.

The packet details pane displays the protocols and fields Wireshark has decoded.

You will typically see a hierarchy similar to:

```text
Frame
Ethernet II
Internet Protocol Version 4
Transmission Control Protocol
Application Protocol
```

The exact hierarchy depends on the packet.

Expand the protocol sections relevant to your question.

Do not expand every field automatically.

Instead ask:

```text
What am I trying to determine?
Which protocol layer contains the evidence?
Which fields can answer that question?
```

## Frame Information

The Frame section contains capture-related metadata.

It can provide information about:

* Packet number
* Capture time
* Packet length
* Captured length
* Interface information
* Encapsulation information

Frame information is particularly useful when investigating:

* Capture truncation
* Timestamp behavior
* Packet size
* Capture-interface details
* Differences between captured and original packet size

## Captured Length vs Original Length

A packet can have a captured length that differs from its original length.

Conceptually:

```text
Original packet
        ↓
Capture process
        ↓
Captured portion
```

If the capture does not contain the complete packet, some information may be unavailable.

This matters when:

* Inspecting application payloads
* Looking for headers
* Examining packet bytes
* Reconstructing traffic
* Investigating malformed or incomplete packets

Never conclude that information was absent from the original network traffic solely because it is absent from a truncated capture.

Distinguish:

```text
Not present in packet
```

from:

```text
Not present in captured evidence
```

## Ethernet Inspection

When Ethernet information is available, inspect fields such as:

* Source MAC address
* Destination MAC address
* EtherType
* VLAN-related information when present

Ask:

```text
Who sent the Ethernet frame?
Who received it?
What network-layer protocol follows?
```

Ethernet addresses describe the local link-layer communication.

They should not automatically be treated as the final identity of the communicating application endpoints.

## IP Inspection

For IPv4 or IPv6 traffic, inspect fields relevant to the investigation.

Common useful information includes:

* Source address
* Destination address
* Protocol or next-header information
* Packet length
* TTL or hop-limit
* Fragmentation-related information
* Header-related flags and fields

Useful questions include:

```text
Which host sent this?
Which host received it?
Which transport protocol is inside?
Is fragmentation involved?
Is the packet unusually large or small?
```

Do not attempt to memorize every IP header field.

Learn to identify which fields matter for the question being investigated.

## Transport-Layer Inspection

TCP and UDP are especially important because they describe how application traffic is transported.

For TCP, inspect fields such as:

* Source port
* Destination port
* Flags
* Sequence number
* Acknowledgment number
* Window information
* TCP options
* Payload-related information

For UDP, inspect:

* Source port
* Destination port
* Length
* Checksum
* Application protocol when identified

The question should determine which fields you inspect.

For example:

```text
Question:
Did the TCP connection begin?

Inspect:
SYN
SYN/ACK
ACK
```

Another example:

```text
Question:
Is the receiver acknowledging the sender's data?

Inspect:
Sequence number
Acknowledgment number
TCP flags
```

## Application-Layer Inspection

When Wireshark can decode an application protocol, inspect the fields that answer the investigation question.

Examples include:

```text
DNS:
    Query name
    Query type
    Response
    Status

HTTP:
    Method
    Host
    URI
    Status code
    Headers

TLS:
    Handshake messages
    Versions
    Extensions
    Server name information when visible
    Certificate information when available
```

The goal is not to memorize every application-layer field.

The goal is to identify useful evidence quickly.

## Expanding and Collapsing Protocol Trees

Use the protocol tree deliberately.

A useful approach is:

```text
Start collapsed
    ↓
Identify relevant protocol
    ↓
Expand that section
    ↓
Inspect relevant fields
    ↓
Collapse when finished
```

This keeps complex packets manageable.

For example, when investigating a DNS response, you usually do not need to inspect unrelated lower-level fields first.

Start with:

```text
DNS
```

Then inspect:

```text
Response information
Question
Answer
Additional information
```

depending on what you are trying to determine.

## Selecting a Field

Selecting a field in the packet details pane helps connect the decoded field with the underlying packet data.

This creates an important mental relationship:

```text
Decoded field
      ↕
Raw bytes
```

This is useful when you need to verify exactly what Wireshark is interpreting.

It is also useful for learning unfamiliar protocols.

Instead of memorizing field layouts:

```text
Select field
    ↓
Observe bytes
    ↓
Understand location
    ↓
Relate bytes to decoded meaning
```

## Packet Bytes

The packet bytes pane shows the raw captured packet data.

You will typically see:

* Hexadecimal representation
* ASCII representation when printable characters exist

For example:

```text
Hex:
48 65 6c 6c 6f

ASCII:
Hello
```

The bytes pane is the lowest-level view available directly in the normal packet inspection workflow.

It is useful when:

* A field needs verification
* A protocol is not fully decoded
* You need to inspect raw payload
* You are investigating malformed traffic
* You need to determine whether specific bytes are actually present
* You are learning an unfamiliar protocol

## Decoded Information vs Raw Evidence

Wireshark performs protocol dissection.

This means it interprets raw packet bytes according to known protocol structures.

Think of the relationship as:

```text
Raw bytes
    ↓
Protocol dissection
    ↓
Decoded fields
```

The decoded field is an interpretation of the underlying bytes.

This distinction becomes important when Wireshark cannot fully decode something.

For example:

```text
Known protocol
    ↓
Good dissection
    ↓
Useful fields
```

versus:

```text
Unknown or unsupported data
    ↓
Limited dissection
    ↓
Raw bytes may still be available
```

## Using Field Values as Evidence

When a field matters, record the actual value.

For example:

```text
Source: 192.0.2.10
Destination: 192.0.2.20
Destination port: 443
TCP flags: SYN
```

This is stronger evidence than writing:

```text
There was a connection attempt.
```

The first statement describes observable packet data.

The second is an interpretation.

Keep the distinction clear.

## Observation vs Interpretation

During packet inspection, separate:

### Observation

What Wireshark directly shows.

Example:

```text
A TCP SYN packet was sent from host A to host B.
```

### Interpretation

What the observation suggests.

Example:

```text
Host A appears to be initiating a TCP connection to host B.
```

### Conclusion

What you determine after examining enough evidence.

Example:

```text
The connection attempt did not complete because no SYN/ACK was observed in the relevant traffic.
```

Do not jump directly from one packet to a conclusion.

## Inspecting Related Packets

After inspecting an important packet, inspect the packets around it.

For example:

```text
SYN
↓
SYN/ACK
↓
ACK
↓
Application request
↓
Application response
```

This sequence provides much more context than inspecting only one packet.

Similarly:

```text
DNS query
↓
DNS response
↓
TCP connection
↓
TLS handshake
↓
Application traffic
```

The packet inspection process should follow the communication story.

## Packet-to-Packet Correlation

Useful correlation points include:

* Source and destination
* Ports
* Sequence and acknowledgment numbers
* DNS transaction information
* TCP stream identity
* Request and response relationships
* Timestamps
* Packet numbers
* Protocol-specific identifiers

The purpose of correlation is to determine which packets belong to the same event.

## Using Wireshark's Field Context

When a field is unfamiliar, use the interface itself as a learning tool.

Useful actions may include:

* Selecting the field
* Examining the field's value
* Opening the field's context menu
* Creating a display filter from a field
* Applying or preparing a filter based on the field
* Looking at related packets

This creates a workflow where packet inspection and filtering reinforce each other.

```text
Inspect field
    ↓
Understand field
    ↓
Filter on field
    ↓
Find related packets
    ↓
Inspect again
```

## Inspecting a TCP SYN Packet

Use the following workflow when analyzing a TCP connection attempt.

### Step 1: Identify the packet

Look for a packet that appears to contain:

```text
SYN
```

### Step 2: Inspect addresses

Determine:

```text
Source IP
Destination IP
```

### Step 3: Inspect ports

Determine:

```text
Source port
Destination port
```

### Step 4: Inspect TCP flags

Confirm whether:

```text
SYN = set
ACK = not set
```

for a typical initial SYN.

### Step 5: Inspect sequence information

Record the relevant sequence number if it matters to the investigation.

### Step 6: Inspect TCP options

Look for options that may affect interpretation or troubleshooting.

### Step 7: Inspect nearby packets

Determine whether a response follows.

Do not conclude that a connection failed from the SYN alone.

## Inspecting a DNS Query

When inspecting a DNS query:

```text
1. Identify the DNS packet
2. Identify the client
3. Identify the DNS server
4. Inspect the query name
5. Inspect the query type
6. Record the timestamp
7. Find the corresponding response
```

Then compare:

```text
Query
    ↓
Response
```

Questions to ask:

* Did a response arrive?
* How long did it take?
* What status was returned?
* Did the answer contain the expected information?
* Were multiple queries generated?
* Did the client retry?

## Inspecting an HTTP Request

When HTTP is visible, inspect:

* Source and destination
* TCP stream
* Request method
* Host
* URI
* Relevant headers
* Request timing

Then find the response.

Inspect:

* Status code
* Response timing
* Relevant headers
* Response content when appropriate

Treat captured application data as potentially sensitive.

Only inspect or retain traffic you are authorized to handle.

## Inspecting TLS Traffic

When traffic is encrypted, inspect what remains visible.

Depending on the capture and protocol version, useful information may include:

* Connection endpoints
* Ports
* TLS handshake messages
* Version information
* Extensions
* Server-name information when visible
* Certificate information when available
* Timing
* Connection establishment behavior

Do not assume that encrypted traffic is completely unanalyzable.

At the same time, do not assume Wireshark can reveal encrypted application content without the required decryption material and appropriate conditions.

## Unknown or Undecoded Protocols

Sometimes Wireshark does not identify the application protocol as expected.

Possible reasons include:

* Unsupported protocol
* Non-standard protocol
* Encrypted payload
* Incorrect dissection
* Non-standard port
* Missing context
* Truncated capture
* Encapsulation
* Malformed traffic

Do not immediately assume the traffic is malicious or broken.

First determine what evidence is actually available.

A useful workflow is:

```text
Check packet structure
    ↓
Check transport ports
    ↓
Check payload
    ↓
Check surrounding packets
    ↓
Check whether the protocol may be encrypted
    ↓
Check capture completeness
    ↓
Investigate further
```

## Malformed Packets

Wireshark may report unusual or malformed protocol structures.

Treat these as observations that require context.

Possible explanations include:

* Genuine malformed traffic
* Capture corruption
* Truncated packet
* Unsupported protocol variation
* Dissector limitations
* Incorrect assumptions about encapsulation

Do not immediately convert a malformed indication into a security conclusion.

Verify the raw packet and surrounding traffic.

## Packet Inspection Workflow

Use this repeatable workflow:

```text
Question
    ↓
Find relevant packet
    ↓
Read packet-list context
    ↓
Open packet details
    ↓
Identify relevant protocol layer
    ↓
Inspect relevant fields
    ↓
Correlate with raw bytes when necessary
    ↓
Find related packets
    ↓
Record observations
    ↓
Interpret evidence
    ↓
Decide next action
```

## Practical Exercise 1: Inspect an Ethernet Frame

Use a capture containing Ethernet traffic.

For one packet:

1. Select the packet.
2. Expand the Ethernet section.
3. Identify source MAC address.
4. Identify destination MAC address.
5. Identify the EtherType.
6. Identify the next protocol.
7. Compare the packet-list protocol with the packet-details information.
8. Select one field and observe the corresponding bytes.

Your goal is to understand the relationship between:

```text
Packet list
    ↓
Ethernet fields
    ↓
Raw bytes
```

## Practical Exercise 2: Inspect an IP Packet

Choose an IPv4 or IPv6 packet.

Identify:

* Source address
* Destination address
* Next transport protocol
* Packet length
* TTL or hop limit when applicable

Then answer:

```text
Who sent the packet?
Who received it?
What transport protocol does it carry?
```

Do not move on until you can answer those questions directly from the packet.

## Practical Exercise 3: Inspect a TCP Packet

Choose a TCP packet.

Identify:

* Source port
* Destination port
* TCP flags
* Sequence number
* Acknowledgment number
* Window information
* TCP options when present

Then inspect the packets immediately before and after it.

Answer:

```text
What role does this packet play in the connection?
```

## Practical Exercise 4: Inspect a DNS Transaction

Find a DNS query and its response.

Record:

```text
Client
DNS server
Query name
Query type
Query time
Response time
Response status
Answer information
```

Then determine:

```text
Did the query receive a response?
How long did the response take?
Did the response contain the expected result?
```

## Practical Exercise 5: Inspect Application Traffic

Find an HTTP, TLS, SSH, or another visible application protocol.

Do not try to document every field.

Instead answer:

```text
Who communicated?
What application protocol was involved?
What information is visible?
What information is encrypted or unavailable?
Which packet appears to begin the exchange?
Which packet provides the response?
```

This exercise develops investigative thinking rather than memorization.

## Practical Exercise 6: Raw Bytes Verification

Select a packet with a clearly decoded field.

Then:

1. Select the field.
2. Observe the highlighted bytes.
3. Compare the decoded value with the underlying representation.
4. Move to another field.
5. Repeat.

The objective is not to memorize hexadecimal values.

The objective is to understand:

```text
Wireshark interpretation
        ↕
Actual captured bytes
```

## Practical Exercise 7: Capture Limitation Check

Find a packet where captured length and original length can be compared.

Determine whether the complete packet was captured.

Then answer:

```text
Is missing information actually absent from the traffic,
or is it simply absent from the captured evidence?
```

This distinction is important in professional analysis.

## Packet Inspection Decision Questions

When examining a packet, ask:

### Identity

```text
What packet is this?
```

### Direction

```text
Who sent it?
Who received it?
```

### Timing

```text
When did it occur?
What happened immediately before and after it?
```

### Protocol

```text
Which protocol is involved?
```

### Structure

```text
Which protocol layers are present?
```

### Evidence

```text
Which fields answer my question?
```

### Correlation

```text
Which other packets belong to this event?
```

### Limitation

```text
What information is missing or unavailable?
```

### Interpretation

```text
What does the evidence suggest?
```

### Next action

```text
What should I inspect next?
```

## Common Packet Inspection Mistakes

### Reading only the Info column

The Info column is a summary.

Use packet details for evidence.

### Looking at only one packet

Most network behavior is a sequence.

Inspect related packets.

### Treating decoded text as unquestionable truth

Wireshark's dissection is an interpretation of packet data.

Verify important evidence against the packet structure and raw bytes when necessary.

### Confusing packet number with protocol sequence number

These are different concepts.

### Ignoring direction

The same protocol event can mean different things depending on who sent it.

### Ignoring timing

Timing often explains:

* Delays
* Retransmissions
* Slow responses
* Connection failures

### Assuming encryption means no useful information exists

Metadata and handshake information may still be available.

### Assuming missing data was never transmitted

Capture truncation and capture limitations can remove information from the evidence.

### Treating expert warnings as conclusions

Warnings and anomalies require context.

### Expanding everything

More information does not automatically mean better analysis.

Expand only what helps answer the question.

## Professional Evidence Format

When documenting an important packet, capture enough information to reproduce the observation.

A useful structure is:

```text
Packet:
Protocol:
Time:
Source:
Destination:
Relevant fields:
Related packets:
Observation:
Interpretation:
Limitation:
Next action:
```

Example:

```text
Packet:
TCP packet 142

Protocol:
TCP

Time:
Relevant capture timestamp

Source:
Client host

Destination:
Server host

Relevant fields:
SYN set
Destination port 443

Related packets:
Subsequent packets in the same connection

Observation:
The client sent a TCP SYN toward the server.

Interpretation:
The client appears to be initiating a TCP connection.

Limitation:
A single SYN does not establish whether the connection succeeded.

Next action:
Inspect for a corresponding SYN/ACK and subsequent ACK.
```

This format keeps evidence separate from interpretation.

## What You Should Be Able to Do

After completing this file, you should be able to:

* Navigate from packet list to packet details
* Expand relevant protocol sections
* Identify important fields
* Interpret source and destination correctly
* Distinguish packet numbers from protocol sequence numbers
* Use timestamps for packet correlation
* Inspect Ethernet information
* Inspect IP information
* Inspect TCP and UDP information
* Inspect visible application-layer information
* Understand decoded fields versus raw bytes
* Use packet bytes to verify important fields
* Recognize capture truncation
* Correlate related packets
* Distinguish observations from interpretations
* Recognize when a packet cannot answer the full question
* Decide what packet or field to inspect next

## Completion Criteria

Consider this file complete when you can take an unfamiliar packet and independently answer:

```text
What is this packet?
Who sent it?
Who received it?
When did it occur?
Which protocol layers are present?
Which fields matter?
What do those fields show?
What bytes support the interpretation?
Which related packets should I inspect?
What can I conclude?
What can I not conclude?
What should I investigate next?
```

The goal is not to memorize packet fields.

The goal is to become comfortable moving from:

```text
Packet
→ Fields
→ Bytes
→ Evidence
→ Interpretation
→ Next question
```

## Professional Standard

A capable Wireshark analyst does not simply recognize protocol names.

They can take an unfamiliar packet, determine what it contains, identify the evidence relevant to the current question, correlate it with related traffic, recognize limitations, and decide what to investigate next.

That is the foundation for effective packet analysis.
