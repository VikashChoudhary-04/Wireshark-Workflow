# Connectivity Failures

## Objective

Connectivity troubleshooting is one of the most practical uses of Wireshark.

The goal is not to memorize filters or identify a single packet that "looks wrong."

The goal is to determine:

* What the user or application expected to happen
* What actually happened on the wire
* Where the expected communication stopped
* Which layer failed first
* What evidence supports that conclusion
* What remains uncertain because of the capture point or available evidence

The core workflow is:

```text
Define the connectivity problem
        ↓
Identify client, target, protocol, and time window
        ↓
Check whether traffic was generated
        ↓
Check local addressing and neighbor resolution
        ↓
Check DNS when a hostname is involved
        ↓
Check routing/reachability clues
        ↓
Check TCP or UDP communication
        ↓
Check TLS when applicable
        ↓
Check application behavior
        ↓
Find the earliest supported failure
        ↓
Compare with a successful case
        ↓
Document evidence and remaining uncertainty
```

Wireshark shows packets that reached the capture point. It does not automatically show everything that happened elsewhere in the network.

That distinction is critical.

## The Connectivity Failure Mental Model

A connectivity problem is usually described from the user's perspective:

```text
"I cannot connect."
"The website does not open."
"The server is unreachable."
"The application times out."
"SSH is not working."
"Only this hostname fails."
"The connection works sometimes."
```

These statements are symptoms, not diagnoses.

Translate the statement into a packet-level question.

For example:

```text
"I cannot connect to the server."

Possible packet questions:

- Did the client generate traffic?
- Was the destination resolved?
- Did ARP or IPv6 neighbor discovery succeed?
- Did packets leave the client?
- Did the destination respond?
- Did a gateway or intermediate device respond?
- Did TCP complete its handshake?
- Was the connection reset?
- Did TLS begin?
- Did the application send a request?
```

The investigation should answer these questions in order rather than immediately assuming a firewall, routing problem, server failure, or DNS problem.

## First Define the Problem

Before opening filters, establish the basic facts.

Record:

| Item                  | Question                                     |
| --------------------- | -------------------------------------------- |
| Client                | Which host initiated the connection?         |
| Target                | Which host or service was being contacted?   |
| Source address        | Which IPv4 or IPv6 address was used?         |
| Destination address   | Which address was contacted?                 |
| Protocol              | DNS, TCP, UDP, TLS, HTTP, SSH, etc.          |
| Port                  | Which service port was involved?             |
| Time                  | When did the failure occur?                  |
| Expected result       | What should have happened?                   |
| Actual result         | What happened instead?                       |
| Capture point         | Where was Wireshark capturing traffic?       |
| Successful comparison | Is there a known-good connection to compare? |

Without this context, packet analysis can easily become disconnected from the actual problem.

## What Does "Cannot Connect" Mean?

"Cannot connect" can represent several completely different packet patterns.

### DNS failure

```text
Client → DNS server: query
DNS server → Client: failure / no useful response
```

The application may never receive an address for the target.

### ARP or neighbor-resolution failure

```text
Client → local network: address-resolution request
Client ← expected neighbor response: missing
```

The client may not be able to reach the next-hop device or local destination.

### TCP connection failure

```text
Client → Server: SYN
Client ← Server: no SYN/ACK
```

The TCP connection never becomes established.

### TCP reset

```text
Client → Server: SYN
Server → Client: RST
```

A reset indicates that a TCP endpoint or device generated a reset. The exact reason still requires investigation.

### TLS failure

```text
TCP handshake succeeds
        ↓
TLS handshake begins
        ↓
TLS negotiation fails or stops
```

Basic network connectivity exists, but the secure session does not complete.

### Application failure

```text
TCP succeeds
TLS succeeds
Application request succeeds
        ↓
Application returns an error
```

The network path may be functioning even though the user sees an application failure.

## The Layer-by-Layer Connectivity Workflow

A useful troubleshooting order is:

```text
1. Capture visibility
2. Local addressing
3. ARP / IPv6 Neighbor Discovery
4. DHCP when relevant
5. DNS when a hostname is involved
6. IP reachability clues
7. TCP / UDP
8. TLS
9. Application protocol
```

Do not blindly inspect every layer if the evidence already establishes that a later layer was reached.

For example:

```text
TCP handshake completed
        ↓
TLS ClientHello observed
        ↓
HTTP request observed
```

There is little value in continuing to investigate whether the client could establish basic IP connectivity.

Instead, move upward to the layer where the behavior diverges.

## Step 1: Confirm Capture Visibility

Before diagnosing the network, verify that the capture can actually see the relevant traffic.

Ask:

* Was the correct interface selected?
* Was the capture running at the right time?
* Is the client visible?
* Is the expected protocol visible?
* Is the capture point capable of observing both directions?
* Is traffic being encrypted or tunneled?
* Could the traffic be bypassing the capture point?

A completely empty capture does not automatically mean the network is broken.

It may mean:

```text
Wrong interface
Wrong capture point
Capture started too late
Traffic never occurred
Traffic uses another interface
Permissions prevented capture
Traffic is outside the visible path
```

Start with evidence about capture visibility before making network conclusions.

## Step 2: Identify the Client and Target

Find the initiating host and destination.

Useful fields include:

```text
ip.src
ip.dst
ipv6.src
ipv6.dst
eth.src
eth.dst
```

For a known client:

```text
ip.addr == 192.0.2.10
```

For a known destination:

```text
ip.addr == 192.0.2.50
```

For IPv6:

```text
ipv6.addr == 2001:db8::10
```

These examples use documentation addresses.

The important principle is to narrow the capture to the communication under investigation.

## Step 3: Check Local Addressing

Before investigating remote connectivity, determine whether the client appears correctly addressed.

Look for:

* DHCP exchanges
* ARP activity
* IPv6 Neighbor Discovery
* Duplicate-address indications
* Unexpected source addresses
* Incorrect gateways where visible
* Repeated address-resolution attempts

A client with an incorrect address or missing gateway may fail before any remote TCP connection is possible.

## Step 4: Analyze ARP

ARP is relevant to IPv4 local-network communication.

A typical exchange is:

```text
Who has 192.0.2.1?
192.0.2.1 is at aa:bb:cc:dd:ee:ff
```

The important question is not merely whether ARP exists.

Ask:

```text
Did the client request the expected address?
Did anyone answer?
Was the answer repeated?
Did the MAC address change unexpectedly?
Does the client repeatedly ask for the same address?
```

A request without an observed response can indicate a problem, but the capture point matters.

If Wireshark is capturing on a host that cannot see traffic from the relevant network segment, the absence of a response is not proof that no response existed elsewhere.

## ARP Failure Pattern

A possible pattern is:

```text
Client → ARP request
Client → ARP request
Client → ARP request
Client → ARP request
```

with no corresponding response.

Possible hypotheses include:

* Target is offline
* Target is not on the expected local network
* VLAN or switching problem
* Incorrect local configuration
* Address-resolution filtering
* Capture visibility limitation

Do not automatically conclude which one is responsible.

Use additional evidence.

## Duplicate IP and ARP Anomalies

Unexpected ARP behavior can sometimes indicate an addressing problem.

Investigate when:

* Multiple MAC addresses appear associated with the same IPv4 address
* ARP announcements change unexpectedly
* The expected gateway mapping changes
* Traffic alternates between different hardware addresses

Possible explanations include:

* Duplicate IP address
* Failover or redundancy
* Virtualization
* Load balancing
* Proxy ARP
* Legitimate network changes
* Misconfiguration

ARP evidence should be interpreted in network context.

## IPv6 Neighbor Discovery

IPv6 uses Neighbor Discovery rather than ARP.

Relevant traffic can include:

```text
Neighbor Solicitation
Neighbor Advertisement
Router Solicitation
Router Advertisement
```

When IPv6 is involved, determine:

```text
Is the client using IPv6?
Is a router advertisement visible?
Is neighbor discovery succeeding?
Is the client attempting IPv6 before IPv4?
```

A connectivity problem may appear to be an application failure when the application is repeatedly attempting an IPv6 path that is not functioning as expected.

## Step 5: Check DHCP When Relevant

DHCP matters primarily when the client is obtaining configuration dynamically.

A simplified DHCP exchange is:

```text
Discover
Offer
Request
ACK
```

The investigation question is:

```text
Did the client receive the configuration required to communicate?
```

Look for:

* DHCP Discover
* DHCP Offer
* DHCP Request
* DHCP ACK
* Repeated Discover messages
* Missing expected responses
* Unexpected DHCP servers

If the client already has valid static configuration or the failure occurs after normal connectivity has been established, DHCP may not be relevant to the current incident.

Do not force every investigation through DHCP.

## Step 6: Separate DNS From Connectivity

A hostname failure does not automatically mean the network path is broken.

For example:

```text
Application requests:
example.test

DNS resolution fails

No TCP connection follows
```

In this case, the application cannot connect because it does not have a usable destination address.

Compare that with:

```text
DNS query succeeds
        ↓
Destination address returned
        ↓
TCP SYN sent
        ↓
No response
```

Now DNS succeeded and the investigation should move to connectivity.

A useful sequence is:

```text
Hostname
   ↓
DNS resolution
   ↓
IP address
   ↓
Network reachability
   ↓
Transport connection
   ↓
Secure session
   ↓
Application transaction
```

Do not label a problem as "DNS" merely because a hostname appears in the user's complaint.

## DNS Failure Patterns

### No DNS response

```text
Client → DNS server: query
Client → DNS server: repeated query
Client → DNS server: repeated query
```

Possible hypotheses:

* DNS server unavailable
* Network path problem
* DNS filtering
* Wrong DNS server
* Capture visibility limitation

### DNS response with an error

A DNS response can provide stronger evidence than a missing response.

For example, the server may explicitly return an error status.

The next question is:

```text
Did the application treat that result as a connection failure?
```

### DNS succeeds but application fails

This is an important distinction.

If:

```text
DNS succeeds
        ↓
TCP fails
```

the DNS transaction itself does not explain the failure.

Move to TCP analysis.

## Step 7: Check IP Reachability Clues

Wireshark may provide evidence about network-layer behavior.

Look for:

* ICMP errors
* ICMPv6 errors
* TTL or hop-limit behavior
* Routing-related ICMP messages
* Repeated packets without responses
* Gateway-related traffic

A relevant ICMP message can be highly informative.

For example:

```text
Destination Unreachable
```

may indicate that a packet could not be delivered.

However, the exact ICMP code matters.

Do not treat every ICMP unreachable message as equivalent.

## ICMP Time Exceeded

An ICMP Time Exceeded message can indicate that a packet's TTL or IPv6 hop limit expired.

A simplified pattern is:

```text
Client → network → ...
        ↓
ICMP Time Exceeded
```

This can provide a clue about routing behavior or a packet traversing too many hops.

It does not by itself establish the complete cause of the application failure.

## Gateway and Routing Clues

When traffic is destined for a remote network, the client normally sends traffic toward its next hop.

Evidence may include:

```text
Client → gateway MAC
Gateway → remote destination
```

At the client capture point, you may not see the entire route.

That limitation matters.

Wireshark can often tell you:

```text
The client generated the packet.
```

It may not tell you:

```text
Exactly where the packet disappeared after leaving the capture point.
```

This is why capture placement is part of the diagnosis.

## Step 8: Analyze TCP

For TCP services, inspect the connection establishment first.

The normal three-way handshake is:

```text
Client → Server: SYN
Server → Client: SYN, ACK
Client → Server: ACK
```

Successful establishment generally gives you:

```text
SYN
SYN/ACK
ACK
```

Then application traffic can begin.

## TCP Pattern: SYN With No SYN/ACK

Example:

```text
Client → Server: SYN
Client → Server: SYN
Client → Server: SYN
Client → Server: SYN
```

Possible hypotheses include:

* Destination unavailable
* Packet filtering
* Routing problem
* Server not reachable
* Service path failure
* Return path problem
* Capture point does not see the response

The key observation is:

```text
SYN generated
No SYN/ACK observed
```

Do not automatically state:

```text
"The server is down."
```

The capture may not be positioned where the server's response would be visible.

## TCP Pattern: SYN Followed by RST

Example:

```text
Client → Server: SYN
Server → Client: RST, ACK
```

This demonstrates that a reset was sent.

Depending on context, possible explanations include:

* No listening service
* Active rejection
* Host or security device behavior
* Application or stack behavior
* Middlebox intervention

The packet itself proves the reset was transmitted from the observed source address.

It does not always prove why it was generated.

## TCP Pattern: Handshake Completes but Application Fails

Example:

```text
SYN
SYN/ACK
ACK
PSH/ACK
PSH/ACK
...
```

This establishes that the TCP connection was created.

If the application still fails, move upward.

Possible next layers:

```text
TLS
HTTP
SSH
SMB
LDAP
Database protocol
Custom application protocol
```

Do not continue describing the incident as a basic connectivity failure when packet evidence shows that transport connectivity succeeded.

## TCP Pattern: Connection Established Then Reset

Example:

```text
SYN
SYN/ACK
ACK
Application traffic
RST
```

This means the connection existed before a reset occurred.

That is materially different from:

```text
SYN
RST
```

Investigate what happened immediately before the reset.

Questions include:

* Who sent the reset?
* What application message preceded it?
* Did TLS fail?
* Did the server reject an invalid request?
* Did a middlebox intervene?
* Did the connection encounter an application-level error?

## UDP Connectivity

UDP does not have a TCP-style handshake.

Therefore:

```text
UDP packet sent
No UDP response
```

does not automatically prove failure.

Possible explanations include:

* Application intentionally does not respond
* Packet was lost
* Response was filtered
* Response took another path
* Service is unavailable
* Capture does not include the response

For UDP, application behavior and protocol semantics become particularly important.

## Step 9: Check TLS

If TCP succeeds and the application uses TLS, inspect the TLS handshake.

A simplified sequence may be:

```text
TCP handshake
        ↓
TLS ClientHello
        ↓
TLS ServerHello
        ↓
Certificate / key exchange messages
        ↓
Encrypted application traffic
```

If the TCP handshake succeeds but TLS stops, the failure has moved above basic network connectivity.

Investigate:

* Whether a ClientHello was sent
* Whether a ServerHello was observed
* TLS version information
* Cipher-suite negotiation
* Certificate-related information
* Alerts
* Timing
* Which side stopped transmitting

Encrypted traffic may prevent inspection of application content, but packet metadata and handshake behavior can still provide useful evidence.

## Step 10: Check the Application Protocol

Once TCP and, when applicable, TLS are established, inspect application behavior.

For HTTP, look for:

```text
Request
Response
Status code
Redirect
Headers
Timing
Connection termination
```

For SSH:

```text
TCP establishment
SSH protocol exchange
Authentication-related exchange
Session behavior
```

For SMB, LDAP, Kerberos, SMTP, or other protocols, use the protocol's visible transaction structure.

The question becomes:

```text
Did the application protocol behave as expected?
```

## Port Closed vs Filtered vs Unreachable

Packet evidence can sometimes distinguish different failure patterns, but these labels must be used carefully.

### Possible closed-port behavior

```text
TCP SYN
      ↓
TCP RST
```

This may indicate that the destination host or another device rejected the connection.

### Possible filtering behavior

```text
TCP SYN
      ↓
No response
```

This can be consistent with filtering, but it can also result from:

* Host being offline
* Routing failure
* Return-path failure
* Packet loss
* Capture limitations

### Possible unreachable behavior

```text
Packet
  ↓
ICMP Destination Unreachable
```

The ICMP type and code provide important context.

The professional approach is:

```text
Observed packet pattern
        ↓
Supported interpretation
        ↓
Possible hypotheses
```

not:

```text
Packet pattern
        ↓
Automatic diagnosis
```

## Distinguish Client, Server, and Middlebox Evidence

A common troubleshooting mistake is assigning every observed packet to the endpoint that the analyst expects.

Suppose:

```text
Client → Server: SYN
Server-looking address → Client: RST
```

The source address may appear to belong to the server.

But the packet could potentially be generated by an intermediate device using that address or acting on behalf of the endpoint.

Therefore ask:

```text
Where was the packet captured?
What does the Ethernet layer show?
Is there a known middlebox?
Does the behavior occur consistently?
Is there a successful comparison?
```

Capture location determines what you can confidently attribute.

## Firewall and Security Controls

Firewalls, ACLs, security groups, WAFs, proxies, and other security controls can affect connectivity.

But they should be treated as hypotheses until packet evidence supports them.

For example:

```text
SYN leaves client
No SYN/ACK observed
```

Possible hypotheses include:

```text
Firewall filtering
Routing failure
Server unavailable
Incorrect destination
Return-path problem
Capture visibility limitation
```

Additional evidence is needed to distinguish them.

A useful troubleshooting statement is:

> "The client repeatedly transmitted TCP SYN packets, but no corresponding SYN/ACK was observed at the capture point."

That is stronger than:

> "The firewall blocked the connection."

The first statement is directly supported by packet evidence.

## Successful vs Failed Comparison

A known-good connection is often one of the strongest troubleshooting tools.

Compare:

```text
Successful:
DNS
SYN
SYN/ACK
ACK
TLS
Application request
Response

Failed:
DNS
SYN
SYN
SYN
...
```

The divergence identifies where the behaviors separate.

This is often more useful than analyzing the failed capture in isolation.

Compare:

* Same client
* Same destination
* Same service
* Same protocol
* Similar time period
* Similar network path when possible

Change one variable at a time when possible.

## Find the Earliest Failure

When several symptoms exist, identify the earliest meaningful divergence.

Example:

```text
DNS succeeds
↓
SYN transmitted
↓
No SYN/ACK
↓
Application timeout
```

The application timeout is a consequence.

The earliest observed failure is the missing TCP response.

Another example:

```text
TCP succeeds
↓
TLS ClientHello
↓
TLS alert
↓
Application timeout
```

The investigation should focus on the TLS stage rather than basic connectivity.

The principle is:

```text
Find the earliest unsupported transition in the expected workflow.
```

## Connectivity Troubleshooting Decision Tree

Use this as a practical mental model:

```text
START
  |
  v
Is relevant traffic visible?
  |
  +-- NO --> Verify interface, timing, capture point, and traffic generation
  |
  +-- YES
        |
        v
Is addressing/neighbor resolution relevant?
        |
        +-- Failure --> Investigate ARP / ND / DHCP / local configuration
        |
        +-- Success
              |
              v
Is hostname resolution required?
              |
              +-- Failure --> Investigate DNS
              |
              +-- Success / not required
                    |
                    v
Is IP reachability supported by the capture?
                    |
                    +-- Failure evidence --> Investigate ICMP/routing/path clues
                    |
                    +-- Unclear
                          |
                          v
Does TCP/UDP communication begin?
                          |
                          +-- No --> Investigate transport-level behavior
                          |
                          +-- Yes
                                |
                                v
Does TCP establish?
                                |
                                +-- No --> Analyze SYN / SYN-ACK / RST / retransmission
                                |
                                +-- Yes
                                      |
                                      v
Does TLS establish when required?
                                      |
                                      +-- No --> Analyze TLS handshake/alerts
                                      |
                                      +-- Yes
                                            |
                                            v
Does the application transaction succeed?
                                            |
                                            +-- No --> Analyze application protocol
                                            |
                                            +-- Yes --> Connectivity established
```

This is not a rigid sequence.

If evidence immediately identifies the relevant layer, move there.

## Practical Investigation 1: Basic TCP Timeout

### Scenario

A user reports:

```text
"I cannot connect to the server."
```

Known information:

```text
Client: 192.0.2.10
Server: 192.0.2.50
Port: 443
```

### Investigation

Start with:

```text
ip.addr == 192.0.2.10 && ip.addr == 192.0.2.50
```

Then identify TCP traffic involving the service.

Look for:

```text
SYN
SYN/ACK
ACK
```

Suppose the capture shows:

```text
192.0.2.10 → 192.0.2.50:443  SYN
192.0.2.10 → 192.0.2.50:443  SYN
192.0.2.10 → 192.0.2.50:443  SYN
```

No SYN/ACK is observed.

### Evidence

```text
Client generated repeated SYN packets.
No SYN/ACK was observed at the capture point.
```

### Interpretation

The TCP connection was not observed becoming established.

### Next questions

```text
Is the server reachable?
Is traffic being filtered?
Is the service listening?
Is the return path working?
Is the capture point capable of seeing the response?
```

Do not select one explanation without additional evidence.

## Practical Investigation 2: DNS Looks Like Connectivity Failure

### Scenario

A user reports:

```text
"The website is down."
```

The capture shows:

```text
DNS query
DNS response
```

The response does not provide the expected usable address.

No TCP connection follows.

### Reasoning

The application never reached the web server.

The evidence indicates that the failure occurred during name resolution or before the connection attempt.

The correct next step is DNS investigation, not TCP retransmission analysis.

## Practical Investigation 3: TCP Works but TLS Fails

### Scenario

A user reports that HTTPS does not work.

The capture shows:

```text
SYN
SYN/ACK
ACK
ClientHello
TLS alert
Connection termination
```

### Reasoning

The network successfully established TCP.

The failure occurred after transport establishment.

Investigate:

* TLS versions
* Negotiation
* Certificate information
* Alerts
* Timing
* Client/server behavior

Do not describe the incident simply as "the server is unreachable."

## Practical Investigation 4: Successful vs Failed Connection

Capture two attempts:

```text
Attempt A: successful
Attempt B: failed
```

Compare the sequences.

Successful:

```text
DNS
SYN
SYN/ACK
ACK
TLS
Application request
Response
```

Failed:

```text
DNS
SYN
SYN
SYN
```

The divergence occurs after the SYN.

This dramatically narrows the investigation.

## Practical Investigation 5: Reset After Connection

Suppose the sequence is:

```text
SYN
SYN/ACK
ACK
Application traffic
RST
```

Do not treat the reset as equivalent to a failed connection attempt.

The connection was established.

Investigate the traffic immediately before the reset and determine:

```text
Who sent the RST?
What message preceded it?
Was there an application error?
Was there a protocol violation?
Was a middlebox involved?
```

## Practical Exercise Set

Use authorized lab traffic, your own systems, or supplied PCAPs.

### Exercise 1 — Identify the Failure Layer

Find a failed connection and determine:

```text
Where does the expected workflow stop?
```

Record:

```text
Expected:
Observed:
First divergence:
Evidence:
```

### Exercise 2 — SYN Timeout

Find a TCP connection where:

```text
SYN
SYN
SYN
...
```

is visible.

Determine:

* Source
* Destination
* Port
* Number of attempts
* Time between attempts
* Whether any response exists

### Exercise 3 — TCP Reset

Find a connection containing:

```text
SYN
RST
```

Then find another containing:

```text
SYN
SYN/ACK
ACK
...
RST
```

Explain why the two patterns represent different stages of failure.

### Exercise 4 — DNS vs TCP

Find one transaction where:

```text
DNS succeeds → TCP begins
```

and another where:

```text
DNS fails → TCP does not begin
```

Explain the difference.

### Exercise 5 — ICMP Evidence

Find an ICMP error associated with another packet.

Record:

```text
Original traffic:
ICMP type/code:
Source:
Destination:
Potential implication:
Remaining uncertainty:
```

### Exercise 6 — ARP Investigation

Find an ARP exchange.

Determine:

```text
Who requested the address?
Who answered?
What MAC address was returned?
Was the exchange repeated?
```

### Exercise 7 — IPv6 Neighbor Discovery

Find an IPv6 Neighbor Solicitation and corresponding Neighbor Advertisement.

Explain:

```text
Who asked?
What address was being resolved?
Who answered?
What evidence shows the exchange succeeded?
```

### Exercise 8 — TLS Failure

Find a TLS connection that does not progress normally.

Determine:

```text
Did TCP establish?
Was ClientHello visible?
Was a ServerHello visible?
Was a TLS alert visible?
Which side stopped progressing?
```

### Exercise 9 — Successful Comparison

Find:

```text
One successful connection
One failed connection
```

Compare them side by side.

Identify the first meaningful divergence.

### Exercise 10 — Capture Limitation

Find a situation where you cannot determine the complete cause from the available capture.

Document:

```text
What is proven?
What is not visible?
What additional capture point or evidence would help?
```

This is an important professional skill.

## Evidence Record

For every serious connectivity investigation, maintain a small evidence record.

```text
Incident:
Date/Time:
Client:
Destination:
Protocol:
Port:

User symptom:

Expected behavior:

Observed behavior:

Capture point:

Relevant packets:

Earliest observed failure:

Evidence:

Interpretation:

Possible hypotheses:

What has been ruled out:

What remains uncertain:

Next evidence required:
```

This prevents conclusions from becoming stronger than the packet evidence.

## Common Mistakes

### Mistake 1: Starting With a Favorite Filter

Do not begin with:

```text
"Which filter should I use?"
```

Begin with:

```text
"What should happen next?"
```

Then build the filter that exposes that evidence.

### Mistake 2: Assuming No Response Means Firewall

A missing response can have many causes.

Always consider:

```text
Routing
Server availability
Filtering
Return path
Packet loss
Capture visibility
```

### Mistake 3: Treating DNS and Connectivity as the Same Problem

DNS determines an address.

It does not establish the subsequent TCP or application connection.

Keep the stages separate.

### Mistake 4: Ignoring Capture Location

A packet missing from a host capture is not necessarily missing from the network.

Capture placement determines what conclusions are possible.

### Mistake 5: Calling Every RST a Firewall

A reset is a packet event.

Determine who sent it and what happened around it before proposing a cause.

### Mistake 6: Ignoring Successful Connections

A successful comparison can reveal the failure point much faster than staring at the failed connection alone.

### Mistake 7: Jumping to Application Conclusions Too Early

If TCP never establishes, do not spend most of the investigation analyzing HTTP behavior that never occurred.

### Mistake 8: Treating UDP Like TCP

UDP has no three-way handshake.

Absence of a response requires protocol-specific interpretation.

### Mistake 9: Ignoring IPv6

If the host is attempting IPv6, investigate that path instead of assuming all traffic is IPv4.

### Mistake 10: Confusing Observation With Diagnosis

Prefer:

```text
"No SYN/ACK was observed."
```

over:

```text
"The firewall blocked the SYN/ACK."
```

unless additional evidence actually establishes the latter.

## Professional Connectivity Workflow

A mature investigation should look like this:

```text
1. Define the user-visible symptom.
2. Identify client, target, protocol, and time.
3. Confirm the relevant traffic is visible.
4. Establish the expected communication sequence.
5. Follow the sequence packet by packet.
6. Identify the first meaningful divergence.
7. Determine which layer owns that divergence.
8. Check whether the capture point can support the conclusion.
9. Compare against a successful transaction when available.
10. Form hypotheses from the evidence.
11. Identify what additional evidence would distinguish them.
12. Document the result without overstating certainty.
```

The objective is not merely:

```text
"Find the bad packet."
```

The objective is:

```text
"Determine where the expected communication stopped,
what the packet evidence proves,
and what remains unknown."
```

## Connectivity Troubleshooting Checklist

Before closing an investigation, verify:

* [ ] Client identified
* [ ] Destination identified
* [ ] Protocol identified
* [ ] Port identified when applicable
* [ ] Time window identified
* [ ] Capture point understood
* [ ] Relevant traffic confirmed visible
* [ ] Expected sequence established
* [ ] ARP/ND considered when relevant
* [ ] DHCP considered when relevant
* [ ] DNS checked when hostname resolution matters
* [ ] ICMP/ICMPv6 evidence checked
* [ ] TCP handshake analyzed when applicable
* [ ] UDP behavior interpreted according to the protocol
* [ ] TLS analyzed when applicable
* [ ] Application behavior analyzed when reached
* [ ] Earliest meaningful failure identified
* [ ] Successful comparison performed when available
* [ ] Capture limitations documented
* [ ] Hypotheses separated from confirmed evidence
* [ ] Next evidence required identified

## Completion Criteria

You should consider this workflow complete when you can independently take a connectivity complaint and answer:

```text
What was supposed to happen?

What actually happened?

Did the relevant traffic occur?

Where did the expected sequence stop?

Which layer does that failure belong to?

What packets prove the observation?

Could the capture point hide part of the communication?

What explanations are supported?

What explanations remain hypotheses?

What additional evidence would resolve the uncertainty?
```

You should also be able to distinguish:

```text
DNS failure
ARP/ND failure
IP reachability problem
TCP connection failure
TCP reset
UDP uncertainty
TLS failure
Application failure
Capture visibility limitation
```

without automatically treating them as the same problem.

The core skill is evidence-driven troubleshooting:

```text
Symptom
  ↓
Expected behavior
  ↓
Observed packets
  ↓
First divergence
  ↓
Evidence-based interpretation
  ↓
Next question
```

That workflow should become automatic before moving into the more specialized troubleshooting scenarios that follow.
