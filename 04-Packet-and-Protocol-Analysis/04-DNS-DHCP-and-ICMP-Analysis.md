# Wireshark DNS, DHCP, and ICMP Analysis

## Objective

This file teaches practical analysis of DNS, DHCP, ICMP, and ICMPv6 traffic in Wireshark.

These protocols are especially valuable because they often explain problems that appear at a higher layer.

For example:

```text id="2qj4y8"
Application cannot connect
        ↓
DNS resolution fails
        ↓
Correct hostname cannot be resolved
```

or:

```text id="8zq6a1"
Client has no usable address
        ↓
DHCP exchange fails
        ↓
Normal IP communication cannot begin
```

or:

```text id="g9s2q4"
Host appears unreachable
        ↓
ICMP error or routing evidence
        ↓
Network-layer behavior becomes visible
```

The core workflow is:

```text id="8x5f4v"
Question
    ↓
Protocol
    ↓
Message sequence
    ↓
Fields
    ↓
Timing
    ↓
Correlation
    ↓
Interpretation
```

---

## Why These Protocols Matter

DNS, DHCP, and ICMP often sit underneath application troubleshooting.

They can explain:

```text id="u0f7b9"
Name-resolution failures
Address-assignment failures
Gateway discovery
Routing errors
Reachability problems
Application delays
Repeated requests
Unexpected network behavior
```

A useful dependency model is:

```text id="9d4t4m"
DHCP / Network configuration
        ↓
IP connectivity
        ↓
DNS resolution
        ↓
TCP/UDP communication
        ↓
Application
```

This is not a strict dependency for every application, but it is a useful troubleshooting model.

---

## DNS Overview

The Domain Name System maps names to network information.

A simple example is:

```text id="5psj9a"
www.example.com
       ↓
DNS query
       ↓
IP address
```

A DNS exchange commonly contains:

```text id="q8r0sk"
Client
    ↓
DNS Query
    ↓
DNS Server
    ↓
DNS Response
```

The response may contain:

```text id="k6h8s3"
A records
AAAA records
CNAME records
Other record types
```

The practical objective is to determine:

```text id="1h6h7j"
What name was queried?
Who queried it?
Which DNS server answered?
What answer was returned?
How long did it take?
Did it succeed?
```

---

## DNS Transport

DNS commonly uses UDP for ordinary queries, but TCP can also be used in legitimate circumstances.

Therefore:

```text id="e8f4t0"
DNS ≠ UDP only
```

When analyzing DNS, begin with the application protocol:

```text id="v7r5w2"
dns
```

Then inspect whether the transport is:

```text id="u5p2n1"
UDP
```

or:

```text id="m3y9x4"
TCP
```

Do not assume the transport solely from the protocol's most common behavior.

---

## DNS Query and Response

A basic exchange is:

```text id="q8m1w3"
Client
    ↓
Query
    ↓
DNS server
    ↓
Response
```

A query asks for information.

A response provides:

```text id="j0x4v9"
Answer
Authority information
Additional information
Response status
```

The query and response should be correlated using the DNS transaction information and packet timing.

---

## DNS Query Identification

A useful starting filter is:

```text id="p2m7s4"
dns
```

Then distinguish queries from responses.

For example:

```text id="g8v2r6"
dns.flags.response == 0
```

can identify DNS queries.

And:

```text id="c6q4k8"
dns.flags.response == 1
```

can identify DNS responses.

Always inspect actual packet details when learning the field.

---

## DNS Query Name

One of the most important DNS fields is the query name.

For example:

```text id="t6w3x2"
dns.qry.name
```

A specific query can be filtered with:

```text id="f4n8j5"
dns.qry.name == "example.com"
```

This lets you answer:

```text id="b8s3q6"
Which packets queried this name?
```

Then investigate:

```text id="w5q7m2"
Who generated the query?
When?
Which DNS server received it?
What response returned?
```

---

## DNS Query Type

DNS queries can request different record types.

Common examples include:

```text id="g5v2n8"
A
AAAA
CNAME
MX
TXT
NS
PTR
```

The query type tells you what information the client requested.

For example:

```text id="z8r3k5"
A
```

generally requests IPv4 address information.

```text id="m7c4x9"
AAAA
```

generally requests IPv6 address information.

Do not infer the final answer solely from the query type.

Inspect the response.

---

## DNS Response Codes

DNS responses include a response code indicating the result of the DNS operation.

Common examples include:

```text id="j4w6v2"
NOERROR
NXDOMAIN
SERVFAIL
REFUSED
```

The exact numeric representation can be inspected in Wireshark.

The important investigative question is:

```text id="r9f1k3"
Did the DNS server successfully answer the query?
```

Then determine why the response has its particular status.

---

## NXDOMAIN

NXDOMAIN indicates that the requested domain name does not exist according to the responding DNS server.

If you observe:

```text id="d8s5p2"
NXDOMAIN
```

record:

```text id="q3m7x6"
Queried name
Client
DNS server
Timestamp
Frequency
```

Repeated NXDOMAIN responses may be completely legitimate or may deserve investigation depending on the environment and query pattern.

Do not label them malicious automatically.

---

## SERVFAIL

SERVFAIL indicates that the DNS server failed to successfully process the query.

Possible reasons can involve:

```text id="a5k9c2"
Upstream resolution
DNS server configuration
DNSSEC-related problems
Temporary server problems
Network conditions
```

The packet itself may not tell you the exact root cause.

Use additional evidence:

```text id="f7m4r1"
Repeated queries
Different DNS servers
Timing
Other DNS responses
Application behavior
```

---

## REFUSED

REFUSED indicates that the server declined to perform the requested operation.

Possible explanations include:

```text id="c9x2m8"
Server policy
Access restrictions
Configuration
Query restrictions
```

The correct interpretation depends on the DNS environment.

---

## DNS Response Timing

DNS timing can affect application performance.

A basic workflow is:

```text id="p7d3k1"
Query
    ↓
Timestamp
    ↓
Response
    ↓
Timestamp
    ↓
Difference
```

Investigate:

```text id="k5n8s4"
Typical response time
Slow responses
Repeated retries
Missing responses
Server changes
```

One slow query may be an isolated event.

A consistent pattern is stronger evidence.

---

## DNS Retries

A client may retry a query when a response is not received as expected.

A practical workflow is:

```text id="m3g6v9"
Query
    ↓
No visible response
    ↓
Repeated query
    ↓
Response or continued retry
```

When this happens, investigate:

```text id="q1w8e5"
Query timing
DNS server
Transport
Network path
Response visibility
Capture completeness
```

Do not assume that the first query was lost.

---

## DNS Multiple Servers

A client may use more than one DNS server.

When troubleshooting resolution, identify:

```text id="v5x3k7"
Primary DNS server
Secondary DNS server
Destination IP
Response timing
Response status
```

A useful workflow is:

```text id="z2m8c6"
Client
    ↓
DNS Server A
    ↓
Response?

If not:
    ↓
DNS Server B
    ↓
Response?
```

This can reveal whether the issue is associated with a particular resolver.

---

## DNS CNAME Chains

A DNS response may return a CNAME that points to another name.

Conceptually:

```text id="q8v4m2"
app.example.com
    ↓
CNAME
    ↓
service.example.net
    ↓
A / AAAA
    ↓
IP address
```

When analyzing name resolution, do not stop at the first name.

Inspect the entire response when relevant.

---

## DNS A and AAAA Responses

A query may return:

```text id="h2n7q4"
A
```

for IPv4 addresses.

Or:

```text id="k6m3p8"
AAAA
```

for IPv6 addresses.

An application may prefer one address family depending on the environment.

When troubleshooting connection behavior, correlate:

```text id="f9x5s1"
DNS response
    ↓
Selected address
    ↓
Subsequent connection
```

This can help identify whether the application connected to the address returned by DNS.

---

## DNS Reverse Lookup

Reverse DNS maps an IP address back toward a name using PTR records.

A reverse lookup may appear as:

```text id="n4k8w2"
IP address
    ↓
PTR query
    ↓
Hostname
```

Reverse DNS can be useful during:

```text id="q7c2m5"
Incident analysis
Host identification
Log correlation
Troubleshooting
```

Do not treat a reverse DNS name as authoritative proof of ownership or identity.

---

## DNS and Security Investigation

DNS can be useful for investigating:

```text id="d6p3v8"
Unexpected destinations
Repeated lookups
High-frequency queries
Unusual names
Failed resolutions
Changing destinations
Host-specific resolution patterns
```

A practical workflow is:

```text id="g4m9x1"
Host
    ↓
DNS traffic
    ↓
Query names
    ↓
Query frequency
    ↓
Responses
    ↓
Resolved addresses
    ↓
Subsequent communication
```

The DNS query is only the beginning.

Correlate it with the traffic that follows.

---

## DNS Investigation: Known Domain

Question:

```text id="t8y2p5"
Which hosts queried example.com?
```

Start:

```text id="r3k7m9"
dns.qry.name == "example.com"
```

Then inspect:

```text id="m4x1c8"
Source IP
Destination DNS server
Timestamp
Query type
Response
```

Then determine:

```text id="v6n3s2"
Which hosts?
How often?
What addresses were returned?
Did they connect to those addresses?
```

---

## DNS Investigation: Suspicious Pattern

Question:

```text id="b7q4m1"
Is one host repeatedly querying unusual names?
```

Start:

```text id="p9c2x6"
dns && ip.src == 192.168.1.10
```

Then inspect:

```text id="w5m8r3"
Query names
Query lengths
Frequency
Intervals
Response codes
Returned addresses
```

Then correlate with:

```text id="g2n6v4"
HTTP
TLS
TCP
UDP
Other application traffic
```

A suspicious-looking DNS pattern requires context before conclusions are drawn.

---

## DHCP Overview

DHCP provides IPv4 configuration to clients.

A common DHCP exchange is:

```text id="k4p8x1"
Discover
    ↓
Offer
    ↓
Request
    ↓
ACK
```

This is commonly called DORA:

```text id="e6m2q9"
D
Discover

O
Offer

R
Request

A
ACK
```

The exact exchange can vary depending on the environment.

---

## DHCP Discover

The client may broadcast a DHCP Discover when it needs configuration.

Conceptually:

```text id="z1n7c4"
Client
    ↓
DHCP Discover
    ↓
Available DHCP infrastructure
```

The packet can provide information about:

```text id="u3m9v2"
Client identity
Requested options
Transaction ID
Broadcast behavior
```

Inspect the packet details rather than relying only on the Info column.

---

## DHCP Offer

A DHCP server can respond with an Offer.

The Offer may include:

```text id="r8k4p2"
Proposed IPv4 address
Subnet mask
Default gateway
DNS servers
Lease information
Server identifier
Other options
```

When analyzing:

```text id="m5x1c7"
Which server sent the Offer?
Which address was offered?
Which configuration options were included?
```

---

## DHCP Request

The client can request a configuration.

This may identify:

```text id="q7v3n8"
Requested IP
Selected server
Client identity
```

The request connects the earlier Discover/Offer stages with the eventual configuration.

---

## DHCP ACK

The DHCP server may acknowledge the request.

A normal sequence is:

```text id="j2c8m5"
Discover
    ↓
Offer
    ↓
Request
    ↓
ACK
```

The ACK can provide the final configuration information.

Afterward, inspect subsequent IP traffic.

This lets you correlate:

```text id="x6n1r4"
DHCP configuration
    ↓
IP communication
```

---

## DHCP Troubleshooting

Question:

```text id="s9q4w6"
Why did the client fail to obtain an IPv4 address?
```

Start with:

```text id="b5m2x8"
dhcp
```

Then build:

```text id="v1r7k3"
Discover?
    ↓
Offer?
    ↓
Request?
    ↓
ACK?
```

If the sequence stops:

```text id="c8n4p6"
Identify the missing stage.
```

Then investigate:

```text id="h2m7s1"
Client
Server
Broadcast visibility
Timing
Retries
Other DHCP servers
Capture location
```

---

## DHCP Multiple Servers

Multiple DHCP servers can respond to a broadcast Discover.

When this occurs, inspect:

```text id="w4x9c2"
Server identifiers
Offered addresses
Timing
Client selection
Final ACK
```

This can help explain unexpected configuration.

Do not assume that every DHCP Offer becomes the client's final configuration.

---

## DHCP Lease Renewal

DHCP does not end with the initial address assignment.

Clients can later renew their leases.

During renewal, investigate:

```text id="n6q2v8"
Request
Response
Lease timing
Server
Address
```

This can be useful when a client suddenly loses network configuration after previously working normally.

---

## DHCP Decline and NAK

DHCP can also contain messages indicating problems or rejection.

For example, a DHCP NAK may indicate that a requested configuration is not acceptable to the server.

When investigating:

```text id="f3m8x5"
Identify message
Identify server
Identify requested address
Inspect timing
Inspect surrounding DHCP exchange
```

Do not interpret the message without considering the DHCP state.

---

## ICMP Overview

ICMP provides network-layer control and error messaging.

Common ICMP types include:

```text id="p5r8k2"
Echo Request
Echo Reply
Destination Unreachable
Time Exceeded
```

ICMP is useful for:

```text id="j7m3x9"
Connectivity testing
Path troubleshooting
Routing diagnostics
Error reporting
```

---

## ICMP Echo

A common ping sequence is:

```text id="v8n4q1"
Echo Request
    ↓
Echo Reply
```

Filter:

```text id="a3x6m9"
icmp
```

Then inspect:

```text id="t2k7p4"
Type
Code
Identifier
Sequence
Timestamp
Source
Destination
```

---

## Ping Timing

To estimate round-trip behavior:

```text id="q9m2v6"
Echo Request timestamp
        ↓
Echo Reply timestamp
        ↓
Difference
```

Compare multiple requests.

For example:

```text id="w4p7c3"
Request 1 → Reply 1
Request 2 → Reply 2
Request 3 → Reply 3
```

A series is more useful than one sample.

---

## Missing Echo Replies

Suppose:

```text id="s3n8k5"
Echo Request
```

appears without:

```text id="f7q2m1"
Echo Reply
```

Possible explanations include:

```text id="c9v4x6"
Destination unreachable
ICMP filtering
Firewall policy
Host not responding
Routing issue
Capture visibility
```

Investigate other traffic before concluding that the host is unavailable.

---

## ICMP Destination Unreachable

An ICMP Destination Unreachable message provides information about a network-layer problem.

When you find one:

```text id="g2k8p5"
1. Identify ICMP type/code.
2. Identify source.
3. Identify destination.
4. Inspect quoted original packet.
5. Correlate timing.
6. Determine what communication triggered the message.
```

The quoted original packet can be especially valuable.

---

## ICMP Time Exceeded

Time Exceeded commonly appears when a packet's TTL expires.

This is important for:

```text id="r5m9x2"
Traceroute-style analysis
Routing loops
Unexpected paths
Hop-count analysis
```

A useful workflow:

```text id="n7c4p8"
Time Exceeded
    ↓
Quoted original packet
    ↓
Original destination
    ↓
TTL
    ↓
Source of ICMP message
```

---

## ICMPv6

ICMPv6 supports both:

```text id="m8q2v5"
IPv6 control/error messages
```

and:

```text id="x4n7c1"
Neighbor Discovery
Router Discovery
Address resolution
```

Start with:

```text id="y6p3m8"
icmpv6
```

Then identify the message type.

---

## IPv6 Neighbor Discovery

IPv6 uses ICMPv6 rather than ARP for neighbor discovery.

A simplified exchange is:

```text id="t9r4x7"
Neighbor Solicitation
    ↓
Neighbor Advertisement
```

These messages help establish or verify local IPv6 reachability.

When troubleshooting:

```text id="f2m8c5"
Is Neighbor Solicitation sent?
Is Neighbor Advertisement returned?
Are the addresses correct?
Are the link-layer options present?
```

---

## Router Solicitation

An IPv6 host can solicit router information.

Conceptually:

```text id="n5q7v3"
Host
    ↓
Router Solicitation
    ↓
IPv6 routers
```

This can lead to a Router Advertisement.

---

## Router Advertisement

Router Advertisements can provide information about:

```text id="p8c2m6"
IPv6 prefixes
Default-router information
Address configuration
Other network parameters
```

When troubleshooting IPv6 configuration, correlate:

```text id="r4m9x1"
Router Solicitation
    ↓
Router Advertisement
    ↓
Address configuration
    ↓
IPv6 communication
```

---

## DNS + TCP/UDP Correlation

DNS is often followed by application communication.

For example:

```text id="k6x2p9"
DNS Query
    ↓
DNS Response
    ↓
Resolved IP
    ↓
TCP SYN
    ↓
Application
```

This creates a powerful troubleshooting workflow.

Question:

```text id="q3n7m5"
Did the client connect to the address DNS returned?
```

Investigate:

```text id="v8c4r2"
DNS response
    ↓
Returned address
    ↓
Subsequent destination IP
```

---

## DNS + Application Failure

Suppose a user reports:

```text id="w5p2n8"
"The website is not opening."
```

Do not start with HTTP.

First ask:

```text id="m9c6x3"
Was DNS queried?
Did the response return?
Was the response successful?
What address was returned?
```

Then:

```text id="j4r8q1"
Did the client attempt TCP/TLS communication with that address?
```

This separates name-resolution problems from transport/application problems.

---

## DHCP + Connectivity Failure

Suppose:

```text id="d6p3m9"
A new client cannot access the network.
```

Check:

```text id="r7x2c5"
DHCP Discover
    ↓
Offer
    ↓
Request
    ↓
ACK
```

If no address is assigned:

```text id="n4m8v1"
Investigate DHCP first.
```

If DHCP succeeds:

```text id="q8c3x6"
Move to:
ARP / Neighbor Discovery
    ↓
IP
    ↓
DNS
    ↓
TCP/UDP
```

---

## ICMP + Connectivity Failure

Suppose:

```text id="b5n9r2"
A server appears unreachable.
```

Investigate:

```text id="m3x7c8"
ICMP
ARP/Neighbor Discovery
IP
TCP
```

Possible sequence:

```text id="j8q4v6"
Address resolution
    ↓
IP communication
    ↓
ICMP response
    ↓
TCP connection
```

Do not rely on ping alone.

A host can block ICMP while still accepting application traffic.

---

## DNS Security Investigation

DNS can provide useful security evidence.

Investigate:

```text id="s2k6m9"
High query frequency
Repeated failed lookups
Unexpected domains
Unusual subdomain patterns
Unexpected DNS servers
Rapidly changing answers
Queries followed by connections
```

A useful workflow is:

```text id="f7c3x8"
Host
    ↓
DNS queries
    ↓
Names
    ↓
Frequency
    ↓
Responses
    ↓
Resolved addresses
    ↓
Subsequent connections
```

The final interpretation must account for legitimate software behavior.

---

## ICMP Security Investigation

ICMP traffic can be useful during security investigations.

Look for:

```text id="x5m9q2"
Unexpected ICMP volume
Unusual destinations
Unexpected ICMP types
Repeated errors
Unexpected routing-related messages
```

But ICMP is a normal part of networking.

Investigate the pattern and context rather than treating ICMP itself as suspicious.

---

## DNS Filtering Workflow

Start:

```text id="z8c4m1"
dns
```

Then:

```text id="p5x7n3"
dns && ip.src == 192.168.1.10
```

Then:

```text id="m9q2v6"
dns && ip.src == 192.168.1.10 && dns.flags.response == 0
```

Then narrow to a query name:

```text id="r3k8c5"
dns && ip.src == 192.168.1.10 && dns.qry.name == "example.com"
```

At each stage ask:

```text id="w7n4p2"
Did the result change as expected?
```

---

## DHCP Filtering Workflow

Start:

```text id="q2m8x5"
dhcp
```

Then identify:

```text id="f6c3n9"
Discover
Offer
Request
ACK
```

Then focus on a client using the appropriate DHCP fields discovered in the packet details pane.

Do not build a large DHCP filter before understanding the exchange.

---

## ICMP Filtering Workflow

Start:

```text id="k4x9m2"
icmp
```

Then identify the message type.

For example:

```text id="n7c3p5"
Echo Request
```

or:

```text id="v2m8q6"
Echo Reply
```

Then narrow by:

```text id="r5x1c9"
Source
Destination
Type
Code
Timing
```

Use the same approach for ICMPv6:

```text id="s8q4m7"
icmpv6
```

---

## Practical Exercise 1 — DNS Query and Response

Find one DNS query.

Record:

```text id="x6m2p8"
Packet number
Client
DNS server
Query name
Query type
Timestamp
```

Then find its response.

Record:

```text id="n4c7r1"
Response packet
Response code
Answer
Timing
```

Explain whether the resolution succeeded.

---

## Practical Exercise 2 — DNS Failure

Find a DNS response with a failure status.

Determine:

```text id="q8m3v6"
Query name
Client
Server
Response code
Timing
Repeated attempts
```

Then determine what application behavior followed.

---

## Practical Exercise 3 — DNS Repetition

Choose one client.

Find a name it queried multiple times.

Record:

```text id="j5x9c2"
Query name
Number of queries
Time intervals
Responses
Response codes
```

Explain possible reasons for the repeated behavior without assuming maliciousness.

---

## Practical Exercise 4 — DNS-to-Connection Correlation

Find:

```text id="p3n8v5"
DNS Query
DNS Response
```

Then identify the returned address.

Search for subsequent traffic to that address.

Determine:

```text id="m7c2x9"
Did the client communicate with the resolved address?
How soon?
Using which protocol?
```

---

## Practical Exercise 5 — DHCP DORA

Find a complete DHCP exchange.

Identify:

```text id="r4x8m2"
Discover
Offer
Request
ACK
```

Record:

```text id="q6c3n9"
Client
DHCP server
Offered IP
Requested IP
Final IP
DNS server
Gateway
Lease information
```

Then inspect the first IP traffic after DHCP completion.

---

## Practical Exercise 6 — DHCP Failure

Find or create a DHCP exchange that does not complete normally.

Determine:

```text id="v8m2p5"
Which message is missing?
Are retries visible?
Is another DHCP server responding?
Is the client broadcasting?
Could the capture point explain the missing message?
```

---

## Practical Exercise 7 — ICMP Ping

Find several Echo Request/Reply pairs.

Record:

```text id="n5c7x1"
Request packet
Reply packet
Sequence
Source
Destination
Request timestamp
Reply timestamp
Approximate RTT
```

Compare multiple requests.

---

## Practical Exercise 8 — ICMP Error

Find an ICMP error message.

Identify:

```text id="q2m9v4"
ICMP type
ICMP code
Source
Destination
Quoted original packet
```

Then inspect the original packet.

Explain the relationship between the original communication and the ICMP error.

---

## Practical Exercise 9 — IPv6 Neighbor Discovery

If IPv6 traffic is available, locate:

```text id="f6x3m8"
Neighbor Solicitation
Neighbor Advertisement
```

Record:

```text id="r9c5p2"
Source
Target
Type
Options
Timing
```

Then identify the IPv6 communication that followed.

---

## Practical Exercise 10 — Application Troubleshooting

Investigate:

```text id="z4m8q1"
"The application cannot reach the server by hostname."
```

Follow:

```text id="c7x2n5"
DNS query
    ↓
DNS response
    ↓
Resolved address
    ↓
TCP/UDP attempt
    ↓
Application protocol
```

Determine where the failure appears.

---

## Practical Exercise 11 — Layered Connectivity Investigation

Investigate:

```text id="p5n7c3"
"Client cannot reach server."
```

Check:

```text id="m2x8q6"
ARP / Neighbor Discovery
    ↓
ICMP if relevant
    ↓
IP
    ↓
TCP/UDP
    ↓
Application
```

Document:

```text id="r8c4v1"
Observed
Interpretation
Uncertainty
```

---

## Practical Exercise 12 — Build a Protocol Timeline

Choose one client and build a timeline containing:

```text id="k6m3x9"
DHCP
DNS
ICMP
TCP/UDP
Application
```

Example:

```text id="t7p2n5"
10:01:01 DHCP ACK
10:01:05 DNS query
10:01:05 DNS response
10:01:06 TCP SYN
10:01:06 TCP SYN-ACK
10:01:06 ACK
10:01:07 Application request
```

The exact sequence will depend on the capture.

The goal is to understand how protocols interact over time.

---

## Common Mistakes

### Mistake 1 — Treating DNS as Only Name-to-IP

DNS supports many record types and functions.

### Mistake 2 — Treating NXDOMAIN as Automatically Malicious

A failed lookup can be completely legitimate.

### Mistake 3 — Treating Missing DNS Response as Proof of Packet Loss

Capture visibility and application behavior matter.

### Mistake 4 — Treating DHCP as a Single Transactionless Packet

DHCP is normally an exchange involving multiple messages.

### Mistake 5 — Assuming DHCP Always Uses the Same Exact Sequence

The environment and client state can affect the exchange.

### Mistake 6 — Treating ICMP as Only Ping

ICMP includes control and error messages.

### Mistake 7 — Assuming Ping Failure Means Host Failure

ICMP may be filtered while application traffic remains available.

### Mistake 8 — Ignoring IPv6

IPv6 uses different addressing and neighbor-discovery mechanisms.

### Mistake 9 — Looking at DNS Without Correlating Subsequent Traffic

The important question may be what happened after resolution.

---

## DNS Investigation Checklist

```text id="v4c8m2"
[ ] Did I identify the client?
[ ] Did I identify the DNS server?
[ ] Did I identify the query name?
[ ] Did I identify the query type?
[ ] Did I identify the response?
[ ] Did I check the response code?
[ ] Did I inspect returned records?
[ ] Did I measure timing?
[ ] Did I check retries?
[ ] Did I check multiple DNS servers?
[ ] Did I correlate the result with subsequent communication?
```

---

## DHCP Investigation Checklist

```text id="j7m3x9"
[ ] Did I identify the client?
[ ] Did I identify DHCP servers?
[ ] Did I find Discover?
[ ] Did I find Offer?
[ ] Did I find Request?
[ ] Did I find ACK?
[ ] Did I inspect requested/offered addresses?
[ ] Did I inspect configuration options?
[ ] Did I check retries?
[ ] Did I check multiple servers?
[ ] Did I correlate DHCP with subsequent IP traffic?
```

---

## ICMP and ICMPv6 Investigation Checklist

```text id="x5c8m1"
[ ] Did I identify the ICMP type?
[ ] Did I identify the code?
[ ] Did I identify source and destination?
[ ] Did I inspect the related packet?
[ ] Did I check timing?
[ ] Did I distinguish Echo from error traffic?
[ ] Did I consider filtering?
[ ] Did I consider routing?
[ ] Did I consider capture visibility?
[ ] For IPv6, did I check Neighbor Discovery where relevant?
```

---

## Professional Workflow

For DNS:

```text id="m9x2c7"
Client
    ↓
Query
    ↓
DNS server
    ↓
Response
    ↓
Status
    ↓
Answer
    ↓
Subsequent connection
```

For DHCP:

```text id="p4c8n1"
Client
    ↓
Discover
    ↓
Offer
    ↓
Request
    ↓
ACK
    ↓
IP communication
```

For ICMP:

```text id="r7m3x5"
Message
    ↓
Type/code
    ↓
Related traffic
    ↓
Timing
    ↓
Network interpretation
```

For IPv6 Neighbor Discovery:

```text id="v6x2q9"
Neighbor/Router message
    ↓
ICMPv6 type
    ↓
Address information
    ↓
Response
    ↓
IPv6 communication
```

---

## Completion Criteria

You are ready to continue when you can independently:

* Identify DNS queries and responses.
* Distinguish DNS queries from responses.
* Identify query names and types.
* Interpret common DNS response outcomes.
* Analyze DNS timing and retries.
* Correlate DNS results with subsequent connections.
* Analyze DHCP Discover, Offer, Request, and ACK.
* Investigate DHCP failures and multiple-server behavior.
* Recognize DHCP configuration information.
* Analyze ICMP Echo traffic.
* Calculate approximate request/reply timing.
* Investigate ICMP error messages.
* Understand the importance of ICMP type and code.
* Recognize ICMPv6 Neighbor Discovery.
* Analyze Router Solicitation and Router Advertisement at a basic level.
* Use DNS, DHCP, and ICMP evidence during connectivity troubleshooting.
* Correlate lower-layer protocols with application behavior.
* Distinguish observations from interpretations.
* Account for capture limitations and protocol-specific behavior.

Once these workflows are comfortable, you can move upward into HTTP, TLS, and other application-layer traffic where protocol semantics become central to troubleshooting and security analysis.
