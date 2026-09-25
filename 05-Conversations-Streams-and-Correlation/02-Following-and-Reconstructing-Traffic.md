# Following and Reconstructing Traffic

## Objective

This file teaches how to move from an interesting conversation to the actual sequence of traffic exchanged between endpoints.

Endpoint and conversation statistics tell you:

```text
Who communicated?
How much?
Which protocol?
```

Stream and reconstruction analysis helps answer:

```text
What was exchanged?
In what order?
Which side sent it?
How did the conversation progress?
Where did it stop or fail?
```

The core workflow is:

```text
Question
    ↓
Identify endpoint
    ↓
Identify conversation
    ↓
Select relevant packet
    ↓
Follow stream
    ↓
Reconstruct communication
    ↓
Return to packet evidence
    ↓
Correlate timing and transport behavior
    ↓
Interpret
```

The goal is not simply to "read the stream."

The goal is to use reconstruction as evidence while preserving packet-level context.

## What Is a Stream?

A stream represents a logical sequence of traffic belonging to a conversation.

For TCP, the stream follows the bidirectional communication associated with a TCP connection.

Conceptually:

```text
Client
  ↓
TCP Stream
  ↑
Server
```

A stream can contain:

```text
Client → Server
Server → Client
Client → Server
Server → Client
...
```

The exact visible application content depends on the protocol and whether the traffic is encrypted.

## Why Follow a Stream?

Following a stream is useful when individual packets are difficult to interpret in isolation.

For example, an HTTP exchange may span many packets:

```text
Packet 1 → Request headers
Packet 2 → Request body
Packet 3 ← Response headers
Packet 4 ← Response body
Packet 5 ← Response continuation
```

Looking at one packet at a time can make the exchange difficult to understand.

Following the stream can provide the communication sequence in a more coherent form.

## Stream vs Packet

Do not replace packet analysis with stream analysis.

Use both.

### Packet view

Best for:

* timestamps
* TCP flags
* sequence numbers
* acknowledgements
* retransmissions
* packet lengths
* lower-layer behavior
* individual protocol fields

### Stream view

Best for:

* conversation context
* application sequence
* request/response relationships
* readable application content
* reconstructing communication

A strong workflow moves between the two.

## Following a TCP Stream

When an interesting TCP packet is selected, use Wireshark's stream-following functionality.

Conceptually:

```text
Interesting packet
      ↓
Identify TCP conversation
      ↓
Follow TCP Stream
      ↓
Review both directions
      ↓
Identify application exchange
      ↓
Return to packet list
```

The exact menu location can vary slightly by Wireshark version.

The underlying workflow remains the same.

## Stream Direction

A reconstructed stream usually distinguishes traffic direction.

Conceptually:

```text
Client → Server
Server → Client
```

When reading the stream, ask:

```text
Who sent this?
Who received it?
What happened immediately before it?
What happened immediately afterward?
```

Direction is especially important when analyzing:

* authentication
* HTTP
* SMTP
* FTP
* application errors
* protocol negotiation

## Stream Index

Wireshark tracks TCP conversations using stream identifiers.

A TCP stream identifier can be useful for narrowing analysis.

For example, Wireshark commonly exposes a TCP stream field such as:

```text
tcp.stream
```

You can use it to isolate one TCP conversation.

Example:

```text
tcp.stream == 5
```

The number is capture-specific.

Do not assume that stream `5` has the same meaning in another capture.

## Why Stream Numbers Matter

Suppose a capture contains:

```text
Stream 0
Stream 1
Stream 2
...
Stream 127
```

You can isolate one conversation without repeatedly specifying source and destination addresses.

This is particularly useful when:

* many connections exist between the same hosts
* the same ports are reused
* multiple application sessions overlap
* a host communicates with the same server repeatedly

## Stream Filtering Workflow

A practical workflow is:

```text
1. Find interesting packet.
2. Determine its TCP stream.
3. Filter to that stream.
4. Inspect packet sequence.
5. Follow the stream.
6. Return to the filtered packet list.
7. Correlate stream content with packet behavior.
```

This creates a controlled investigation scope.

## Stream Reconstruction Is Not Magic

Following a stream does not automatically tell you:

```text
Why something happened.
```

It helps show:

```text
What was exchanged.
```

You still need to interpret:

* protocol behavior
* response codes
* timing
* transport behavior
* application context

For example:

```text
HTTP 500
```

shows an application-level error response.

It does not automatically explain the backend cause.

## HTTP Stream Reconstruction

HTTP is one of the easiest places to see the value of stream following.

A simplified exchange:

```text
Client → GET /login
Client → Host: example.test

Server → HTTP/1.1 302 Found
Server → Location: /auth

Client → GET /auth

Server → HTTP/1.1 200 OK
```

The reconstructed exchange provides context that individual packets may not.

## HTTP Stream Investigation

When following an HTTP stream, identify:

```text
Request method
Host
URI
Headers
Request body if visible
Response status
Response headers
Response body if visible
Redirects
Cookies if visible
```

Then return to packet-level analysis to determine:

```text
Timing
TCP behavior
Retransmissions
Packet boundaries
Connection termination
```

## HTTP Request Body

A request body may be visible in plaintext HTTP.

Examples include:

* form submissions
* JSON
* XML
* API requests
* file uploads

Visibility depends on the protocol and capture.

Never assume that a request body is always available.

HTTPS normally encrypts application content.

## Sensitive Data

Stream reconstruction can expose sensitive information.

Potentially visible data may include:

* usernames
* passwords
* session tokens
* cookies
* API keys
* personal information
* internal URLs
* application data

Only reconstruct traffic you are authorized to inspect.

In reports, avoid unnecessarily copying sensitive values.

Prefer:

```text
Authentication material was transmitted.
```

instead of reproducing the actual credential when the exact value is not necessary.

## Stream Reconstruction and Encryption

For encrypted traffic:

```text
Follow TCP Stream
```

may show:

```text
Encrypted bytes
```

rather than readable application content.

This is expected.

For TLS:

```text
TCP
 ↓
TLS
 ↓
Encrypted application data
```

Following the TCP stream does not automatically decrypt TLS.

## TLS Stream Analysis

A TLS stream can still help establish:

```text
Client and server
Connection sequence
Handshake timing
Handshake success/failure
Encrypted data direction
Connection termination
```

But you may not see:

```text
HTTP URI
HTTP headers
HTTP response body
Application commands
```

unless the traffic is legitimately decrypted or otherwise visible.

## Stream Following and Protocol Hierarchy

Before following a stream, identify the protocol stack.

For example:

```text
Ethernet
  └── IPv4
       └── TCP
            └── TLS
                 └── HTTP
```

This tells you what kind of reconstruction is possible.

Another example:

```text
Ethernet
  └── IPv4
       └── TCP
            └── SSH
```

The SSH application data will normally be encrypted.

## Following UDP Conversations

UDP does not provide a TCP-style byte stream.

There is:

```text
No TCP connection
No sequence-controlled byte stream
No TCP handshake
```

However, Wireshark can still help follow UDP-based conversations or protocol-specific flows.

For UDP, think:

```text
Datagram
    ↓
Application message
    ↓
Response
```

rather than:

```text
TCP byte stream
```

The exact reconstruction capability depends on the application protocol.

## TCP Stream Reassembly

Large application messages can be split across multiple TCP segments.

Wireshark may reassemble the segments for protocol dissection or stream following.

Conceptually:

```text
TCP Segment 1
TCP Segment 2
TCP Segment 3
       ↓
Reassembled application data
```

This is important because one application message does not necessarily equal one TCP packet.

## TCP Segmentation

Never assume:

```text
One packet = one HTTP request
```

or:

```text
One packet = one application message
```

A single application message may span multiple packets.

Conversely, a TCP packet may contain data associated with more than one logical application operation depending on the protocol and buffering.

Analyze the application layer and transport layer separately.

## Reassembly and Missing Packets

Reconstruction depends on the packets available to Wireshark.

If packets are missing:

```text
Application reconstruction
        ↓
May be incomplete
```

Possible causes include:

* capture loss
* dropped packets
* capture filters
* incorrect capture point
* truncated capture
* unsupported encapsulation
* malformed traffic

Therefore:

```text
Incomplete stream
```

does not automatically mean:

```text
Incomplete application transmission
```

It may mean the capture itself is incomplete.

## Capture Point Matters

Suppose traffic is captured:

```text
Client → Network → Server
```

Different capture locations may show different visibility.

A capture near the client may not reveal:

* server-side traffic
* backend communication
* load-balancer behavior
* internal service-to-service traffic

Therefore, when reconstructing a conversation, record:

```text
Where was the capture taken?
```

This can affect interpretation.

## Following Streams in Troubleshooting

A useful troubleshooting workflow is:

```text
User reports:
"The application failed."
        ↓
Find affected endpoint
        ↓
Find relevant conversation
        ↓
Identify stream
        ↓
Follow stream
        ↓
Understand application exchange
        ↓
Return to packet list
        ↓
Check TCP timing/errors
        ↓
Locate earliest observable failure
```

This connects application-level symptoms to packet-level evidence.

## Example: Successful HTTP Exchange

Consider:

```text
TCP handshake
      ↓
HTTP GET
      ↓
HTTP 200 OK
      ↓
Response data
      ↓
TCP termination
```

The stream helps reconstruct the application interaction.

Packet analysis can then answer:

```text
How long did the request take?
Were packets retransmitted?
Did the connection terminate normally?
```

## Example: HTTP Error

Consider:

```text
TCP handshake
      ↓
HTTP GET /api/data
      ↓
HTTP 500
      ↓
Connection closes
```

The stream shows the application exchange.

The correct conclusion is:

```text
The server returned an HTTP 500 response.
```

Do not automatically conclude:

```text
The database failed.
```

The capture may not contain enough evidence for that.

## Example: TCP Failure Before Application Data

Consider:

```text
SYN
SYN/ACK
ACK
Retransmissions
RST
```

No application request is visible.

This suggests the failure occurred before successful application exchange.

That is different from:

```text
TCP succeeds
HTTP request sent
HTTP 500 returned
```

The stream helps make this distinction clear.

## Stream and Retransmission Correlation

Suppose the application appears slow.

First inspect the stream:

```text
Request
      ↓
Long delay
      ↓
Response
```

Then inspect the packet sequence:

```text
Request segments
      ↓
Retransmission
      ↓
Additional delay
      ↓
Response
```

The retransmission is evidence of transport-level packet loss or another condition causing TCP to retransmit.

It does not automatically prove why the loss occurred.

## Stream and Duplicate ACK Correlation

Similarly:

```text
Application exchange
      ↓
Repeated delay
      ↓
Duplicate ACKs
      ↓
Retransmission
```

This can help explain observed delays.

Always correlate application symptoms with transport evidence.

## Stream and TCP Reset Correlation

A reconstructed application exchange may abruptly stop.

Return to packet-level analysis and check for:

```text
TCP RST
```

Determine:

```text
Who sent the reset?
When?
What was the application doing immediately before it?
```

The side sending the reset may provide important context, but the reset alone does not prove the root cause.

## Stream and Connection Termination

Normal TCP termination generally involves FIN/ACK exchanges.

Example:

```text
Client → FIN
Server → ACK
Server → FIN
Client → ACK
```

An abrupt:

```text
RST
```

is different.

When documenting an application failure, note whether the conversation ended with:

```text
Normal TCP termination
```

or:

```text
Reset
```

## Reconstructing an Application Timeline

A useful technique is to create a timeline.

Example:

```text
10:00:01.100  TCP SYN
10:00:01.120  TCP SYN/ACK
10:00:01.125  TCP ACK
10:00:01.130  HTTP GET /login
10:00:01.600  HTTP 302
10:00:01.610  HTTP GET /auth
10:00:02.100  HTTP 200
10:00:02.150  TCP FIN
```

This provides a much clearer picture than isolated packet descriptions.

## Stream-Based Evidence Record

For an important stream, document:

```text
Stream:
Source:
Destination:
Protocol:
Start time:
End time:
Duration:
Application behavior:
Important requests:
Important responses:
Transport behavior:
Termination:
Limitations:
```

This creates a reusable investigation record.

## Following Streams Across Multiple Conversations

One application transaction can involve multiple connections.

For example:

```text
Browser
 ├── DNS conversation
 ├── TCP/TLS stream A
 ├── TCP/TLS stream B
 └── TCP/TLS stream C
```

Modern applications frequently use:

* connection reuse
* multiple parallel connections
* APIs
* content delivery networks
* third-party services

Do not assume one user action equals one network stream.

## Correlating Multiple Streams

When several streams are involved:

```text
User action
    ↓
DNS
    ↓
Connection A
    ↓
Connection B
    ↓
Connection C
```

Use:

* timestamps
* destination names
* SNI
* IP addresses
* ports
* application context
* request paths when visible

to correlate them.

## HTTP/2 and Stream Concepts

HTTP/2 introduces another level of stream abstraction.

You may have:

```text
TCP connection
    ↓
TLS
    ↓
HTTP/2 connection
    ├── HTTP/2 stream 1
    ├── HTTP/2 stream 3
    ├── HTTP/2 stream 5
    └── HTTP/2 stream 7
```

Do not confuse:

```text
TCP stream
```

with:

```text
HTTP/2 logical stream
```

They are different concepts.

A single TCP connection can carry multiple HTTP/2 streams.

## QUIC and HTTP/3

Modern web applications may use QUIC and HTTP/3.

The model changes:

```text
HTTP/3
  ↓
QUIC
  ↓
UDP
  ↓
IP
```

Instead of:

```text
HTTP
  ↓
TLS
  ↓
TCP
```

QUIC integrates transport and cryptographic mechanisms differently from traditional TCP/TLS.

When HTTP/3 is present, do not assume that a web session will appear as TCP traffic.

## Stream Reconstruction With QUIC

For encrypted QUIC traffic, visibility can include:

* endpoints
* UDP ports
* packet timing
* connection behavior
* visible protocol metadata
* packet sizes
* transport-level events

Application content may remain encrypted.

The same evidence discipline applies:

```text
Observed
≠
Inferred
≠
Proven
```

## Practical Exercise 1: Follow an HTTP Stream

Find an HTTP conversation.

Follow the TCP stream.

Document:

```text
Request
Response
Status
Important headers
Visible body
Connection termination
```

Then return to the packet list.

Identify the packets corresponding to the major stream events.

## Practical Exercise 2: Follow a TLS Stream

Find a TLS connection.

Follow the TCP stream.

Determine:

```text
Can application content be read?
What handshake information is visible?
When does encrypted data begin?
How does the connection terminate?
```

Document what remains unknown because of encryption.

## Practical Exercise 3: Stream Number Investigation

Choose an interesting TCP packet.

Determine:

```text
tcp.stream
```

Filter on that stream.

Then identify:

```text
First packet
TCP handshake
First application data
Major application exchange
Last packet
```

## Practical Exercise 4: HTTP Error Reconstruction

Find an HTTP error.

Follow the relevant stream.

Determine:

```text
Request
Response
Status code
Response headers
Visible response body
Connection behavior
```

Then inspect the packet sequence for timing and TCP behavior.

## Practical Exercise 5: Retransmission and Stream Correlation

Find a TCP stream containing a retransmission.

Determine:

```text
What application exchange was occurring?
Which packet was retransmitted?
How much time passed?
Did the application response arrive afterward?
```

Explain whether the retransmission could plausibly contribute to the observed delay.

## Practical Exercise 6: Reset and Stream Correlation

Find a stream that terminates with a TCP reset.

Determine:

```text
Who sent the RST?
What application data had been exchanged?
How long had the connection existed?
What happened immediately before the reset?
```

Write a conclusion limited to the evidence.

## Practical Exercise 7: Incomplete Stream

Find a stream that appears incomplete.

Determine:

```text
Was the connection actually incomplete?
Or could the capture itself be incomplete?
```

Check for:

* missing packets
* capture boundaries
* truncation
* packet loss indicators
* incomplete handshake
* abrupt capture end

## Practical Exercise 8: Multi-Stream Web Transaction

Select one web activity that uses multiple connections.

Map:

```text
User action
   ├── DNS
   ├── Stream A
   ├── Stream B
   └── Stream C
```

For each stream, record:

```text
Destination
Protocol
Port
Start time
Purpose if identifiable
```

## Practical Exercise 9: Build an Application Timeline

Choose one application transaction.

Create:

```text
Timestamp
Event
Evidence
```

Example:

| Time | Event             | Evidence        |
| ---- | ----------------- | --------------- |
| T1   | TCP established   | SYN/SYN-ACK/ACK |
| T2   | Request sent      | HTTP request    |
| T3   | Delay             | Timestamp gap   |
| T4   | Response received | HTTP response   |
| T5   | Connection closed | FIN/ACK         |

## Practical Exercise 10: Observation vs Interpretation

Choose a reconstructed stream.

Write:

```text
Observation:
The client sent GET /login.

Observation:
The server returned HTTP 401.

Interpretation:
The server rejected the request as unauthorized.

Unknown:
The capture does not establish why the credentials were rejected.
```

This is the expected reasoning style for the rest of the repository.

## Common Mistakes

### Mistake 1: Treating Stream Reconstruction as the Final Answer

A stream shows communication context.

You still need packet-level evidence.

### Mistake 2: Assuming One Packet Equals One Application Message

TCP segmentation and reassembly make this unreliable.

### Mistake 3: Assuming One TCP Stream Equals One User Action

Modern applications may use multiple connections.

### Mistake 4: Forgetting Encryption

Following a TLS stream does not automatically expose application content.

### Mistake 5: Ignoring Missing Packets

An incomplete reconstruction may reflect capture limitations.

### Mistake 6: Ignoring Capture Location

A capture only shows what was visible at its capture point.

### Mistake 7: Confusing TCP Streams With HTTP/2 Streams

They are different layers.

### Mistake 8: Treating a TCP Reset as a Root Cause

A reset is an observed event.

Its cause requires additional evidence.

## Professional Stream Investigation

A strong stream investigation looks like:

```text
Question:
Why did the application request fail?

1. Identify the affected endpoint.
2. Identify the relevant conversation.
3. Identify the TCP stream.
4. Follow the stream.
5. Reconstruct the request/response.
6. Identify the application response.
7. Return to packet-level view.
8. Inspect TCP timing.
9. Check retransmissions.
10. Check resets or abnormal termination.
11. Identify the earliest observable failure.
12. Document limitations.
```

Example conclusion:

```text
The TCP connection was successfully established.
The client sent an HTTP request.
The server returned HTTP 503.
No TCP retransmissions were observed before the response.
The capture therefore shows an application-layer service-unavailable response rather than a failure to establish TCP connectivity.

The capture does not establish the backend cause of the 503 response.
```

This is evidence-based and appropriately scoped.

## Stream Analysis Checklist

### Before Following

* [ ] Identify the endpoint.
* [ ] Identify the conversation.
* [ ] Identify the protocol.
* [ ] Select a relevant packet.
* [ ] Determine the stream identifier where applicable.

### During Reconstruction

* [ ] Identify both directions.
* [ ] Identify application messages.
* [ ] Identify request/response relationships.
* [ ] Note visible content.
* [ ] Note encryption.
* [ ] Note missing or incomplete data.

### After Reconstruction

* [ ] Return to packet-level view.
* [ ] Check timestamps.
* [ ] Check retransmissions.
* [ ] Check duplicate ACKs.
* [ ] Check resets.
* [ ] Check termination.
* [ ] Correlate application and transport behavior.

### Documentation

* [ ] Record source.
* [ ] Record destination.
* [ ] Record protocol.
* [ ] Record stream identifier.
* [ ] Record important events.
* [ ] Separate observations from interpretations.
* [ ] Record limitations.

## Completion Criteria

You are ready to continue when you can independently:

* explain what a TCP stream represents
* distinguish packets from streams
* follow a TCP stream in Wireshark
* identify stream numbers
* reconstruct readable HTTP exchanges
* understand why encrypted streams may not be readable
* correlate stream content with TCP packets
* identify retransmissions affecting a conversation
* investigate resets and termination
* recognize incomplete reconstruction
* account for capture-point limitations
* distinguish TCP streams from HTTP/2 logical streams
* understand the basic implications of HTTP/3 and QUIC
* reconstruct application timelines
* correlate multiple streams belonging to one application activity
* document observations separately from interpretations

The core skill is:

```text
Conversation
    ↓
Stream
    ↓
Reconstruction
    ↓
Packet-level correlation
    ↓
Timeline
    ↓
Evidence-based interpretation
```

The next stage is learning how to correlate these conversations and streams across time, events, and multiple protocols.
