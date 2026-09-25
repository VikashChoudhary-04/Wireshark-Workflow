# Wireshark Filter Workflows

## Objective

This file turns display-filter construction into complete investigation workflows.

The goal is no longer simply:

```text
"Can I write this filter?"
```

The goal is:

```text
"Can I use filters to answer an investigation question?"
```

A professional filtering workflow follows:

```text
Question
    ↓
Initial hypothesis
    ↓
Broad filter
    ↓
Observe
    ↓
Narrow filter
    ↓
Correlate
    ↓
Test alternatives
    ↓
Validate
    ↓
Interpret
    ↓
Next question
```

A filter is therefore not the investigation itself.

It is an instrument used to reduce the search space and expose evidence.

---

## The Core Workflow

For every investigation, use:

```text
Question
→ Evidence
→ Filter
→ Observation
→ Interpretation
→ Decision
→ Next Question
```

Example:

```text
Question:
Why can this client not reach the server?

Evidence:
TCP connection establishment

Initial filter:
ip.addr == 192.168.1.10 && ip.addr == 192.168.1.20

Observation:
SYN packets are visible.

Next question:
Is a SYN-ACK returning?

Next filter:
tcp.flags.syn == 1 && tcp.flags.ack == 1
```

The important part is that each filter should answer a specific question.

---

## Broad-to-Narrow Analysis

Start broad enough to understand the traffic.

Then narrow only when you have a reason.

A typical progression is:

```text
All packets
    ↓
Protocol
    ↓
Host
    ↓
Direction
    ↓
Service
    ↓
Specific event
    ↓
Specific field/value
```

For example:

```text
dns
```

then:

```text
dns && ip.src == 192.168.1.10
```

then:

```text
dns && ip.src == 192.168.1.10 && dns.qry.name == "example.com"
```

This progression makes it easier to identify which assumption changed the result.

---

## Workflow 1 — Find Traffic from a Host

### Question

```text
What traffic did 192.168.1.10 generate?
```

### Step 1 — Filter the Source

```text
ip.src == 192.168.1.10
```

### Step 2 — Observe

Inspect:

```text
Protocols
Destinations
Ports
Packet volume
Timing
```

Do not immediately decide what the traffic means.

First determine what is actually present.

### Step 3 — Separate Protocols

If TCP dominates:

```text
ip.src == 192.168.1.10 && tcp
```

If DNS is relevant:

```text
ip.src == 192.168.1.10 && dns
```

### Step 4 — Continue from the Evidence

Ask:

```text
Which destinations?
Which services?
Which connections?
Which events?
```

The next filter should be determined by what you observe.

---

## Workflow 2 — Find Traffic to a Host

### Question

```text
What traffic was sent to 192.168.1.20?
```

Start with:

```text
ip.dst == 192.168.1.20
```

Then examine:

```text
Source hosts
Protocols
Ports
Timing
Connection patterns
```

If you are investigating web traffic:

```text
ip.dst == 192.168.1.20 && tcp.port == 443
```

Remember that this shows TCP port 443 traffic, not proof of a particular application protocol.

---

## Workflow 3 — Investigate a Single Conversation

### Question

```text
What happened between client 192.168.1.10 and server 192.168.1.20?
```

Start with:

```text
ip.addr == 192.168.1.10 && ip.addr == 192.168.1.20
```

This identifies packets involving both hosts.

Then narrow by protocol if necessary:

```text
ip.addr == 192.168.1.10 && ip.addr == 192.168.1.20 && tcp
```

Then inspect:

```text
Source port
Destination port
TCP flags
Sequence
Acknowledgment
Timing
Retransmissions
Application protocol
```

The filter identifies the conversation.

The packet sequence explains what happened.

---

## Workflow 4 — Investigate a Specific Service

### Question

```text
Which traffic targeted TCP port 443?
```

Start with:

```text
tcp.dstport == 443
```

Then examine:

```text
Source hosts
Destination hosts
Connection counts
TCP handshakes
TLS handshakes
Failures
Timing
```

If the question changes to:

```text
Which traffic involved port 443 in either direction?
```

Use:

```text
tcp.port == 443
```

Understand the distinction:

```text
tcp.dstport == 443
```

is directional.

```text
tcp.port == 443
```

asks whether port 443 is involved.

---

## Workflow 5 — Investigate TCP Connection Establishment

### Question

```text
Are clients successfully establishing TCP connections?
```

Start broadly:

```text
tcp
```

Then investigate SYN packets:

```text
tcp.flags.syn == 1
```

Look for SYN-ACK responses:

```text
tcp.flags.syn == 1 && tcp.flags.ack == 1
```

Then investigate resets:

```text
tcp.flags.reset == 1
```

And connection termination:

```text
tcp.flags.fin == 1
```

Do not treat these filters independently.

Build a sequence:

```text
SYN
 ↓
SYN-ACK
 ↓
ACK
 ↓
Application traffic
 ↓
FIN/ACK or RST
```

Then determine where the expected sequence breaks.

---

## Workflow 6 — Investigate Failed TCP Connections

### Question

```text
Why is a TCP connection failing?
```

Start with the relevant hosts:

```text
ip.addr == 192.168.1.10 && ip.addr == 192.168.1.20 && tcp
```

Inspect the beginning of the connection.

Ask:

```text
Is a SYN sent?
Is a SYN-ACK returned?
Is there an ACK?
Does the connection retry?
Is there a reset?
```

Useful filters include:

```text
tcp.flags.syn == 1
```

```text
tcp.flags.syn == 1 && tcp.flags.ack == 1
```

```text
tcp.flags.reset == 1
```

Then examine packet timing and direction.

The filter narrows the evidence.

The sequence establishes the diagnosis.

---

## Workflow 7 — Investigate TCP Retransmissions

### Question

```text
Are packets being retransmitted?
```

Begin with the relevant connection:

```text
ip.addr == 192.168.1.10 && ip.addr == 192.168.1.20 && tcp
```

Then use Wireshark's TCP analysis information to identify retransmissions.

A display filter can be used to isolate retransmission-related analysis when the corresponding field is present.

The important workflow is:

```text
Connection
    ↓
TCP analysis
    ↓
Retransmission
    ↓
Packet timing
    ↓
Direction
    ↓
Possible cause
```

Do not automatically conclude that every retransmission proves packet loss.

Consider:

```text
Capture location
Timestamp accuracy
Duplicate packets
Out-of-order delivery
Network conditions
Capture artifacts
```

---

## Workflow 8 — Investigate Zero-Window Conditions

### Question

```text
Is a TCP receiver unable to accept more data?
```

Start with:

```text
tcp
```

Then inspect TCP window-related fields.

A zero advertised window can be identified with an appropriate field filter such as:

```text
tcp.window_size_value == 0
```

Then determine:

```text
Which host advertised it?
Which connection?
When did it occur?
How long did it persist?
What was the application doing?
```

A zero window is an observation.

It is not by itself a complete explanation for application slowness.

---

## Workflow 9 — Investigate DNS Activity

### Question

```text
What DNS activity did a client generate?
```

Start:

```text
dns
```

Then:

```text
dns && ip.src == 192.168.1.10
```

Separate queries:

```text
dns.flags.response == 0
```

And responses:

```text
dns.flags.response == 1
```

Then investigate query names:

```text
dns.qry.name
```

If you identify a specific domain:

```text
dns.qry.name == "example.com"
```

Continue by examining:

```text
Query frequency
Response status
Response addresses
Repeated queries
Query timing
Source host
Destination DNS server
```

---

## Workflow 10 — Investigate DNS Failures

### Question

```text
Why is name resolution failing?
```

Start:

```text
dns
```

Identify the client:

```text
dns && ip.src == 192.168.1.10
```

Separate queries from responses:

```text
dns.flags.response == 0
```

Then examine the corresponding responses.

Look for:

```text
Response presence
Response timing
Response code
Returned records
Repeated queries
Different DNS servers
```

The investigation becomes:

```text
Client sends query
        ↓
Does a response return?
        ↓
How quickly?
        ↓
What response code?
        ↓
What answer?
        ↓
Does the client retry?
```

---

## Workflow 11 — Investigate Repeated DNS Queries

### Question

```text
Is a client repeatedly requesting the same name?
```

Start:

```text
dns && ip.src == 192.168.1.10
```

Then inspect:

```text
dns.qry.name
```

Identify repeated names.

Then narrow:

```text
dns && ip.src == 192.168.1.10 && dns.qry.name == "example.com"
```

Next investigate:

```text
Timing
Number of queries
Responses
Response codes
Intervals
```

Repeated DNS requests can have many legitimate explanations.

The packet pattern must be interpreted in context.

---

## Workflow 12 — Investigate HTTP Requests

### Question

```text
Which HTTP requests were made?
```

Start:

```text
http
```

Then examine:

```text
http.request.method
http.host
http.request.uri
http.response.code
```

For GET requests:

```text
http.request.method == "GET"
```

For POST requests:

```text
http.request.method == "POST"
```

For a particular host:

```text
http.host == "example.com"
```

Combine conditions when necessary:

```text
http.host == "example.com" && http.request.method == "POST"
```

Then correlate requests with responses.

---

## Workflow 13 — Investigate HTTP Errors

### Question

```text
Are HTTP requests receiving errors?
```

Start:

```text
http.response.code
```

Then examine response codes.

You may narrow to an exact status code when the investigation requires it.

For example:

```text
http.response.code == 404
```

or:

```text
http.response.code == 500
```

Then ask:

```text
Which client?
Which host?
Which URI?
When?
How often?
Were there retries?
```

Do not interpret an HTTP status code without examining the request and surrounding traffic.

---

## Workflow 14 — Investigate TLS Handshakes

### Question

```text
Is TLS negotiation occurring successfully?
```

Start:

```text
tls
```

Then identify handshake packets.

Inspect:

```text
ClientHello
ServerHello
Certificate
Alert
Version
Cipher information
Server Name Indication when present
```

The workflow is:

```text
TLS traffic
    ↓
Handshake
    ↓
ClientHello
    ↓
Server response
    ↓
Certificate / negotiation
    ↓
Encrypted application traffic
```

If the handshake stops unexpectedly, investigate:

```text
Timing
Direction
TLS alerts
TCP resets
TCP retransmissions
Connection termination
```

---

## Workflow 15 — Investigate Encrypted Traffic

### Question

```text
What can I determine about encrypted communication?
```

Start:

```text
tls
```

Then examine visible metadata.

Depending on the capture and protocol, you may still determine:

```text
Endpoints
Ports
Timing
Handshake behavior
TLS versions
Cipher information
Certificates
SNI when present
Packet sizes
Connection duration
TCP behavior
```

You may not be able to inspect application contents.

Separate:

```text
What the capture proves
```

from:

```text
What the capture does not expose
```

This distinction is essential in forensic analysis.

---

## Workflow 16 — Investigate a Suspicious Host

### Question

```text
What network activity did this host perform?
```

Start:

```text
ip.addr == 192.168.1.10
```

Then examine:

```text
DNS
HTTP
TLS
SSH
SMB
LDAP
Kerberos
Other protocols
```

Break the investigation into protocol-specific questions.

For DNS:

```text
dns && ip.addr == 192.168.1.10
```

For TCP:

```text
tcp && ip.addr == 192.168.1.10
```

For HTTP:

```text
http && ip.addr == 192.168.1.10
```

For TLS:

```text
tls && ip.addr == 192.168.1.10
```

Then build a timeline.

Do not label the host malicious merely because it communicates with an unusual destination.

---

## Workflow 17 — Investigate Potential Scanning

### Question

```text
Does the capture contain a pattern consistent with port scanning?
```

Start by identifying the source host:

```text
ip.src == 192.168.1.10 && tcp
```

Then examine:

```text
Destination addresses
Destination ports
SYN patterns
Connection responses
Timing
```

A useful investigative sequence is:

```text
Source host
    ↓
Many destinations?
    ↓
Many destination ports?
    ↓
Repeated SYN attempts?
    ↓
What responses occur?
    ↓
What is the time distribution?
```

A large number of SYN packets is an observation.

Whether the pattern represents scanning requires contextual analysis.

---

## Workflow 18 — Investigate Potential Lateral Communication

### Question

```text
Is one internal host communicating with many other internal systems?
```

Start with the suspected source:

```text
ip.src == 192.168.1.10
```

Then inspect:

```text
Destination addresses
Destination ports
Protocol distribution
Connection timing
Connection frequency
```

Narrow to protocols of interest:

```text
ip.src == 192.168.1.10 && smb
```

or:

```text
ip.src == 192.168.1.10 && ldap
```

or:

```text
ip.src == 192.168.1.10 && kerberos
```

The purpose is to identify communication patterns.

The capture alone may not establish intent.

---

## Workflow 19 — Investigate Packet Timing

### Question

```text
Where is time being spent in a network exchange?
```

Start with the relevant conversation:

```text
ip.addr == 192.168.1.10 && ip.addr == 192.168.1.20
```

Then identify:

```text
Request
Response
Delay
Retransmission
Processing gap
Connection setup time
```

Use packet timestamps and timing tools to correlate events.

The workflow is:

```text
Request
    ↓
Timestamp
    ↓
Response
    ↓
Timestamp
    ↓
Difference
    ↓
Interpretation
```

Do not confuse network delay with application processing time without evidence.

---

## Workflow 20 — Investigate Application Slowness

### Question

```text
Why does an application appear slow?
```

Do not start by assuming the network is responsible.

Break the problem into layers:

```text
Client
    ↓
DNS
    ↓
TCP connection
    ↓
TLS negotiation
    ↓
Application request
    ↓
Server response
    ↓
Data transfer
```

Use filters to isolate each stage.

For example:

```text
dns
```

then:

```text
tcp
```

then:

```text
tls
```

then:

```text
http
```

or the appropriate application protocol.

Ask at each stage:

```text
Is the delay here?
```

This prevents premature conclusions.

---

## Workflow 21 — Investigate a Known Incident Window

### Question

```text
What happened between 10:30 and 10:35?
```

Start by locating the time window.

Then:

```text
Reduce the capture to the relevant period
    ↓
Identify dominant protocols
    ↓
Identify active hosts
    ↓
Identify unusual events
    ↓
Build focused filters
```

Do not begin with a highly specific filter if you do not yet know what happened.

The first objective is situational awareness.

---

## Workflow 22 — Investigate a Known Packet

Sometimes you already have an important packet number.

Example:

```text
frame.number == 150
```

Inspect:

```text
Packet details
Conversation
Previous packet
Next packet
Protocol
Addresses
Ports
Timing
```

Then expand outward.

Use nearby packets to understand the event.

A packet should rarely be interpreted in complete isolation.

---

## Workflow 23 — Work Backward from an Error

Suppose you identify:

```text
HTTP 500
```

Do not stop there.

Ask:

```text
What request caused it?
Which client sent the request?
Which server responded?
When did it occur?
Were previous requests successful?
Did the same URI fail repeatedly?
Was the TCP connection healthy?
```

Filtering becomes a chain:

```text
Error
    ↓
Response
    ↓
Request
    ↓
Conversation
    ↓
Client
    ↓
Earlier events
```

This is a powerful general investigation pattern.

---

## Workflow 24 — Work Forward from an Initial Event

The reverse approach is also useful.

Suppose you identify:

```text
SYN
```

Ask:

```text
Did SYN-ACK return?
Did ACK follow?
Did application data appear?
Did TLS begin?
Did the server reset?
Did the connection retransmit?
```

The investigation becomes:

```text
Initial event
    ↓
Immediate response
    ↓
Next protocol stage
    ↓
Outcome
```

This is especially useful for troubleshooting.

---

## Workflow 25 — Compare Two Hosts

### Question

```text
Do two clients show the same network behavior?
```

Build separate filters.

Host A:

```text
ip.addr == 192.168.1.10
```

Host B:

```text
ip.addr == 192.168.1.11
```

Compare:

```text
DNS
TCP setup
TLS
HTTP
Destination
Timing
Retransmissions
Errors
```

This can help distinguish:

```text
Client-specific problem
```

from:

```text
Service-wide behavior
```

The comparison should be based on observed packet evidence.

---

## Workflow 26 — Filter by Multiple Alternatives

Sometimes the investigation concerns several related protocols or services.

For example:

```text
tcp.port == 80 || tcp.port == 443
```

Then examine:

```text
Which hosts?
Which direction?
Which protocol dissections?
Which timing?
```

Do not combine alternatives simply because they are technically related.

Every condition should serve the investigation question.

---

## Workflow 27 — Avoid Confirmation Bias

Filtering can accidentally create confirmation bias.

Suppose you believe:

```text
"This host is scanning."
```

You might immediately filter:

```text
ip.src == 192.168.1.10 && tcp.flags.syn == 1
```

and focus only on packets supporting the hypothesis.

Instead ask:

```text
What evidence would support the hypothesis?
What evidence would contradict it?
What legitimate explanation could produce the same pattern?
```

Then examine:

```text
SYNs
Responses
Destination distribution
Timing
Successful connections
Application context
```

Filtering should test hypotheses, not merely confirm them.

---

## Workflow 28 — Use Negative Evidence Carefully

Suppose:

```text
tcp.flags.syn == 1
```

returns packets.

That tells you SYN packets exist.

Suppose you do not see SYN-ACK packets in the expected direction.

That is potentially useful evidence.

But ask:

```text
Is the capture point capable of seeing both directions?
Is the capture complete?
Is the traffic asymmetric?
Was the response outside the capture window?
Was another path involved?
```

Absence of evidence is not automatically evidence of absence.

---

## Workflow 29 — When a Filter Produces Too Much Data

If this:

```text
tcp
```

returns thousands of packets, narrow based on the question.

For example:

```text
tcp && ip.addr == 192.168.1.10
```

Then:

```text
tcp && ip.addr == 192.168.1.10 && tcp.port == 443
```

Then:

```text
tcp && ip.addr == 192.168.1.10 && tcp.port == 443 && tcp.flags.syn == 1
```

Each reduction should have a reason.

Do not narrow simply to make the packet count smaller.

---

## Workflow 30 — When a Filter Produces Nothing

If:

```text
http
```

returns nothing, ask:

```text
Is the traffic actually HTTP?
Is it encrypted?
Is it another protocol?
Is the capture after the HTTP exchange?
Is the packet dissection correct?
```

If:

```text
tcp.dstport == 443
```

returns nothing, ask:

```text
Is the service using another port?
Is the traffic UDP/QUIC instead?
Is the capture incomplete?
```

Zero results should trigger investigation, not random filter changes.

---

## Filter Workflows and Wireshark Statistics

Filtering and statistics should work together.

For example:

```text
Filter:
dns
```

Then inspect statistics related to:

```text
Endpoints
Conversations
Protocol hierarchy
Packet counts
```

Statistics can reveal patterns that are difficult to see from the packet list alone.

Then return to filtering.

The workflow becomes:

```text
Filter
    ↓
Statistics
    ↓
Pattern
    ↓
Focused filter
    ↓
Packet inspection
```

---

## Filter Workflows and Conversations

Once you identify an important host pair, move from broad filtering toward conversation analysis.

Example:

```text
ip.addr == 192.168.1.10 && ip.addr == 192.168.1.20
```

Then inspect:

```text
Conversation
Stream
Ports
Sequence
Timing
Application protocol
```

A filter identifies the relevant packet population.

Conversation and stream tools help reconstruct the exchange.

---

## Filter Workflows and Follow Stream

When an application protocol can be reconstructed, filtering can lead into stream analysis.

Typical workflow:

```text
Identify protocol
    ↓
Identify host pair
    ↓
Identify connection
    ↓
Identify stream
    ↓
Follow stream
    ↓
Reconstruct communication
    ↓
Return to packet-level evidence
```

Do not treat a reconstructed stream as a replacement for packet evidence.

Use both.

---

## Filter Workflows and Expert Information

Expert Information can help identify packets requiring attention.

A practical workflow is:

```text
Capture
    ↓
Expert Information
    ↓
Identify event
    ↓
Filter related traffic
    ↓
Inspect surrounding packets
    ↓
Determine whether the event matters
```

An expert-info warning is an observation generated by Wireshark's analysis.

It still requires context.

---

## Filter Workflows for Large Captures

Large captures require disciplined narrowing.

Use:

```text
Time
Host
Protocol
Conversation
Service
Event
```

in a logical order.

For example:

```text
Incident window
    ↓
Suspected host
    ↓
Relevant protocol
    ↓
Relevant server
    ↓
Specific event
```

Avoid repeatedly scrolling through the packet list without a question.

---

## Build Reusable Investigation Patterns

Do not memorize only individual filters.

Memorize workflows.

### Host Workflow

```text
Host
→ Protocol
→ Destination
→ Service
→ Event
→ Conversation
```

### Connectivity Workflow

```text
Hosts
→ TCP
→ SYN
→ SYN-ACK
→ ACK
→ Data
→ Termination
```

### DNS Workflow

```text
Client
→ DNS
→ Query
→ Response
→ Status
→ Repetition
→ Timing
```

### Web Workflow

```text
Client
→ HTTP/TLS
→ Request
→ Response
→ Status
→ Timing
→ Stream
```

### Security Investigation Workflow

```text
Host
→ Destinations
→ Protocols
→ Services
→ Timing
→ Repetition
→ Correlation
→ Evidence
```

---

## Practical Exercise 1 — Host Investigation

Using a suitable capture:

```text
1. Identify one host.
2. Filter its traffic.
3. Identify its major protocols.
4. Choose one protocol.
5. Narrow to that protocol.
6. Identify important destinations.
7. Investigate one conversation.
8. Document what you observed.
```

Do not begin with a predetermined conclusion.

---

## Practical Exercise 2 — Connectivity Failure

Find a TCP connection that does not appear to establish normally.

Answer:

```text
Which host initiated it?
Was a SYN sent?
Was a SYN-ACK returned?
Was an ACK sent?
Was there a retransmission?
Was there a reset?
Where does the expected sequence stop?
```

Build a filter for each question.

---

## Practical Exercise 3 — DNS Investigation

Choose one client.

Determine:

```text
Which DNS server did it contact?
Which names did it query?
Which queries received responses?
Which queries were repeated?
Were any response codes unusual?
```

Record the filters used for each stage.

---

## Practical Exercise 4 — Web Investigation

Choose one HTTP conversation.

Determine:

```text
Client
Server
Host
Request method
Requested URI
Response status
Timing
Connection behavior
```

Use filtering to locate each piece of evidence.

---

## Practical Exercise 5 — Suspicious Communication Investigation

Choose a host and investigate its communications without assuming maliciousness.

Determine:

```text
Destinations
Protocols
Ports
Frequency
Timing
DNS activity
HTTP/TLS activity
Repeated connections
```

Then write:

```text
Observed:
What the packets directly show.

Interpretation:
What the pattern may indicate.

Uncertainty:
What the capture cannot establish.
```

---

## Practical Exercise 6 — Contradict Your Hypothesis

Choose a hypothesis such as:

```text
"The application is slow because of network retransmissions."
```

Use filters to look for evidence supporting it.

Then deliberately look for evidence against it.

Investigate:

```text
Retransmissions
TCP timing
Server response timing
DNS timing
TLS timing
Application response timing
```

Do not stop when you find evidence supporting your initial idea.

---

## Practical Exercise 7 — Build a Filter Tree

Start with:

```text
ip.addr == <host>
```

Then create a decision tree:

```text
Host
├── DNS
│   ├── Queries
│   └── Responses
│
├── TCP
│   ├── Connection setup
│   ├── Application traffic
│   └── Termination
│
├── TLS
│   ├── Handshake
│   └── Encrypted traffic
│
└── Other protocols
```

Use the tree to guide your investigation.

---

## Practical Exercise 8 — Time-Bounded Investigation

Choose a capture containing an identifiable event.

Define:

```text
Start time
End time
```

Then investigate only that period.

Determine:

```text
Which hosts were active?
Which protocols were active?
What changed during the window?
Which packet represents the important event?
What happened immediately before it?
What happened immediately afterward?
```

---

## Practical Exercise 9 — Filter Failure Analysis

Intentionally create a filter that produces zero results.

Then debug it systematically.

Record:

```text
Original question:
Filter:
Expected result:
Actual result:
First assumption checked:
Field checked:
Packet inspected:
Correction:
Final filter:
```

The goal is to develop troubleshooting discipline.

---

## Practical Exercise 10 — End-to-End Investigation

Choose a realistic question:

```text
Why did this client fail to communicate with this server?
```

Perform:

```text
1. Identify client.
2. Identify server.
3. Filter the conversation.
4. Determine protocol.
5. Inspect connection establishment.
6. Inspect application exchange.
7. Check timing.
8. Check retransmissions or resets.
9. Inspect termination.
10. Identify the point where behavior diverges from expectation.
11. Document evidence.
12. State uncertainty.
```

Your final conclusion must be based on packet evidence rather than the filter itself.

---

## The Filter Workflow Checklist

Before considering a filtering investigation complete:

```text
[ ] Did I define a specific question?
[ ] Did I identify the evidence needed?
[ ] Did I start broad enough?
[ ] Did I narrow based on observations?
[ ] Did I use the correct field?
[ ] Did I use source/destination direction correctly?
[ ] Did I validate important filters?
[ ] Did I inspect surrounding packets?
[ ] Did I correlate related traffic?
[ ] Did I consider alternative explanations?
[ ] Did I distinguish observation from interpretation?
[ ] Did I record important packet references?
[ ] Did I identify uncertainty?
[ ] Did I define the next question if the investigation continues?
```

---

## Professional Standard

A professional Wireshark user should not think of display filters as isolated commands.

They should think in workflows:

```text
Question
    ↓
Filter
    ↓
Evidence
    ↓
Correlation
    ↓
Interpretation
    ↓
Decision
```

The strongest analysts are not necessarily the people who remember the most filter syntax.

They are the people who can:

```text
Ask the right question
→ Find the right evidence
→ Build the right filter
→ Test the result
→ Challenge their assumptions
→ Correlate packets
→ Explain what the capture proves
```

That is the purpose of question-driven filtering.

---

## Completion Criteria

You are ready to move forward when you can independently take an unfamiliar packet-analysis problem and:

* Define the investigation question.
* Identify the evidence required.
* Build an initial filter.
* Progressively narrow the traffic.
* Switch between broad and focused views.
* Analyze host and service relationships.
* Investigate TCP connection behavior.
* Investigate DNS activity.
* Investigate HTTP and TLS traffic.
* Investigate suspicious communication patterns without jumping to conclusions.
* Use filters together with conversations, streams, statistics, and packet inspection.
* Debug filters that return errors or no results.
* Validate observations against packet evidence.
* Document uncertainty and limitations.
* Decide what question should be investigated next.

At this point, filtering becomes an investigation skill rather than a syntax exercise.
