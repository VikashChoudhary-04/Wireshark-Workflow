# Timeline and Correlation

## Objective

This file teaches how to connect separate packets, conversations, streams, protocols, and events into one coherent timeline.

Real network problems rarely exist inside a single packet.

A user may report:

```text
"The application stopped working."
```

The capture may contain:

```text
DNS
↓
TCP
↓
TLS
↓
HTTP
↓
TCP retransmission
↓
HTTP retry
↓
Connection reset
```

The analyst's job is to determine:

```text
What happened?
When did it happen?
In what order?
Which events are related?
What evidence connects them?
What remains unknown?
```

The core workflow is:

```text
Question
    ↓
Identify relevant time window
    ↓
Find initial event
    ↓
Build timeline
    ↓
Correlate conversations
    ↓
Correlate protocols
    ↓
Identify dependencies
    ↓
Find earliest observable failure
    ↓
Interpret
    ↓
Document evidence and limitations
```

## Why Timeline Analysis Matters

Packet captures contain events in time order, but the packet list alone does not always explain the relationship between those events.

For example:

```text
10:01:02  DNS query
10:01:02  DNS response
10:01:03  TCP SYN
10:01:03  TCP SYN/ACK
10:01:03  TCP ACK
10:01:03  TLS ClientHello
10:01:03  TLS ServerHello
10:01:04  HTTP request
10:01:07  TCP retransmission
10:01:08  HTTP response
```

A timeline reveals the sequence.

Without correlation, these may appear to be unrelated packets.

## Event vs Timeline vs Correlation

These concepts are related but different.

### Event

One observable occurrence.

Example:

```text
TCP SYN
```

### Timeline

A chronological sequence of events.

Example:

```text
DNS
→ TCP
→ TLS
→ HTTP
```

### Correlation

Connecting events because there is evidence they belong to the same activity.

Example:

```text
DNS response
        ↓
Resolved IP
        ↓
TCP connection to that IP
        ↓
TLS connection
```

Correlation is what turns a list of events into an investigation.

## The Timeline Mental Model

Use:

```text id="5e4i9k"
Event A
   ↓
Event B
   ↓
Event C
   ↓
Event D
```

Then ask:

```text id="w7sv6b"
Are these events related?
```

If yes:

```text id="w0x0p0"
What evidence connects them?
```

This prevents unsupported assumptions.

## Establish the Time Window

Do not analyze an entire capture blindly.

Start with a relevant time window.

For example:

```text id="e8h7f2"
User reports failure at 10:15.
```

Start around:

```text id="e7qyr7"
10:14:30 → 10:16:00
```

Then expand if necessary.

This reduces noise and improves reasoning.

## Packet Timestamps

Wireshark displays packet timestamps based on the capture's timestamp information and selected time-display settings.

Useful questions include:

```text id="6k5tki"
When did the packet arrive?
How much time passed before the next relevant packet?
How long did the handshake take?
How long between request and response?
```

Timing can reveal:

* delays
* retries
* idle periods
* bursts
* periodic behavior
* connection duration

## Relative vs Absolute Time

Different time displays can answer different questions.

### Absolute time

Useful when correlating with:

* logs
* alerts
* incident timestamps
* user reports
* system events

### Relative time

Useful for understanding:

```text id="h82t6j"
How long after Event A did Event B occur?
```

For example:

```text id="a3f0pd"
T+0.000  SYN
T+0.012  SYN/ACK
T+0.013  ACK
T+0.020  HTTP request
T+1.500  HTTP response
```

This makes latency easier to reason about.

## Time Display and Accuracy

Do not assume packet timestamps are perfect representations of the actual event time.

Potential factors include:

* capture hardware
* capture software
* buffering
* host clock behavior
* virtualization
* capture architecture
* timestamp resolution

For many investigations, relative ordering is more important than extremely precise wall-clock interpretation.

When correlating with external logs, account for possible clock differences.

## Time Delta

A time delta represents the interval between relevant events.

For example:

```text id="t5e1ij"
Request:
10:00:00.100

Response:
10:00:01.600
```

Approximate response delay:

```text id="z8qf39"
1.500 seconds
```

The delay itself is evidence.

The reason for the delay requires further analysis.

## Building a Timeline

A useful timeline format is:

| Time | Event        | Source     | Destination | Protocol | Interpretation             |
| ---- | ------------ | ---------- | ----------- | -------- | -------------------------- |
| T1   | DNS query    | Client     | DNS server  | DNS      | Name resolution started    |
| T2   | DNS response | DNS server | Client      | DNS      | Address returned           |
| T3   | SYN          | Client     | Server      | TCP      | Connection initiated       |
| T4   | SYN/ACK      | Server     | Client      | TCP      | Server responded           |
| T5   | ClientHello  | Client     | Server      | TLS      | TLS negotiation started    |
| T6   | HTTP request | Client     | Server      | HTTP     | Application request sent   |
| T7   | HTTP 503     | Server     | Client      | HTTP     | Application returned error |

The final column should distinguish evidence from assumptions.

## Evidence vs Interpretation

A timeline should not mix facts and conclusions carelessly.

Example:

```text id="nujd3y"
Evidence:
HTTP 503 returned.

Interpretation:
The server reported that the requested service was unavailable.

Unknown:
Why the service was unavailable.
```

This is better than:

```text id="r7e7dk"
The backend server crashed.
```

unless the capture actually contains evidence of a crash.

## Correlating DNS With TCP

One common workflow is:

```text id="i1h9jf"
DNS query
    ↓
DNS response
    ↓
Returned IP
    ↓
TCP connection to returned IP
```

Example:

```text id="9by1x2"
Query:
api.example.test

Response:
10.10.10.50

Then:
Client → 10.10.10.50:443
```

This provides strong correlation.

But be careful:

* DNS can return multiple addresses.
* Cached results may mean no DNS query appears.
* Applications may use hard-coded addresses.
* Hosts files can affect resolution.
* DNS may use encrypted transports.
* Multiple clients may use the same destination.

Do not assume every connection must have a preceding DNS packet in the same capture.

## Correlating TCP With TLS

For HTTPS:

```text id="o5y8qb"
TCP handshake
    ↓
TLS handshake
    ↓
Encrypted application data
```

A timeline can show whether the TLS handshake began only after TCP successfully established.

If the TCP connection fails before TLS:

```text id="wwq7jz"
TCP failure
    ↓
No TLS handshake
```

then the failure occurred before TLS negotiation.

## Correlating TLS With HTTP

When TLS is decrypted or HTTP is otherwise visible:

```text id="3h0m0a"
TLS handshake
    ↓
Encrypted application data
    ↓
HTTP request
    ↓
HTTP response
```

If the TLS handshake fails:

```text id="h5t5g8"
TLS failure
    ↓
No application request
```

This distinguishes a TLS-layer failure from an HTTP-layer failure.

## Correlating HTTP Requests and Responses

A timeline should connect:

```text id="4jzhm9"
Request
    ↓
Response
```

For example:

```text id="1dzg1f"
10:20:01.100  GET /login
10:20:01.250  302 Found
10:20:01.300  GET /auth
10:20:01.800  200 OK
```

This reveals an application sequence rather than isolated packets.

## Correlating Multiple HTTP Requests

Modern applications commonly make many requests.

Example:

```text id="8x6zh3"
GET /index.html
GET /style.css
GET /app.js
GET /api/user
GET /api/data
```

Do not assume every request has equal importance.

Identify:

* the initiating request
* dependencies
* failed requests
* slow requests
* repeated requests
* redirects
* API calls

## User Action Correlation

A user may report:

```text
"Login takes 10 seconds."
```

The capture may show:

```text id="d13qvh"
DNS
→ TCP
→ TLS
→ GET /login
→ POST /authenticate
→ API request
→ delayed response
→ GET /dashboard
```

The important task is to identify which event accounts for the observed delay.

Do not automatically attribute the entire 10 seconds to the network.

## Correlating TCP Timing With Application Timing

Suppose:

```text id="twq5x5"
TCP established quickly.
HTTP request sent.
Response arrives 5 seconds later.
No retransmissions.
```

This provides evidence that:

```text id="5kt0pq"
The delay occurred after the application request was sent and before the response arrived.
```

It does not by itself prove:

```text id="8s91gp"
The server application took exactly 5 seconds to process the request.
```

Other factors may exist between request and response.

## Correlating Retransmissions

Consider:

```text id="rj4qtk"
HTTP request
↓
TCP retransmission
↓
Delay
↓
HTTP response
```

The retransmission provides transport-layer evidence that may explain part of the observed delay.

A professional conclusion might be:

```text id="d6w0ut"
A TCP retransmission occurred during the request/response exchange and preceded the delayed response.
```

This is stronger than simply stating:

```text id="jz6mru"
The network is slow.
```

## Correlating Duplicate ACKs

Duplicate acknowledgements may help explain TCP recovery behavior.

A simplified sequence:

```text id="d1b6nm"
Data
↓
Duplicate ACKs
↓
Retransmission
↓
Additional data
```

Use this to understand the transport behavior surrounding an application delay.

Do not treat duplicate ACKs alone as proof of a particular physical cause.

## Correlating TCP Resets

Example:

```text id="5m70n0"
Application request
↓
TCP RST
```

Questions:

```text id="1j4x1g"
Which endpoint sent the RST?
What happened immediately before it?
Was the request complete?
Was a response sent?
Was the connection newly established?
```

The reset is an event requiring context.

## Correlating TCP Termination

Normal termination may look like:

```text id="rqg47u"
FIN
ACK
FIN
ACK
```

A reset may look like:

```text id="s5avmb"
RST
```

Timeline analysis helps determine whether termination happened:

* after successful application exchange
* during an active transaction
* immediately after connection establishment
* after an error

## Cross-Protocol Correlation

Many investigations require multiple protocols.

Example:

```text id="1i7cvs"
DHCP
 ↓
DNS
 ↓
TCP
 ↓
TLS
 ↓
HTTP
```

Each protocol provides different evidence.

Do not force one protocol to explain the entire incident.

## Example: New Host Cannot Access Website

A client receives an address through DHCP.

Then:

```text id="8o2pqe"
DHCP lease
    ↓
DNS query
    ↓
DNS response
    ↓
TCP connection
    ↓
TLS handshake
    ↓
HTTP request
```

If DNS fails, later layers may never occur.

If DNS succeeds but TCP fails, investigate TCP/network reachability.

If TCP succeeds but TLS fails, investigate TLS.

If TLS succeeds but HTTP returns 503, investigate the application/service layer.

This layered correlation is one of the most important Wireshark skills.

## Example: DNS Failure Correlation

Timeline:

```text id="9v2pqa"
10:00:00  DNS query
10:00:01  DNS timeout
10:00:02  DNS retry
10:00:03  DNS timeout
```

Then:

```text id="pjm4gc"
No TCP connection to the intended service
```

Possible interpretation:

```text id="z8xkqi"
The capture shows repeated DNS resolution attempts without a successful DNS response before the observed application attempt.
```

Do not automatically conclude that the DNS server is down.

Other possibilities require investigation.

## Example: TLS Failure Correlation

Timeline:

```text id="1k2t2c"
TCP SYN
TCP SYN/ACK
TCP ACK
TLS ClientHello
TLS Alert
TCP FIN/RST
```

Interpretation:

```text id="4g5y3w"
TCP connectivity was established before the observed TLS failure.
```

This is more precise than:

```text id="m2k31x"
The server is unreachable.
```

## Example: Application Failure Correlation

Timeline:

```text id="a4y6m0"
DNS succeeds
TCP succeeds
TLS succeeds
HTTP request sent
HTTP 500 returned
```

Interpretation:

```text id="4d2u4n"
The capture shows successful lower-layer connectivity and TLS negotiation followed by an HTTP server error.
```

Unknown:

```text id="u7p2yw"
The backend cause of the HTTP 500.
```

## Correlating Multiple Clients

A capture may contain many hosts.

Suppose:

```text id="j1k4o7"
Client A → Server → succeeds
Client B → Server → succeeds
Client C → Server → fails
```

This comparison can be valuable.

Questions include:

```text id="h9y6qa"
What differs between the clients?
DNS?
Source IP?
Destination?
TCP behavior?
TLS?
Application request?
Timing?
```

Comparative analysis can help isolate the problem.

## Correlating Successful and Failed Sessions

This is one of the strongest troubleshooting techniques.

Example:

```text id="e7g0dw"
Successful:
DNS → TCP → TLS → HTTP 200

Failed:
DNS → TCP → TLS → HTTP 503
```

The common path is less likely to explain the difference.

Focus on the divergence.

This is not proof of causation, but it is a useful way to narrow investigation.

## Correlation by Destination

Multiple sessions may target:

```text id="a7x2wy"
Same destination IP
```

or:

```text id="v5d8x4"
Same hostname
```

Group them carefully.

A hostname can resolve to multiple IPs.

A single IP can host multiple applications.

Correlation should consider:

```text id="v6x5x1"
IP
Port
Protocol
Time
Application metadata
```

## Correlation by Source

Source-centric investigation is useful when analyzing:

* infected-host hypotheses
* endpoint troubleshooting
* unusual traffic
* authentication failures
* scanning
* repeated connections

Workflow:

```text id="0j3h4c"
Source host
    ↓
All conversations
    ↓
Time window
    ↓
Interesting destinations
    ↓
Protocol
    ↓
Stream
    ↓
Timeline
```

## Correlation by Time Window

Suppose an alert occurs at:

```text id="qv84c2"
14:32:10
```

Search around the event.

Start narrow:

```text id="7jv09j"
14:32:00 → 14:32:20
```

Then expand if needed.

This is generally more effective than examining the entire capture.

## Periodic Communication

Repeated communication at regular intervals can reveal:

```text id="v1g9p7"
Polling
Keepalives
Monitoring
Scheduled tasks
Telemetry
Application refresh
```

It can also be an investigation signal in security analysis.

Do not classify periodic traffic as malicious solely because it is periodic.

Measure:

```text id="4w6xq7"
Interval
Destination
Protocol
Volume
Payload visibility
Context
```

## Burst Analysis

Traffic can occur in bursts.

Example:

```text id="8qv3m4"
Quiet
↓
Large burst
↓
Quiet
↓
Large burst
```

Possible explanations include:

* downloads
* uploads
* backups
* application polling
* batch processing
* synchronization

Use timing and protocol context to understand the pattern.

## Correlating External Logs

Wireshark analysis may be stronger when correlated with authorized external evidence such as:

* application logs
* firewall logs
* DNS logs
* authentication logs
* server logs
* endpoint alerts

The workflow is:

```text id="aq1hzi"
Wireshark event
      ↓
Timestamp
      ↓
External log
      ↓
Matching event
      ↓
Cross-validation
```

Be aware of clock differences.

A timestamp match is stronger when:

* clocks are synchronized
* identifiers match
* source/destination match
* timing is plausible

## Timestamp Drift

If systems have different clocks:

```text id="u4y8c7"
Wireshark: 10:00:05
Server log: 10:00:08
```

do not immediately assume they are unrelated.

Potential explanations include:

* clock offset
* timezone differences
* logging delay
* buffering
* timestamp interpretation

Use multiple correlation points when possible.

## Correlation Confidence

Not every correlation has the same strength.

### Strong correlation

```text id="7a9l6v"
Same source
Same destination
Same port
Same protocol
Very close timestamp
Matching application context
```

### Moderate correlation

```text id="6w0x1e"
Same source
Same destination
Similar time
Related protocol
```

### Weak correlation

```text id="f7h2cw"
Same destination only
```

Do not present weak correlation as confirmed causation.

## Causation vs Correlation

This distinction is critical.

Suppose:

```text id="c8l1e9"
Retransmission occurs
Application response is delayed
```

You can say:

```text id="y1t7by"
The retransmission occurred during the delayed exchange.
```

You should be cautious about saying:

```text id="e6l3uo"
The retransmission caused the application delay.
```

unless the evidence supports that causal relationship.

Wireshark frequently shows correlation more clearly than causation.

## Timeline Construction Workflow

Use this repeatable process:

```text id="q6m6ez"
1. Define the question.
2. Define the affected host or transaction.
3. Define the time window.
4. Identify the first relevant event.
5. Identify dependent events.
6. Correlate source and destination.
7. Correlate protocols.
8. Record timestamps.
9. Identify delays.
10. Identify retries or failures.
11. Find the earliest observable failure.
12. Separate evidence from interpretation.
13. Document unknowns.
```

## Timeline Evidence Table

For important investigations, use:

| Time | Source | Destination | Protocol | Event       | Evidence        | Interpretation                  |
| ---- | ------ | ----------- | -------- | ----------- | --------------- | ------------------------------- |
| T1   | Client | DNS         | DNS      | Query       | DNS request     | Resolution started              |
| T2   | DNS    | Client      | DNS      | Response    | A/AAAA response | Address returned                |
| T3   | Client | Server      | TCP      | SYN         | TCP SYN         | Connection initiated            |
| T4   | Server | Client      | TCP      | SYN/ACK     | TCP response    | Server reachable at TCP layer   |
| T5   | Client | Server      | TLS      | ClientHello | TLS handshake   | TLS negotiation started         |
| T6   | Client | Server      | HTTP     | Request     | HTTP request    | Application transaction started |
| T7   | Server | Client      | HTTP     | 503         | HTTP response   | Service unavailable response    |

This structure encourages disciplined reasoning.

## Practical Exercise 1: Build a Complete Timeline

Choose one application transaction.

Trace:

```text id="m6yql3"
DNS
→ TCP
→ TLS
→ HTTP
```

Create a timestamped timeline.

For every event, record:

```text id="k0jynm"
Time
Protocol
Source
Destination
Event
```

## Practical Exercise 2: Find the Earliest Failure

Choose a failed transaction.

Build the timeline.

Then ask:

```text id="d0l3re"
What is the first observable event that differs from a successful transaction?
```

This is often more useful than focusing on the final error.

## Practical Exercise 3: Compare Successful and Failed Sessions

Find:

```text id="2q5xwx"
One successful session
One failed session
```

Compare:

```text id="1zqf37"
DNS
TCP
TLS
HTTP
Timing
Retransmissions
Termination
```

Identify where their behavior diverges.

## Practical Exercise 4: DNS-to-Application Correlation

Find a hostname resolution followed by application traffic.

Document:

```text id="6ydt7e"
DNS query
DNS response
Returned address
Subsequent destination
Application protocol
Timing
```

Determine whether the destination matches the DNS evidence.

## Practical Exercise 5: TCP-to-Application Correlation

Find an application delay.

Determine:

```text id="jz6m0a"
TCP establishment
Application request
Response
Retransmissions
Duplicate ACKs
Response timing
```

Explain the relationship without overclaiming causation.

## Practical Exercise 6: TLS-to-Application Correlation

Find a TLS-protected application session.

Determine:

```text id="8a5z2e"
TCP establishment
TLS handshake start
TLS handshake completion
Encrypted application data
Connection termination
```

If application content is unavailable, document the visibility limitation.

## Practical Exercise 7: Multi-Client Comparison

Find two clients communicating with the same service.

Compare:

```text id="s8xx9k"
DNS
TCP
TLS
Application response
Timing
```

Determine where the sessions differ.

## Practical Exercise 8: Periodic Traffic

Find a repeated communication pattern.

Measure:

```text id="2w8b5j"
Destination
Protocol
Approximate interval
Packet count
Byte count
```

Determine possible explanations and state which are supported by evidence.

## Practical Exercise 9: Incident Timeline

Create a timeline for a hypothetical incident:

```text id="n7kz8a"
User reports failure
    ↓
Network traffic begins
    ↓
Protocol events occur
    ↓
Failure appears
    ↓
Connection ends
```

Write:

```text id="k3s0yh"
Observed facts
Interpretation
Unknowns
```

## Practical Exercise 10: Evidence-Based Conclusion

Take your completed timeline and write a conclusion using this structure:

```text id="49s1yc"
The capture shows:

1. ...
2. ...
3. ...

The earliest observable failure was:

...

The evidence supports:

...

The capture does not establish:

...
```

This format should become natural before moving into advanced statistics and anomaly analysis.

## Common Mistakes

### Mistake 1: Looking Only at the Final Error

The final error may be a consequence rather than the original failure.

### Mistake 2: Ignoring Time

Order and delay often provide essential evidence.

### Mistake 3: Correlating Only by IP

Shared servers and reused addresses make IP-only correlation weak.

### Mistake 4: Ignoring Ports

Ports can distinguish multiple application relationships between the same hosts.

### Mistake 5: Ignoring Protocol Layers

DNS, TCP, TLS, and HTTP can each fail independently.

### Mistake 6: Treating Correlation as Causation

Two events occurring together does not automatically prove one caused the other.

### Mistake 7: Ignoring Capture Limitations

A missing packet can change the apparent sequence.

### Mistake 8: Treating Timestamps as Perfect

Capture and system timing can introduce offsets or uncertainty.

### Mistake 9: Assuming Every Application Event Has a Network Packet

Applications may cache, retry internally, or perform local processing.

### Mistake 10: Over-Interpreting Encrypted Traffic

Visible metadata does not equal visible application content.

## Professional Correlation Pattern

A strong investigation can be summarized as:

```text id="c2d3sl"
Question
    ↓
Time window
    ↓
Affected host
    ↓
Endpoint
    ↓
Conversation
    ↓
Stream
    ↓
Protocol sequence
    ↓
Timestamp correlation
    ↓
Transport/application correlation
    ↓
Earliest observable failure
    ↓
Evidence-based conclusion
```

Example:

```text id="xg2nq4"
Question:
Why did the user experience a failed web request?

Timeline:
DNS resolution succeeded.
TCP connection established.
TLS handshake completed.
HTTP request was sent.
TCP retransmission occurred.
HTTP response arrived after a delay.
Server returned HTTP 503.

Interpretation:
The capture shows successful DNS, TCP, and TLS establishment before the delayed HTTP response.

The TCP retransmission occurred during the delayed exchange.

Conclusion:
The earliest visible application failure is the HTTP 503 response.

Limitation:
The capture does not establish why the server generated the 503 response or whether the retransmission was the primary cause of the delay.
```

This is the standard of reasoning expected throughout the repository.

## Professional Correlation Checklist

### Scope

* [ ] Define the question.
* [ ] Identify affected host.
* [ ] Define time window.
* [ ] Identify relevant protocols.

### Timeline

* [ ] Record timestamps.
* [ ] Identify first relevant event.
* [ ] Identify dependent events.
* [ ] Record delays.
* [ ] Record retries.
* [ ] Record failures.
* [ ] Record termination.

### Correlation

* [ ] Correlate DNS with destination.
* [ ] Correlate TCP with application traffic.
* [ ] Correlate TLS with application traffic.
* [ ] Correlate requests with responses.
* [ ] Correlate multiple streams.
* [ ] Compare successful and failed sessions where possible.

### Reasoning

* [ ] Separate observation from interpretation.
* [ ] Distinguish correlation from causation.
* [ ] Identify earliest observable failure.
* [ ] Avoid unsupported root-cause claims.
* [ ] Document unknowns.
* [ ] Document capture limitations.

## Completion Criteria

You are ready to continue when you can independently:

* build a packet timeline
* work with absolute and relative timestamps
* calculate useful time differences
* correlate DNS with subsequent connections
* correlate TCP establishment with application activity
* correlate TLS with encrypted application traffic
* correlate HTTP requests and responses
* correlate retransmissions with application events
* investigate resets and termination
* compare successful and failed sessions
* correlate multiple streams belonging to one user action
* distinguish correlation from causation
* identify the earliest observable failure
* document uncertainty and capture limitations
* produce an evidence-based investigation timeline

The core skill is:

```text id="zy2g3x"
Packets
    ↓
Conversations
    ↓
Streams
    ↓
Timeline
    ↓
Cross-protocol correlation
    ↓
Earliest observable failure
    ↓
Evidence-based conclusion
```

This completes the **Conversations, Streams, and Correlation** section and prepares you for the next stage: using Wireshark statistics, timing analysis, I/O graphs, and expert information to identify performance problems and anomalies.
