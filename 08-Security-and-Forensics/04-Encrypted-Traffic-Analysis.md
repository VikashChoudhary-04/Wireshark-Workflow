# Encrypted Traffic Analysis

## Objective

Encryption changes what Wireshark can directly observe.

It does not make traffic invisible.

Even when application payloads are encrypted, a packet capture can still reveal useful metadata such as:

```text
Who communicated?
When did communication occur?
Which hosts communicated?
Which ports were used?
Which protocols were visible?
How long did sessions last?
How much data moved?
How frequently did connections occur?
Which TLS versions or handshake details were visible?
What certificates or names were exposed?
Did connections succeed or fail?
```

The objective of this workflow is to analyze encrypted traffic without inventing information that the capture cannot support.

The core principle is:

```text
Visible Metadata
      ↓
Protocol Evidence
      ↓
Timing
      ↓
Connection Behavior
      ↓
Traffic Volume
      ↓
Correlation
      ↓
Evidence-Based Interpretation
```

## What Encryption Changes

Encryption primarily protects application content.

For example, an HTTPS session may hide:

```text
HTTP request body
HTTP response body
Application content
Credentials
Cookies
Some requested resource details
```

But the capture may still expose:

```text
Source IP
Destination IP
Source port
Destination port
Packet timing
Packet sizes
TCP behavior
TLS handshake
TLS version
Cipher information
Server certificate information
Server name indication where exposed
Connection duration
Session direction
Traffic volume
```

Therefore:

```text
Encrypted ≠ Invisible
```

## Define the Investigation Question

Start with a specific question.

Examples:

```text
Which internal host established an encrypted connection?

Which external destinations were contacted?

Did a TLS session succeed?

Which TLS version was negotiated?

Was the connection repeatedly established?

Which certificate information is visible?

Did multiple hosts communicate with the same destination?

Is the timing consistent with the application's expected behavior?
```

Avoid starting with:

```text
"Find something suspicious."
```

Start with an observable question.

## Establish the Scope

Record:

```text
Capture:
Time window:
Known host:
Expected service:
Known destination:
Relevant protocol:
Capture location:
```

This prevents unrelated encrypted sessions from becoming part of the investigation.

## Identify Encrypted Protocols

Start by identifying protocols visible in the capture.

Common examples include:

```text
TLS
HTTPS
SSH
IPsec
QUIC
VPN-related traffic
Encrypted application protocols
```

Do not assume that every TCP/443 session is identical.

Modern applications may use:

```text
HTTPS over TLS
HTTP/3 over QUIC
API traffic
WebSocket-based applications
CDN traffic
Cloud services
```

Protocol identification should come from the packet evidence where possible.

## TLS Investigation

TLS provides a useful metadata layer even when application content cannot be decrypted.

Begin by identifying:

```text
Client
Server
Connection
Handshake
Version
Cipher information
Certificate information
Name information
Timing
Termination
```

## TLS Handshake

A simplified TLS investigation begins with:

```text
Client
  ↓
ClientHello
  ↓
ServerHello
  ↓
Certificate / related handshake messages
  ↓
Encrypted application traffic
```

The exact handshake differs between TLS versions and configurations.

Do not expect every capture to contain the same fields.

## ClientHello

Inspect the ClientHello for available information such as:

```text
TLS version information
Supported versions
Cipher suites
Extensions
Server Name Indication
Supported groups
Signature algorithms
Application-Layer Protocol Negotiation
```

These fields can help characterize the client and intended connection.

Do not treat any single ClientHello field as proof of application identity.

## ServerHello

Inspect:

```text
Negotiated TLS version
Selected cipher suite
Extensions
Negotiated parameters
```

Compare the result with what the client offered.

## Server Name Indication

When visible, SNI can provide useful destination context.

For example:

```text
Client → destination IP
SNI → example.internal
```

This can help associate an IP address with a requested hostname.

However:

```text
SNI visibility depends on protocol version and connection configuration.
```

Do not assume that every encrypted connection exposes the hostname.

## Certificate Information

When certificates are visible in the handshake, inspect:

```text
Subject
Issuer
Validity period
Subject Alternative Names
Public-key information
Certificate chain
```

Questions to ask:

```text
Does the certificate identify the expected service?

Does the hostname appear in the certificate?

Is the certificate currently valid according to the captured metadata?

Is the issuer expected?

Are multiple hostnames present?

Is the certificate shared across multiple destinations?
```

Certificate evidence is contextual evidence, not a complete trust decision.

## Certificate and SNI Comparison

A useful correlation is:

```text
Destination IP
      ↓
SNI
      ↓
Certificate names
```

For example:

```text
Destination:
203.0.113.10

SNI:
api.example.test

Certificate SAN:
api.example.test
```

This creates a coherent relationship.

If they differ:

```text
Destination:
203.0.113.10

SNI:
service.example.test

Certificate:
*.another.example
```

investigate further.

Possible explanations include:

* CDN
* Reverse proxy
* Shared hosting
* Load balancer
* Multi-domain certificate
* Application architecture
* Misconfiguration

A mismatch is not automatically malicious.

## ALPN

Application-Layer Protocol Negotiation can provide useful protocol context.

Look for negotiated or offered protocols such as:

```text
h2
http/1.1
h3
```

Use this information to understand what application protocol may operate over the encrypted session.

Do not confuse:

```text
TLS
```

with:

```text
HTTP
```

TLS provides encryption.

The application protocol runs above it.

## TLS Version Analysis

Record visible TLS versions.

Common examples include:

```text
TLS 1.2
TLS 1.3
```

Older versions may also appear in historical captures.

Do not judge a connection solely from the version number.

Interpret the version in the context of:

```text
Application
Server
Client
Capture date
Environment
Security requirements
```

## Cipher Information

Cipher information can help characterize the negotiated session.

Record:

```text
Cipher suite
Key exchange information where visible
Supported versus selected values
```

Do not turn cipher identification into a security verdict without broader context.

## TLS Connection Timing

Timing remains visible even when content is encrypted.

Measure:

```text
ClientHello time
Server response time
Handshake completion
First application data
Connection duration
Termination
```

A simplified timeline:

```text
ClientHello
   ↓
ServerHello
   ↓
Handshake
   ↓
Encrypted application data
   ↓
Idle period
   ↓
More application data
   ↓
Connection close
```

This can reveal:

* Initial connection delay
* Handshake delay
* Long idle periods
* Short repeated sessions
* Long-lived sessions
* Connection churn

## TLS Handshake Failure

A failed handshake can be caused by many factors.

Possible explanations include:

```text
Certificate validation issue
Protocol mismatch
Cipher negotiation failure
Server rejection
Network interference
Firewall/proxy behavior
Client configuration
Application failure
```

Investigate the exact visible alert or sequence.

Do not infer the root cause from the presence of an unsuccessful handshake alone.

## Repeated TLS Connections

Repeated connections can be normal.

For example:

```text
10:00:00 → TLS connection
10:00:30 → TLS connection
10:01:00 → TLS connection
```

Possible explanations include:

* Polling
* API requests
* Application refresh
* Short-lived web sessions
* Monitoring
* Background services
* Automated communication

Investigate:

```text
Destination
Interval
Connection duration
Traffic volume
SNI
Application context
```

## Long-Lived TLS Sessions

A single connection lasting a long time can also be normal.

Examples:

```text
WebSocket
Streaming
Messaging
Remote administration
Long-lived API connection
Monitoring
```

Investigate the amount and direction of traffic rather than treating duration alone as suspicious.

## Traffic Volume

Encrypted payloads may still provide useful volume information.

Record:

```text
Packets sent
Packets received
Bytes sent
Bytes received
Duration
Average packet size
Burst behavior
```

Compare directions:

```text
Client → Server
Server → Client
```

A highly asymmetric session may have a different meaning from a balanced exchange.

But traffic volume alone cannot identify application content.

## Packet Size Patterns

Packet sizes can help characterize behavior.

Look for:

```text
Small regular packets
Large bursts
Repeated packet sizes
Long periods of silence
Periodic bursts
Sustained data transfer
```

These observations can support hypotheses about communication behavior.

They should not be treated as proof of specific content.

## TCP Analysis Beneath TLS

Encrypted application traffic still runs over transport protocols.

For TLS over TCP, examine:

```text
TCP handshake
Sequence numbers
Acknowledgments
Retransmissions
Duplicate ACKs
Window behavior
Resets
Connection termination
```

This can help distinguish:

```text
Application problem
Transport problem
Network problem
```

For example:

```text
TLS handshake begins
↓
TCP retransmissions
↓
Long delay
↓
Connection reset
```

The capture may suggest transport instability rather than a TLS configuration problem.

## TLS vs TCP Timing

Compare:

```text
TCP connection establishment
        ↓
TLS handshake
        ↓
First application data
```

This allows you to separate:

```text
TCP setup delay
```

from:

```text
TLS handshake delay
```

and:

```text
Application response delay
```

This is particularly useful during performance investigations.

## HTTPS Without Visible HTTP

A common mistake is expecting to see:

```text
GET
POST
HTTP/1.1
Host
URI
```

inside encrypted HTTPS traffic.

Without appropriate decryption material, these fields may not be visible.

Instead, investigate:

```text
TLS
SNI where visible
Certificate
ALPN
Destination IP
Timing
Traffic volume
TCP behavior
```

Do not claim that a specific URL was requested if the capture does not expose it.

## Encrypted DNS

DNS itself may also be encrypted.

Examples include:

```text
DNS over TLS
DNS over HTTPS
```

This changes the investigation workflow.

Instead of expecting traditional DNS packets, identify:

```text
Encrypted connection
Destination
Port
TLS metadata
Timing
Application context
```

For DNS over HTTPS, traffic may resemble ordinary HTTPS.

Therefore, destination and protocol metadata may be necessary to establish context.

## QUIC and HTTP/3

Modern web traffic may use QUIC over UDP.

Investigate:

```text
UDP communication
Destination
Port
QUIC packets
Connection timing
TLS-related handshake information
Connection duration
Traffic volume
```

Do not assume:

```text
UDP = DNS
```

or:

```text
TCP/443 = HTTPS
```

Protocol dissection should guide the investigation.

## SSH Traffic

SSH is encrypted at the application layer.

Wireshark can still reveal:

```text
Source
Destination
TCP port
Connection establishment
SSH identification exchange when visible
Session timing
Packet volume
Connection termination
```

But encrypted session content generally cannot reveal:

```text
Commands
Shell output
Credentials
Files
```

unless appropriate decryption or endpoint evidence exists.

Do not infer specific commands from packet timing alone.

## IPsec and VPN Traffic

VPN traffic can conceal internal application communication.

Depending on the capture point, you may see:

```text
Outer source
Outer destination
Tunnel protocol
Packet sizes
Timing
Encrypted payload
```

The inner communication may not be visible.

This creates an important distinction:

```text
Observed:
Host A communicated with VPN endpoint.

Not necessarily observed:
Host A communicated with internal Host B.
```

The second statement requires visibility inside the tunnel or additional evidence.

## Capture Location Matters

Encrypted traffic must always be interpreted relative to the capture point.

Example:

```text
Endpoint
  ↓
VPN
  ↓
Internet
```

A capture:

```text
Before VPN encryption
```

may show different information from a capture:

```text
Inside VPN
```

or:

```text
After VPN encapsulation
```

Always record:

```text
Capture point:
Interface:
Network segment:
Tunnel visibility:
```

## Proxy-Terminated TLS

A corporate proxy may terminate TLS.

The traffic path can become:

```text
Client
  ↓
TLS
  ↓
Proxy
  ↓
New TLS session
  ↓
Server
```

This may result in two separate encrypted sessions.

Investigate:

```text
Client → Proxy
Proxy → Server
```

Do not automatically assume that the client directly communicated with the final server.

## Load Balancers

A load balancer can create relationships such as:

```text
Client
  ↓
Load Balancer
  ↓
Backend Server
```

The destination IP seen by the client may represent the load balancer rather than the backend system.

Certificate, SNI, timing, and infrastructure context can help explain the relationship.

## Encrypted Traffic and DNS Correlation

Even when HTTPS content is encrypted, DNS can provide useful context.

A common sequence is:

```text
DNS query
   ↓
DNS response
   ↓
TLS connection to returned IP
   ↓
SNI
   ↓
Certificate
   ↓
Encrypted application traffic
```

Correlate:

```text
Hostname
Resolved IP
Destination IP
Timestamp
SNI
Certificate
```

This can provide stronger evidence than any single field.

## DNS-to-TLS Correlation

Suppose:

```text
10:00:00
DNS:
api.example.test → 203.0.113.10

10:00:01
TLS:
192.0.2.20 → 203.0.113.10:443

SNI:
api.example.test
```

The relationship is strongly supported by the packet sequence.

Record the timestamps and relevant packet numbers.

## Multiple Domains on One IP

A single IP may serve multiple domains.

Possible reasons include:

```text
CDN
Shared hosting
Reverse proxy
Load balancer
Cloud infrastructure
Multi-tenant service
```

Therefore:

```text
One IP ≠ One Application
```

SNI and certificate information can help distinguish the intended service where available.

## One Domain Across Multiple IPs

Likewise:

```text
api.example.test
   ↓
203.0.113.10
203.0.113.11
203.0.113.12
```

may be normal.

Possible reasons:

* Load balancing
* Geographic distribution
* CDN
* Failover
* Multiple service instances

Do not treat changing destination IPs as automatically suspicious.

## Encrypted Traffic and Suspicious Communication

When investigating potentially suspicious encrypted communication, focus on the combination of:

```text
Destination
Frequency
Timing
Connection duration
Packet volume
SNI
Certificate
DNS relationship
Protocol
Host role
Repeated connections
Follow-on behavior
```

Avoid relying on one indicator.

## Example Investigation

Observed:

```text
Host:
192.0.2.50

Destination:
203.0.113.20:443

SNI:
service.example.test

Certificate:
service.example.test

Connections:
Every 60 seconds

Duration:
Approximately 3 seconds

Traffic:
Small outbound request
Small inbound response
```

Possible interpretation:

```text
Periodic application communication
```

That is more defensible than claiming:

```text
The host is beaconing.
```

The stronger conclusion requires additional context.

## Comparing Competing Explanations

For recurring encrypted communication, consider:

```text
Hypothesis A:
Application polling

Hypothesis B:
Monitoring

Hypothesis C:
Background synchronization

Hypothesis D:
Automated service communication

Hypothesis E:
Suspicious periodic communication
```

For each, record:

```text
Supporting evidence
Contradicting evidence
Missing evidence
```

This prevents premature classification.

## Practical Exercise 1 — TLS Handshake

Find a TLS connection.

Record:

```text
Client:
Server:
Destination port:
TLS version:
Cipher information:
SNI:
Certificate:
ALPN:
Handshake timing:
```

Mark fields that are unavailable.

## Practical Exercise 2 — DNS → TLS Correlation

Find a hostname resolution followed by a TLS connection.

Record:

```text
DNS query:
DNS response:
Resolved IP:
TLS destination:
SNI:
Timestamp difference:
```

Determine whether the DNS and TLS events can reasonably be correlated.

## Practical Exercise 3 — Certificate Investigation

Choose a TLS connection with visible certificate information.

Record:

```text
Subject:
Issuer:
Validity:
SAN:
Destination:
SNI:
```

Compare the certificate names with the requested hostname.

Document any differences and possible benign explanations.

## Practical Exercise 4 — Repeated Encrypted Sessions

Find a host making repeated encrypted connections.

Measure:

```text
Connection count:
Destination:
Interval:
Duration:
Bytes sent:
Bytes received:
```

Determine whether the pattern is:

```text
Periodic
Bursting
Continuous
Irregular
```

Do not classify the intent yet.

## Practical Exercise 5 — TLS Performance

Find a slow encrypted connection.

Separate:

```text
TCP establishment time
TLS handshake time
First application-data delay
Subsequent response delay
```

Identify which stage contributes most to the observed delay.

## Practical Exercise 6 — Encrypted SSH

Find an SSH session.

Record:

```text
Source:
Destination:
Port:
TCP handshake:
SSH metadata:
Session duration:
Packet volume:
Termination:
```

List which information is visible and which remains encrypted.

## Practical Exercise 7 — QUIC

Find QUIC traffic if available.

Record:

```text
Source:
Destination:
UDP port:
Connection timing:
Visible handshake information:
Traffic duration:
Packet volume:
```

Explain why identifying UDP traffic alone is insufficient to determine its application.

## Practical Exercise 8 — VPN Visibility

Find tunnel or VPN-related traffic.

Determine:

```text
Outer source:
Outer destination:
Tunnel protocol:
Visible metadata:
Encrypted region:
Inner traffic visibility:
```

State clearly what the capture can and cannot establish.

## Practical Exercise 9 — Competing Hypotheses

Choose one repeated encrypted communication pattern.

Create:

```text
Hypothesis A:
Hypothesis B:
Hypothesis C:
```

For each:

```text
Supporting evidence:
Contradicting evidence:
Missing evidence:
```

Then state what additional evidence would discriminate between the hypotheses.

## Practical Exercise 10 — Independent Encrypted-Traffic Investigation

Choose an unfamiliar encrypted session.

Without starting with a classification:

```text
1. Identify the endpoints.
2. Identify the protocol.
3. Establish the connection timeline.
4. Inspect visible handshake metadata.
5. Inspect certificate information.
6. Inspect SNI where available.
7. Inspect ALPN where available.
8. Examine TCP or UDP behavior.
9. Measure traffic volume.
10. Correlate DNS.
11. Consider proxies and VPNs.
12. Document capture limitations.
13. Separate observation from interpretation.
14. Identify missing evidence.
```

The objective is to determine how much can actually be learned from encrypted traffic.

## Evidence Record

Use this structure:

```text
Investigation:
Capture:
Time window:

Source:

Destination:

Protocol:

Transport:

TLS/Encryption details:

SNI:

Certificate:

ALPN:

DNS relationship:

Connection duration:

Packet count:

Bytes sent:

Bytes received:

Timing observations:

Repeated behavior:

Infrastructure intermediaries:

Capture location:

Visible evidence:

Encrypted/unavailable evidence:

Interpretation:

Alternative explanations:

Unknowns:

Additional evidence required:
```

## Observation vs Interpretation

### Observation

```text
Host 192.0.2.50 established a TLS connection to
203.0.113.20:443 at 10:00:01.
```

### Interpretation

```text
The session is consistent with encrypted application communication.
```

### Unsupported conclusion

```text
The host downloaded a malicious payload.
```

The third statement requires evidence that is not present in the first two observations.

## Common Mistakes

### Mistake 1: Treating Encryption as No Evidence

Metadata can remain highly informative.

### Mistake 2: Assuming HTTPS Reveals HTTP

Without decryption, HTTP content may not be visible.

### Mistake 3: Treating SNI as the Complete Destination Identity

CDNs, proxies, and shared infrastructure can complicate interpretation.

### Mistake 4: Assuming the Certificate Proves the Application Is Legitimate

Certificates provide identity-related evidence, not a complete application trust decision.

### Mistake 5: Ignoring DNS

DNS can provide valuable context for encrypted sessions.

### Mistake 6: Ignoring Capture Location

VPNs, proxies, and NAT can fundamentally change what is visible.

### Mistake 7: Calling Periodic Traffic a Beacon Automatically

Periodic communication has many legitimate causes.

### Mistake 8: Inferring Commands From SSH Packet Patterns

Encrypted SSH content should not be reconstructed from unsupported assumptions.

### Mistake 9: Ignoring QUIC

Modern encrypted web traffic may use UDP rather than TCP.

### Mistake 10: Overstating What Encryption Analysis Proves

Always distinguish:

```text
Observed
```

from:

```text
Inferred
```

and:

```text
Unknown
```

## Professional Encrypted-Traffic Workflow

Use this workflow during authorized investigations:

```text
Question
  ↓
Define scope
  ↓
Identify encrypted protocol
  ↓
Identify endpoints
  ↓
Inspect handshake
  ↓
Inspect visible identity metadata
  ↓
Inspect certificate
  ↓
Inspect SNI/ALPN where available
  ↓
Correlate DNS
  ↓
Analyze timing
  ↓
Analyze traffic volume
  ↓
Analyze transport behavior
  ↓
Consider NAT/proxy/VPN/load balancer
  ↓
Document visibility limitations
  ↓
Separate observations from interpretations
  ↓
Identify missing evidence
```

The central question is:

```text
"What can this capture actually establish about the encrypted communication?"
```

not:

```text
"What application content do I assume was exchanged?"
```

## Investigation Checklist

Before closing an encrypted-traffic investigation:

* [ ] Scope defined
* [ ] Source identified
* [ ] Destination identified
* [ ] Protocol identified
* [ ] Transport protocol identified
* [ ] TLS/handshake inspected where applicable
* [ ] TLS version recorded where visible
* [ ] Cipher information recorded where visible
* [ ] SNI inspected where available
* [ ] Certificate inspected where available
* [ ] ALPN inspected where available
* [ ] DNS correlation performed
* [ ] Connection timing measured
* [ ] Traffic volume considered
* [ ] TCP/UDP behavior examined
* [ ] Repeated communication analyzed
* [ ] NAT considered
* [ ] Proxy considered
* [ ] VPN/tunnel visibility considered
* [ ] Load balancer/CDN effects considered
* [ ] Encrypted content limitations documented
* [ ] Observations separated from interpretations
* [ ] Alternative explanations considered
* [ ] Additional evidence identified

## Completion Criteria

You should be able to independently investigate encrypted traffic and answer:

```text
Who communicated?

With whom?

When?

Using which protocol?

What handshake information is visible?

What TLS metadata is visible?

Is SNI visible?

What certificate information is available?

Can DNS be correlated?

How long did the session last?

How much traffic moved?

Was communication periodic or continuous?

Could a proxy, VPN, NAT, CDN, or load balancer affect interpretation?

What is encrypted and unavailable?

What does the capture actually establish?

What remains unknown?

What additional evidence would be required?
```

The core mental model is:

```text
Encryption
   ↓
Reduced Content Visibility
   ↓
Remaining Metadata
   ↓
Handshake + Identity
   ↓
DNS + Endpoint Correlation
   ↓
Timing + Volume
   ↓
Infrastructure Context
   ↓
Evidence-Based Interpretation
```
