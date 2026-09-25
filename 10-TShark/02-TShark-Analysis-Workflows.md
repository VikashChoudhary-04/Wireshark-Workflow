# TShark Analysis Workflows

## Purpose

This file turns TShark fundamentals into repeatable packet-analysis workflows.

The goal is not to memorize commands.

The goal is to take an investigation question, translate it into TShark actions, extract useful evidence, and reach a defensible conclusion.

Use this workflow:

```text
Investigation Question
        ↓
Identify the relevant traffic
        ↓
Apply a display filter
        ↓
Select useful fields
        ↓
Inspect packets or statistics
        ↓
Correlate timestamps, endpoints, ports, streams, and protocols
        ↓
Validate important observations
        ↓
Record evidence
        ↓
Form an interpretation
        ↓
Decide the next question
```

Use TShark only against traffic you are authorized to capture or analyze.

## Core Investigation Pattern

Start with a question instead of a command.

Examples:

```text
Which hosts communicated with the server?

Which DNS queries were made?

Did the TCP connection establish successfully?

Were packets retransmitted?

Which TLS server names are visible?

Which HTTP requests were made?

Which endpoint generated the largest amount of traffic?

When did the suspicious communication begin?

Which TCP stream contains the relevant conversation?
```

Then translate the question into progressively narrower analysis.

Example:

```text
Question:
Did a client successfully connect to a web server?

↓
Identify the client/server traffic

↓
Inspect TCP handshake

↓
Inspect destination port

↓
Inspect TCP termination or reset

↓
Inspect HTTP/TLS traffic if present

↓
Correlate timestamps

↓
Determine what actually happened
```

Avoid beginning with a large collection of unrelated commands.

## Start With Capture Metadata

Before analyzing a capture, understand what you are working with.

```bash
capinfos capture.pcapng
```

Useful information includes:

* capture format
* packet count
* capture duration
* first packet time
* last packet time
* interface information
* encapsulation
* packet size information

Then inspect the packet structure:

```bash
tshark -r capture.pcapng -c 20
```

For a more detailed view:

```bash
tshark -r capture.pcapng -c 5 -V
```

The purpose of this first pass is orientation.

Do not immediately assume that the capture contains the traffic you are looking for.

## Establish the Protocol Landscape

A protocol hierarchy provides a fast overview of what exists inside a capture.

```bash
tshark -r capture.pcapng -q -z io,phs
```

This can reveal protocols such as:

```text
Ethernet
  IPv4
    TCP
      TLS
        HTTP
    UDP
      DNS
```

The exact hierarchy depends on the capture.

Use it to answer:

```text
What protocols are present?

Which protocols dominate the capture?

Is the traffic mostly TCP or UDP?

Are DNS, HTTP, TLS, ICMP, ARP, or other protocols present?

Which protocol deserves deeper investigation?
```

Protocol hierarchy is an orientation tool, not a final conclusion.

## Identify Endpoints

Find the major communicating hosts.

```bash
tshark -r capture.pcapng -q -z endpoints,ip
```

For IPv6:

```bash
tshark -r capture.pcapng -q -z endpoints,ipv6
```

For Ethernet addresses:

```bash
tshark -r capture.pcapng -q -z endpoints,eth
```

Use endpoint statistics to answer:

```text
Which IP addresses appear?

Which hosts communicate frequently?

Which systems appear to be servers?

Which systems appear to be clients?

Are there unexpected endpoints?
```

Do not classify a host as malicious simply because it appears frequently.

Traffic volume and endpoint role require context.

## Analyze Conversations

Endpoint statistics tell you which hosts exist.

Conversation statistics help determine how they communicate.

For TCP:

```bash
tshark -r capture.pcapng -q -z conv,tcp
```

For UDP:

```bash
tshark -r capture.pcapng -q -z conv,udp
```

Use conversations to identify:

* source address
* destination address
* source port
* destination port
* packet counts
* byte counts
* conversation duration

A useful investigation pattern is:

```text
Endpoint
    ↓
Conversation
    ↓
Port
    ↓
Protocol
    ↓
Packets
    ↓
Timing
    ↓
Payload or higher-level protocol
```

This prevents conclusions based only on IP addresses.

## Question-Driven Display Filtering

TShark becomes significantly more useful when filters are built from questions.

Suppose the question is:

```text
Which DNS traffic exists?
```

Start broad:

```bash
tshark -r capture.pcapng -Y "dns"
```

Then narrow:

```bash
tshark -r capture.pcapng -Y "dns.qry.name"
```

Then extract fields:

```bash
tshark -r capture.pcapng \
  -Y "dns.qry.name" \
  -T fields \
  -e frame.time \
  -e ip.src \
  -e ip.dst \
  -e dns.qry.name
```

The important progression is:

```text
Protocol
→ Specific field
→ Specific condition
→ Extracted evidence
```

## Extract Timestamps

Time is one of the most important pieces of packet evidence.

Basic timestamp:

```bash
tshark -r capture.pcapng \
  -T fields \
  -e frame.number \
  -e frame.time
```

More useful investigation record:

```bash
tshark -r capture.pcapng \
  -T fields \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e tcp.srcport \
  -e tcp.dstport
```

Use timestamps to establish:

```text
What happened first?

How long did the connection take?

Did another event happen immediately afterward?

Did repeated activity occur?

Did a failure happen before or after another protocol event?
```

Avoid treating timestamps as absolute truth without considering capture time configuration and clock differences between systems.

## Extract Addresses and Ports

A generic TCP evidence view:

```bash
tshark -r capture.pcapng \
  -Y "tcp" \
  -T fields \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src \
  -e tcp.srcport \
  -e ip.dst \
  -e tcp.dstport
```

For UDP:

```bash
tshark -r capture.pcapng \
  -Y "udp" \
  -T fields \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src \
  -e udp.srcport \
  -e ip.dst \
  -e udp.dstport
```

This provides a compact connection-oriented view.

It is especially useful when a full packet decode contains too much information.

## DNS Investigation Workflow

DNS is often one of the easiest protocols to analyze with TShark because useful metadata is commonly visible.

### Step 1: Find DNS Traffic

```bash
tshark -r capture.pcapng -Y "dns"
```

### Step 2: Extract Queries

```bash
tshark -r capture.pcapng \
  -Y "dns.qry.name" \
  -T fields \
  -e frame.time \
  -e ip.src \
  -e ip.dst \
  -e dns.qry.name
```

### Step 3: Inspect Responses

```bash
tshark -r capture.pcapng \
  -Y "dns.flags.response == 1" \
  -T fields \
  -e frame.time \
  -e ip.src \
  -e ip.dst \
  -e dns.qry.name \
  -e dns.flags.rcode
```

### Step 4: Investigate Repeated Queries

Start with extracted names:

```bash
tshark -r capture.pcapng \
  -Y "dns.qry.name" \
  -T fields \
  -e dns.qry.name
```

If shell tooling is available, you can summarize repetitions:

```bash
tshark -r capture.pcapng \
  -Y "dns.qry.name" \
  -T fields \
  -e dns.qry.name | sort | uniq -c | sort -nr
```

This can help identify frequently queried names.

Frequency alone does not establish malicious behavior.

### Step 5: Investigate Failures

Look at response codes:

```bash
tshark -r capture.pcapng \
  -Y "dns.flags.response == 1" \
  -T fields \
  -e dns.qry.name \
  -e dns.flags.rcode
```

The next question should be based on the observation.

For example:

```text
Repeated failures
        ↓
Which client generated them?
        ↓
Which DNS server responded?
        ↓
When did they occur?
        ↓
Were successful queries mixed with failures?
```

## TCP Investigation Workflow

TCP analysis should answer whether a connection:

```text
was attempted
→ established
→ exchanged data
→ experienced problems
→ terminated
```

### Step 1: Find TCP Traffic

```bash
tshark -r capture.pcapng -Y "tcp"
```

### Step 2: Inspect the Handshake

Find SYN packets:

```bash
tshark -r capture.pcapng \
  -Y "tcp.flags.syn == 1 && tcp.flags.ack == 0"
```

Find SYN-ACK packets:

```bash
tshark -r capture.pcapng \
  -Y "tcp.flags.syn == 1 && tcp.flags.ack == 1"
```

Find resets:

```bash
tshark -r capture.pcapng \
  -Y "tcp.flags.reset == 1"
```

Find retransmissions:

```bash
tshark -r capture.pcapng \
  -Y "tcp.analysis.retransmission"
```

Find duplicate acknowledgments:

```bash
tshark -r capture.pcapng \
  -Y "tcp.analysis.duplicate_ack"
```

Find out-of-order packets:

```bash
tshark -r capture.pcapng \
  -Y "tcp.analysis.out_of_order"
```

### Step 3: Extract Connection Metadata

```bash
tshark -r capture.pcapng \
  -Y "tcp" \
  -T fields \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src \
  -e tcp.srcport \
  -e ip.dst \
  -e tcp.dstport \
  -e tcp.stream
```

The `tcp.stream` value is especially useful for isolating one TCP conversation.

### Step 4: Investigate One Stream

Once a stream number is identified:

```bash
tshark -r capture.pcapng \
  -Y "tcp.stream == 0"
```

Replace `0` with the relevant stream number.

This allows you to move from:

```text
Entire capture
```

to:

```text
Specific TCP conversation
```

### Step 5: Interpret TCP Evidence Carefully

A retransmission means a packet was retransmitted.

It does not automatically mean:

```text
the server is broken
```

or:

```text
the network is definitely congested
```

Possible explanations must be evaluated using:

* packet timing
* direction
* sequence numbers
* acknowledgments
* duplicate ACKs
* window behavior
* resets
* application behavior
* capture position

The packet is evidence.

The explanation requires correlation.

## TCP Timing Workflow

Extract TCP timing-related fields when available:

```bash
tshark -r capture.pcapng \
  -Y "tcp" \
  -T fields \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e tcp.stream \
  -e tcp.seq \
  -e tcp.ack
```

For TCP analysis fields generated by Wireshark's TCP analysis engine, inspect the fields available in your installed version.

You can discover them with:

```bash
tshark -G fields | grep -i "tcp.analysis"
```

Field names and available dissections can vary by Wireshark version.

## HTTP Investigation Workflow

HTTP analysis depends on whether the traffic is actually visible as HTTP.

Start with:

```bash
tshark -r capture.pcapng -Y "http"
```

Extract request information:

```bash
tshark -r capture.pcapng \
  -Y "http.request" \
  -T fields \
  -e frame.time \
  -e ip.src \
  -e ip.dst \
  -e http.request.method \
  -e http.host \
  -e http.request.uri
```

Extract response information:

```bash
tshark -r capture.pcapng \
  -Y "http.response" \
  -T fields \
  -e frame.time \
  -e ip.src \
  -e ip.dst \
  -e http.response.code
```

Investigate one host:

```bash
tshark -r capture.pcapng \
  -Y 'http.host == "example.local"'
```

Use the actual host observed in the capture.

Useful questions:

```text
Which client made the request?

Which server received it?

What HTTP method was used?

What URI was requested?

What response status was returned?

Did redirects occur?

Did requests fail?

Which TCP stream carried the request?
```

Do not assume HTTP content is available merely because traffic uses TCP port 80.

Protocol dissection and observed application behavior should be verified from the packet data.

## TLS Investigation Workflow

TLS encrypts application content, but useful metadata can remain visible.

Start with:

```bash
tshark -r capture.pcapng -Y "tls"
```

Extract TLS handshake information:

```bash
tshark -r capture.pcapng \
  -Y "tls.handshake" \
  -T fields \
  -e frame.time \
  -e ip.src \
  -e ip.dst \
  -e tcp.stream
```

Inspect available TLS fields:

```bash
tshark -G fields | grep -i "^.*tls.*handshake"
```

Depending on the capture and TLS version, useful metadata may include:

* handshake messages
* protocol version information
* cipher information
* certificate information
* server name information where visible
* alert messages
* TCP stream identifiers
* timing

For example, investigate TLS alerts:

```bash
tshark -r capture.pcapng -Y "tls.alert"
```

The absence of decrypted application content does not mean TLS traffic is unanalyzable.

A useful workflow is:

```text
TCP connection
→ TLS handshake
→ visible metadata
→ timing
→ certificate information if present
→ alerts/failures
→ encrypted application phase
```

## ICMP Investigation Workflow

Start with:

```bash
tshark -r capture.pcapng -Y "icmp"
```

For IPv6:

```bash
tshark -r capture.pcapng -Y "icmpv6"
```

Extract useful fields:

```bash
tshark -r capture.pcapng \
  -Y "icmp" \
  -T fields \
  -e frame.time \
  -e ip.src \
  -e ip.dst \
  -e icmp.type \
  -e icmp.code
```

Use ICMP analysis to investigate questions such as:

```text
Was a host reachable?

Did a destination respond?

Was a destination unreachable?

Did a packet encounter a routing-related error?

Did ICMP errors correlate with another connection failure?
```

Do not equate absence of ICMP replies with proof that a host is offline.

Firewalls and network policies can suppress ICMP.

## ARP Investigation Workflow

Find ARP traffic:

```bash
tshark -r capture.pcapng -Y "arp"
```

Extract important fields:

```bash
tshark -r capture.pcapng \
  -Y "arp" \
  -T fields \
  -e frame.time \
  -e eth.src \
  -e arp.src.proto_ipv4 \
  -e arp.src.hw_mac \
  -e arp.dst.proto_ipv4 \
  -e arp.dst.hw_mac \
  -e arp.opcode
```

Questions:

```text
Who is asking for an IP address?

Who is answering?

Which MAC address is associated with an IP?

Are there repeated ARP requests?

Does the observed mapping change?
```

Unexpected ARP behavior should be investigated in context rather than automatically labeled as an attack.

## Extract TCP Stream Identifiers

Stream IDs make correlation easier.

```bash
tshark -r capture.pcapng \
  -Y "tcp" \
  -T fields \
  -e frame.number \
  -e tcp.stream \
  -e ip.src \
  -e tcp.srcport \
  -e ip.dst \
  -e tcp.dstport
```

To inspect a specific stream:

```bash
tshark -r capture.pcapng \
  -Y "tcp.stream == 5"
```

To extract selected information from that stream:

```bash
tshark -r capture.pcapng \
  -Y "tcp.stream == 5" \
  -T fields \
  -e frame.number \
  -e frame.time \
  -e ip.src \
  -e ip.dst \
  -e tcp.seq \
  -e tcp.ack
```

This is one of the most useful transitions in command-line packet analysis:

```text
Find traffic
→ Identify stream
→ Isolate stream
→ Analyze stream
```

## Protocol Statistics

TShark can expose useful statistics without printing every packet.

Protocol hierarchy:

```bash
tshark -r capture.pcapng -q -z io,phs
```

IP endpoints:

```bash
tshark -r capture.pcapng -q -z endpoints,ip
```

TCP conversations:

```bash
tshark -r capture.pcapng -q -z conv,tcp
```

UDP conversations:

```bash
tshark -r capture.pcapng -q -z conv,udp
```

These statistics are useful for orientation and triage.

Use packet-level filtering when you need evidence about a specific event.

## I/O Statistics

I/O statistics can help identify traffic patterns over time.

A basic form is:

```bash
tshark -r capture.pcapng -q -z io,stat,1
```

The final value represents the interval.

For example:

```text
1 second
```

produces one-second intervals.

Larger intervals can make longer captures easier to understand.

Use I/O statistics to ask:

```text
When did traffic increase?

When did traffic decrease?

Was activity concentrated in a short period?

Did multiple events happen at the same time?
```

I/O graphs are particularly useful when investigating:

* application slowdowns
* bursts
* outages
* periodic traffic
* unusual activity
* timing relationships

## Packet Count and Analysis Scope

When validating a workflow, limiting the number of packets can make output manageable.

```bash
tshark -r capture.pcapng -c 20
```

Use `-c` when you intentionally want only the first specified number of packets from the input.

For live capture workflows, autostop conditions can also be useful.

For example:

```bash
tshark -i 1 -a duration:30
```

captures for a limited duration.

Another useful condition is a packet count:

```bash
tshark -i 1 -a packets:100
```

Use these only with authorized traffic.

## Quiet Statistics Mode

Statistics commands often use:

```bash
-q
```

For example:

```bash
tshark -r capture.pcapng -q -z endpoints,ip
```

The purpose is to suppress normal packet output so the requested statistics become the primary result.

Think of:

```text
normal output
```

as:

```text
packet-by-packet evidence
```

and:

```text
-q -z ...
```

as:

```text
summary/statistical evidence
```

Both are useful, but they answer different questions.

## Structured Field Extraction

For machine-friendly analysis:

```bash
tshark -r capture.pcapng \
  -Y "dns" \
  -T fields \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e dns.qry.name
```

To use a custom separator:

```bash
tshark -r capture.pcapng \
  -Y "dns" \
  -T fields \
  -E separator=$'\t' \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e dns.qry.name
```

Shell syntax differs between environments.

On Windows PowerShell, prefer syntax appropriate for PowerShell rather than copying Bash-specific quoting blindly.

For CSV-style output:

```bash
tshark -r capture.pcapng \
  -Y "dns" \
  -T fields \
  -E header=y \
  -E separator=, \
  -E quote=d \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e dns.qry.name
```

Always inspect the resulting output before treating it as a clean dataset.

Fields can be absent, repeated, or contain characters that require careful parsing.

## Validate GUI Findings With TShark

TShark is especially useful for validating a discovery made in the Wireshark GUI.

Example workflow:

```text
Wireshark GUI
    ↓
Identify suspicious DNS query
    ↓
Record query name and frame number
    ↓
Reproduce the observation with TShark
    ↓
Extract timestamp, source, destination, and query
    ↓
Preserve command and output
```

Example:

```bash
tshark -r capture.pcapng \
  -Y 'dns.qry.name == "example.local"' \
  -T fields \
  -e frame.number \
  -e frame.time \
  -e ip.src \
  -e ip.dst \
  -e dns.qry.name
```

This creates a repeatable command-line representation of the finding.

## Build Commands From Investigation Questions

Do not memorize complete commands.

Build them from components.

### Question

```text
Which client contacted the DNS server?
```

### Protocol

```text
dns
```

### Fields

```text
timestamp
source
destination
query
```

### Command

```bash
tshark -r capture.pcapng \
  -Y "dns.qry.name" \
  -T fields \
  -e frame.time \
  -e ip.src \
  -e ip.dst \
  -e dns.qry.name
```

Another example:

### Question

```text
Which TCP connections experienced retransmissions?
```

### Filter

```text
tcp.analysis.retransmission
```

### Fields

```text
timestamp
source
destination
stream
```

### Command

```bash
tshark -r capture.pcapng \
  -Y "tcp.analysis.retransmission" \
  -T fields \
  -e frame.time \
  -e ip.src \
  -e ip.dst \
  -e tcp.stream
```

This is the skill you want to develop.

## Combining Statistics and Packet Analysis

A strong investigation usually combines two levels.

### Level 1: Statistical overview

```bash
tshark -r capture.pcapng -q -z endpoints,ip
tshark -r capture.pcapng -q -z conv,tcp
```

### Level 2: Packet-level investigation

```bash
tshark -r capture.pcapng -Y "tcp.analysis.retransmission"
```

### Level 3: Focused extraction

```bash
tshark -r capture.pcapng \
  -Y "tcp.stream == 7" \
  -T fields \
  -e frame.number \
  -e frame.time \
  -e ip.src \
  -e ip.dst \
  -e tcp.seq \
  -e tcp.ack
```

The general pattern is:

```text
Summarize
→ Identify
→ Isolate
→ Inspect
→ Correlate
→ Conclude
```

## Multiple Captures

When an investigation involves multiple capture files, first analyze each capture independently.

Avoid assuming that packet numbering, timestamps, or stream IDs are globally interchangeable between files.

Useful workflow:

```text
Capture A
→ establish timeline and endpoints

Capture B
→ establish timeline and endpoints

Compare:
→ addresses
→ protocols
→ timestamps
→ conversations
→ observed behavior
```

If captures need to be combined, use appropriate Wireshark command-line utilities such as `mergecap` and understand how timestamps and encapsulation are handled.

Do not combine captures simply because they appear related.

Establish why the captures belong to the same investigation first.

## Evidence Extraction Workflow

For an important finding, extract enough information to reproduce it.

Record:

```text
Capture:
capture.pcapng

Question:
Which client generated the DNS request?

Filter:
dns.qry.name == "example.local"

Frame:
123

Timestamp:
observed frame time

Source:
client IP

Destination:
DNS server IP

Protocol:
DNS

Relevant field:
dns.qry.name

Interpretation:
The client sent a DNS query for the observed name.
```

The distinction matters:

```text
Observation:
A DNS query for example.local was observed.

Interpretation:
The client attempted to resolve example.local.

Conclusion:
Requires additional context.
```

Do not skip directly from packet output to a strong conclusion.

## Practical Exercise 1: Capture Orientation

Use a supplied or self-generated authorized PCAP.

Run:

```bash
capinfos capture.pcapng
```

Then:

```bash
tshark -r capture.pcapng -q -z io,phs
```

Then:

```bash
tshark -r capture.pcapng -q -z endpoints,ip
```

Answer:

```text
How many packets are present?

What protocols are visible?

Which IP endpoints exist?

Which protocols deserve further investigation?
```

## Practical Exercise 2: DNS Investigation

Find all DNS queries.

```bash
tshark -r capture.pcapng \
  -Y "dns.qry.name" \
  -T fields \
  -e frame.time \
  -e ip.src \
  -e ip.dst \
  -e dns.qry.name
```

Then summarize repeated names:

```bash
tshark -r capture.pcapng \
  -Y "dns.qry.name" \
  -T fields \
  -e dns.qry.name | sort | uniq -c | sort -nr
```

Answer:

```text
Which client generated the queries?

Which names were queried?

Which names appeared repeatedly?

Which DNS server responded?

Were there failed responses?
```

## Practical Exercise 3: TCP Problem Investigation

Find retransmissions:

```bash
tshark -r capture.pcapng \
  -Y "tcp.analysis.retransmission"
```

Extract their streams:

```bash
tshark -r capture.pcapng \
  -Y "tcp.analysis.retransmission" \
  -T fields \
  -e frame.number \
  -e frame.time \
  -e ip.src \
  -e ip.dst \
  -e tcp.stream
```

Pick one stream and investigate it:

```bash
tshark -r capture.pcapng \
  -Y "tcp.stream == STREAM_ID"
```

Replace:

```text
STREAM_ID
```

with the observed stream number.

Determine:

```text
Which endpoints were involved?

Which ports were used?

When did the retransmission occur?

Were duplicate ACKs present?

Were resets present?

What other evidence surrounds the event?
```

## Practical Exercise 4: HTTP or TLS Investigation

If HTTP exists:

```bash
tshark -r capture.pcapng -Y "http"
```

If TLS exists:

```bash
tshark -r capture.pcapng -Y "tls"
```

Extract:

```text
timestamp
source
destination
stream
visible application metadata
```

Then answer:

```text
What communication occurred?

Which endpoint initiated it?

Which server was contacted?

What metadata remains visible?

What information is unavailable because of encryption?
```

## Practical Exercise 5: GUI-to-TShark Validation

Open the same capture in Wireshark.

Identify one meaningful observation using the GUI.

Examples:

```text
A DNS request
A TCP retransmission
A TCP reset
An HTTP request
A TLS handshake
An ICMP error
```

Record:

```text
frame number
timestamp
source
destination
protocol
stream if applicable
```

Then reproduce the finding using TShark.

The objective is not simply to produce the same packet.

The objective is to prove that you can translate GUI analysis into a repeatable command-line workflow.

## Common Mistakes

### Mistake: Treating TShark as a packet-dumping tool

The goal is not to print thousands of packets.

Use:

```text
filters
fields
statistics
streams
timelines
```

to reduce the problem.

### Mistake: Starting with overly broad output

Instead of:

```bash
tshark -r capture.pcapng -V
```

for a large capture, first establish:

```text
protocols
endpoints
conversations
```

Then narrow the analysis.

### Mistake: Assuming ports identify protocols

A port number is useful evidence, but it is not sufficient by itself to establish application protocol identity.

Use protocol dissection and packet evidence.

### Mistake: Treating Wireshark-generated analysis fields as absolute truth

Fields such as:

```text
tcp.analysis.retransmission
tcp.analysis.duplicate_ack
```

are analysis results produced from observed packet relationships.

Validate important findings with the surrounding packet sequence.

### Mistake: Ignoring capture limitations

A capture can be incomplete because of:

* capture location
* packet loss during capture
* filtering
* truncation
* unsupported or incomplete dissection
* encryption
* missing interfaces
* asymmetric visibility

Absence of evidence is not automatically evidence that an event did not happen.

### Mistake: Copying shell syntax between operating systems

Commands that work in Bash may behave differently in:

```text
PowerShell
Command Prompt
zsh
other shells
```

Understand the command and adapt quoting and pipelines to the environment.

### Mistake: Extracting data without preserving context

A list of IP addresses is not a complete investigation.

Preserve:

```text
time
source
destination
protocol
port
stream
frame
filter
```

when relevant.

## Professional Workflow

A repeatable TShark investigation can follow this sequence:

```text
1. Preserve the original capture
2. Inspect capture metadata
3. Establish protocol hierarchy
4. Identify endpoints
5. Identify conversations
6. Define the investigation question
7. Build a display filter
8. Extract relevant fields
9. Isolate streams when required
10. Correlate timestamps
11. Validate important observations
12. Record evidence
13. Separate observation from interpretation
14. Identify remaining uncertainty
15. Decide the next investigative question
```

For important investigations, preserve the exact commands used.

Example:

```text
Capture:
incident.pcapng

Command:
tshark -r incident.pcapng -Y "tcp.analysis.retransmission" -T fields -e frame.number -e frame.time -e ip.src -e ip.dst -e tcp.stream

Purpose:
Identify TCP streams containing retransmissions.

Result:
Stream 12 contained repeated retransmission events.

Next question:
What happened immediately before and after the retransmissions?
```

## Completion Criteria

You are ready to move forward when you can:

* explain the difference between packet output, statistics, and field extraction
* inspect a PCAP with TShark without blindly dumping everything
* use `-Y` to narrow an investigation
* extract useful fields with `-T fields`
* identify endpoints and conversations
* identify and isolate TCP streams
* investigate DNS traffic
* investigate TCP handshakes and analysis events
* investigate HTTP when it is visible
* investigate TLS metadata and limitations
* investigate ICMP and ARP traffic
* use timestamps as part of correlation
* use protocol hierarchy and conversation statistics for orientation
* use I/O statistics to understand traffic over time
* validate a Wireshark GUI observation with TShark
* distinguish observed evidence from interpretation
* preserve enough context to reproduce an important finding
* explain limitations of the capture before making a strong conclusion

The target skill is:

```text
Question
    ↓
TShark filter
    ↓
Relevant fields/statistics
    ↓
Focused evidence
    ↓
Correlation
    ↓
Interpretation
    ↓
Defensible conclusion
```

Once this becomes natural, TShark stops being a collection of commands and becomes a repeatable packet-analysis workflow.
