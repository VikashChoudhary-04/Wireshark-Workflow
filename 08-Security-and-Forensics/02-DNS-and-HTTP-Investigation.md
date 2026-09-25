# DNS and HTTP Investigation

## Objective

DNS and HTTP are two of the most useful protocols for network-based security investigation because they can reveal relationships between hosts, names, destinations, requests, responses, timing, and application behavior.

In an authorized investigation, these protocols can help answer questions such as:

```text id="4p7k2m"
What hostname did the system resolve?

Which DNS server handled the request?

What address was returned?

What HTTP host was contacted?

What URI was requested?

What response was returned?

Did the DNS result lead to the observed connection?

Did the communication repeat?

Did the behavior differ from the host's normal activity?
```

The goal is not to treat unusual DNS or HTTP traffic as automatically malicious.

The goal is to reconstruct the observable sequence:

```text id="8s3n6q"
Host activity
    ↓
DNS query
    ↓
DNS response
    ↓
Address selection
    ↓
Connection
    ↓
HTTP request
    ↓
HTTP response
    ↓
Additional communication
```

Then determine which parts are supported by packet evidence and which require additional investigation.

## Why DNS and HTTP Work Well Together

DNS answers:

```text id="c5r8v1"
"What address was associated with this name?"
```

HTTP can answer:

```text id="m2x6z9"
"What application request was sent to that destination?"
```

Together they can provide a useful relationship:

```text id="q7t4p0"
Hostname
   ↓
DNS answer
   ↓
IP address
   ↓
TCP connection
   ↓
HTTP Host
   ↓
URI
   ↓
Response
```

This relationship is valuable during troubleshooting and security investigation.

However, the relationship is not always one-to-one.

Reasons include:

* DNS caching
* Shared hosting
* CDNs
* Proxies
* Load balancers
* Multiple addresses
* Connection reuse
* Direct IP connections
* Encrypted DNS
* HTTPS encryption
* DNS answers changing over time

Always verify the observed packets.

## Define the Investigation

Before filtering traffic, establish:

| Item              | Question                            |
| ----------------- | ----------------------------------- |
| Host              | Which system is being investigated? |
| Time              | What time window matters?           |
| DNS               | Which resolver is involved?         |
| Hostname          | Which name is relevant?             |
| Destination       | Which IP or service is relevant?    |
| Protocol          | DNS, HTTP, HTTPS?                   |
| Application       | What generated the traffic?         |
| Expected behavior | What should normally happen?        |

A security investigation becomes much easier when the scope is explicit.

## Step 1: Identify the Investigated Host

Start with the host's network address.

For IPv4:

```text id="d1f5h8"
ip.addr == 192.0.2.10
```

For IPv6:

```text id="j4n7q2"
ipv6.addr == 2001:db8::10
```

These are documentation examples.

Once the host is isolated, identify:

```text id="v6c2x9"
DNS queries
HTTP requests
TCP connections
TLS connections
Other related protocols
```

Do not immediately filter only for DNS or HTTP.

First understand the host's overall communication context when the capture is unfamiliar.

## Step 2: Build a DNS Timeline

Find the relevant DNS query.

Record:

```text id="p3s8w1"
Time:
Source:
DNS server:
Query name:
Record type:
Response:
Answer:
```

Then determine what happened afterward.

Example:

```text id="a6k2m9"
10:00:01 DNS query: api.example.test
10:00:01 DNS response: 203.0.113.50
10:00:01 TCP SYN → 203.0.113.50:80
10:00:01 HTTP request
```

This is stronger evidence than simply saying:

```text id="r4t7y0"
"The host visited api.example.test."
```

The packet capture should support the relationship.

## DNS-to-IP Correlation

When a DNS response contains an address, locate subsequent traffic involving that address.

For example:

```text id="u8x1c4"
DNS:
example.test → 203.0.113.50

Later:
Client → 203.0.113.50:80
```

Record the timing.

The closer the events are and the more consistent the protocol sequence, the stronger the correlation.

But remember that a cached answer may have been obtained before the capture began.

## DNS-to-IP Correlation Limitations

Do not assume:

```text id="k5m9p2"
DNS response
↓
Immediate connection
```

must always occur.

Possible reasons for no visible connection include:

* Cached or pre-existing connection
* Application abandoned the request
* Different returned address selected
* Connection made through a proxy
* Traffic outside capture visibility
* Application using another interface
* IPv6/IPv4 selection
* Encrypted or tunneled traffic

Document what is actually observed.

## Multiple A Records

A DNS response may return multiple addresses:

```text id="n2q6v8"
example.test
    ↓
203.0.113.10
203.0.113.20
203.0.113.30
```

The client may select one of them.

Determine:

```text id="b4d7j1"
Which address was used?
Were multiple addresses attempted?
Did one fail?
Did another succeed?
```

This can be particularly useful when a service uses load balancing.

## A and AAAA Correlation

A hostname can return both:

```text id="x8z3c5"
A
AAAA
```

The client may attempt IPv6 first and then fall back to IPv4.

A security or troubleshooting investigation should therefore record:

```text id="m6p1r9"
A result:
AAAA result:
First connection:
Successful connection:
Failed connection:
```

This can explain apparently intermittent behavior.

## CNAME Correlation

A hostname may resolve through a CNAME:

```text id="q2w7e4"
requested.example.test
        ↓
CNAME
        ↓
service.example.test
        ↓
A / AAAA
```

Record the full observable chain.

This may reveal that a seemingly unfamiliar destination belongs to a known service infrastructure.

## DNS Response Codes in Security Investigation

Pay attention to:

```text id="g9k4s2"
NOERROR
NXDOMAIN
SERVFAIL
REFUSED
```

Repeated errors can reveal application behavior.

For example:

```text id="v1c6x8"
Many NXDOMAIN queries
```

could indicate:

* Misconfigured application
* Search-domain behavior
* Service discovery
* Software lookup behavior
* Application-generated failures

It may also be relevant to security analysis, but context is required.

## Repeated DNS Queries

Look for:

```text id="p5r0t3"
Same hostname repeatedly queried
```

Questions:

```text id="u7y2a6"
How often?
For how long?
Same resolver?
Same response?
Same address?
Did an application connection follow?
```

Repeated DNS activity can result from:

* Short cache lifetime
* Application polling
* Resolver configuration
* Failed connection attempts
* Service discovery
* Legitimate periodic behavior

Do not classify it based on repetition alone.

## High DNS Diversity

A host may query many distinct names.

Record:

```text id="e3i8o2"
Number of unique names
Number of successful responses
Number of failed responses
Time window
```

Compare against the host's expected role.

A browser workstation naturally produces more diverse DNS activity than a narrowly scoped infrastructure server.

## Long or Unusual DNS Names

Long or changing DNS labels can be worth investigating.

Possible legitimate explanations include:

* CDN identifiers
* Tracking
* Application-generated identifiers
* Service discovery
* Cloud infrastructure
* Telemetry

Possible security hypotheses can also exist.

Do not conclude DNS tunneling or data encoding solely from a long hostname.

Look for a broader pattern:

```text id="y6u1w5"
High query frequency
+
Long changing labels
+
Repeated destination
+
Unusual volume
+
Consistent timing
```

Even then, additional evidence is needed.

## DNS Query Timing

Measure:

```text id="a4d9f2"
Query → response
```

Look for:

* Very slow responses
* Repeated timeouts
* Resolver switching
* Bursts
* Periodic queries

A timing anomaly may explain an application delay or may represent unusual behavior.

The same evidence can be relevant to both troubleshooting and security investigation.

## DNS Server Changes

Determine whether the host communicates with multiple DNS servers.

For example:

```text id="j8m3q7"
Host → Resolver A
Host → Resolver B
```

Compare:

```text id="c5v0x2"
Response codes
Answers
Timing
Frequency
```

An unexpected resolver can deserve investigation.

But legitimate causes include:

* DHCP configuration
* VPN
* Enterprise security software
* Local forwarding
* Network changes

Use configuration and endpoint context to interpret the observation.

## Step 3: Follow the DNS Result Into the Connection

After identifying the DNS answer, locate subsequent traffic.

Example:

```text id="n7p2s6"
DNS:
api.example.test → 203.0.113.50

TCP:
Client → 203.0.113.50:80

HTTP:
GET /api/status
Host: api.example.test
```

This provides a strong observable chain.

Record:

```text id="w4y8u1"
DNS name
Returned address
Destination address
Destination port
HTTP Host
URI
```

Then compare them.

## HTTP Investigation

HTTP provides direct application-layer information when traffic is not encrypted.

Important fields include:

```text id="z2c6m0"
Method
Host
URI
Status code
Headers
User-Agent
Content type
Request size
Response size
Timing
```

A security investigation should first establish what the application actually requested.

## HTTP Methods

Common methods include:

```text id="f8h1k4"
GET
POST
PUT
PATCH
DELETE
HEAD
OPTIONS
```

A method is context, not a verdict.

For example:

```text id="p3r7t9"
POST
```

is common in legitimate APIs and web applications.

Investigate:

```text id="v5x0z2"
Destination
URI
Content type
Application context
Response
```

rather than treating a method itself as suspicious.

## HTTP Host Header

The Host header can help correlate an HTTP request with a hostname.

Example:

```text id="b6d2j8"
Host: api.example.test
```

Compare it with:

```text id="h0m4q7"
Observed DNS name
Destination IP
```

These values may not match one-to-one because of:

* Shared hosting
* Reverse proxies
* CDNs
* Load balancers
* Direct IP connections
* Virtual hosting

The comparison is still useful.

## HTTP URI

The URI can reveal the resource requested.

Examples:

```text id="n2p8r5"
/login
/api/users
/download
/upload
/search
```

Investigate:

```text id="c7x3z9"
Which host?
Which URI?
Which method?
Which response?
How often?
```

Avoid making a security conclusion from a URI name alone.

## HTTP Response Codes

Common status classes include:

```text id="u1w6y8"
2xx — successful
3xx — redirection
4xx — client-side/request-related errors
5xx — server-side errors
```

A response code helps describe application behavior.

For example:

```text id="g5j9m2"
GET /login
→ 302 redirect
```

may be perfectly normal.

Likewise:

```text id="q8s4v1"
GET /api/data
→ 500
```

may indicate an application problem.

The code does not automatically indicate malicious activity.

## HTTP Redirect Chains

A request may result in:

```text id="a2d7f5"
GET /start
↓
302
↓
GET /login
↓
200
```

Follow the chain.

This can reveal:

* Authentication flows
* Application routing
* CDN behavior
* Service migrations
* Unexpected destinations

Document the full observable sequence.

## HTTP User-Agent

The User-Agent can provide context about the client.

For example:

```text id="j6n0p4"
Browser
Operating system
Application
Library
```

Treat it as self-reported application metadata.

It can be useful for correlation, but it should not be treated as proof of the actual process or device identity.

## HTTP Headers

Headers can reveal application behavior such as:

```text id="r3v8x1"
Content-Type
Content-Length
Authorization-related metadata
Cookie
Referer
Cache-Control
Accept
```

In authorized captures, inspect sensitive headers carefully.

Do not expose credentials, tokens, cookies, or other secrets unnecessarily in reports.

## Sensitive Data Handling

HTTP captures may contain:

* Session cookies
* Authorization tokens
* User information
* Passwords
* API keys
* Application data

Treat packet captures as sensitive evidence.

When documenting an investigation:

```text id="m7q2s5"
Redact secrets
Avoid copying credentials
Minimize exposed personal information
Store captures securely
Share only with authorized personnel
```

The goal is to analyze the traffic without unnecessarily reproducing sensitive content.

## HTTP Request/Response Timing

Measure:

```text id="p9t4w7"
Request
   ↓
Response
```

A long delay may indicate:

* Server processing
* Backend dependency
* Network delay
* Packet loss
* Proxy behavior
* Application queueing

As always, the packet capture establishes timing, not necessarily the internal cause.

## HTTP and TCP Correlation

HTTP traffic is carried by a TCP connection in traditional HTTP/1.x deployments.

A useful sequence is:

```text id="v2x8z6"
TCP handshake
↓
HTTP request
↓
HTTP response
```

When investigating a slow or suspicious HTTP transaction, inspect the TCP behavior around the HTTP messages.

Look for:

* Retransmissions
* Duplicate ACKs
* Resets
* Window limitations
* Connection reuse
* Connection termination

## HTTP Connection Reuse

One TCP connection may carry multiple HTTP requests.

For example:

```text id="b4d9h2"
TCP handshake
↓
HTTP request A
HTTP response A
↓
HTTP request B
HTTP response B
↓
HTTP request C
HTTP response C
```

Do not treat every HTTP request as a new network connection.

Follow the relevant TCP stream.

## HTTP Without DNS

A client can connect directly to an IP address.

Example:

```text id="f6j1n8"
TCP:
192.0.2.10 → 203.0.113.50:80

HTTP:
Host: example.test
```

No DNS query may appear in the capture.

Possible explanations include:

* Cached resolution
* Static configuration
* Direct IP connection
* DNS occurred before capture
* Proxy behavior

Do not assume that a missing DNS query means the hostname was never resolved.

## HTTP Host and DNS Mismatch

Consider:

```text id="k3m7p1"
DNS:
example.test → 203.0.113.50

HTTP:
Host: another.example.test
Destination:
203.0.113.50
```

This can be legitimate in shared infrastructure.

Investigate:

```text id="w8y2c5"
Is the IP hosting multiple names?
Is a reverse proxy involved?
Is a CDN involved?
Was DNS cached?
```

Do not immediately classify the mismatch as suspicious.

## HTTPS Investigation

Traditional HTTPS encrypts HTTP contents.

You may still observe:

```text id="n4q8s2"
DNS
TCP
TLS
Destination IP
Port
TLS handshake metadata
Timing
Connection frequency
Packet sizes
Connection duration
```

The HTTP method, URI, and headers generally will not be directly visible without decryption.

Therefore distinguish:

```text id="r6t0v3"
Network metadata
```

from:

```text id="x1z5b7"
Encrypted application content
```

## TLS Server Name Information

When visible, TLS metadata such as SNI can help associate an encrypted connection with a hostname.

For example:

```text id="c9f3h6"
TLS:
Server Name: api.example.test
```

This can complement DNS analysis.

But SNI and DNS are different protocol observations.

Do not treat one as proof that the other occurred during the same capture.

## DNS → TLS Correlation

A useful sequence is:

```text id="j2m6q0"
DNS:
api.example.test → 203.0.113.50

TLS:
Client → 203.0.113.50
Server Name: api.example.test
```

This creates a stronger relationship between:

```text id="p4r8t1"
Name
Address
Encrypted connection
```

But proxies, CDNs, shared infrastructure, and connection reuse can complicate the relationship.

## HTTP vs HTTPS Visibility

A practical comparison:

| Information    | HTTP    | HTTPS           |
| -------------- | ------- | --------------- |
| Source IP      | Visible | Visible         |
| Destination IP | Visible | Visible         |
| Port           | Visible | Visible         |
| TCP behavior   | Visible | Visible         |
| TLS handshake  | No      | Usually visible |
| Host header    | Visible | Encrypted       |
| URI            | Visible | Encrypted       |
| HTTP method    | Visible | Encrypted       |
| Response body  | Visible | Encrypted       |
| Timing         | Visible | Visible         |

This is why encrypted traffic still supports useful security analysis even when application content is unavailable.

## HTTP Security Investigation Example

Suppose the capture shows:

```text id="t5v9x2"
DNS query:
cdn.example.test

DNS response:
203.0.113.50

TCP:
Client → 203.0.113.50:80

HTTP:
GET /download/update.exe
Host: cdn.example.test

HTTP:
200 OK
```

The packet evidence establishes:

```text id="a7c1e4"
The client resolved the hostname.
The resolver returned the observed address.
The client connected to that address.
The client requested the observed URI.
The server returned HTTP 200.
```

It does not establish:

```text id="m3q8w5"
Whether the downloaded file is malicious.
```

That requires additional analysis.

## Suspicious HTTP Investigation Example

Suppose a host repeatedly requests:

```text id="z2b6d8"
/random-looking-path
```

from the same external destination.

A professional workflow is:

```text id="h6k0p4"
1. Count requests.
2. Measure intervals.
3. Inspect Host.
4. Inspect URI patterns.
5. Inspect methods.
6. Inspect responses.
7. Check DNS history.
8. Check TCP behavior.
9. Compare against host baseline.
10. Identify endpoint/application context.
```

Do not conclude maliciousness from URI appearance alone.

## DNS and HTTP Timeline

For investigations, build a timeline such as:

```text id="n8r2v6"
10:00:01.000
DNS query: api.example.test

10:00:01.025
DNS response: 203.0.113.50

10:00:01.030
TCP SYN

10:00:01.045
TCP SYN/ACK

10:00:01.046
TCP ACK

10:00:01.050
HTTP GET /api/status

10:00:01.120
HTTP 200 OK
```

This timeline lets you reason about:

* DNS delay
* TCP delay
* Application delay
* Sequence
* Correlation

## Identify the Earliest Meaningful Divergence

Consider:

```text id="p4t8x2"
DNS succeeds
↓
TCP succeeds
↓
HTTP request sent
↓
Long delay
↓
HTTP 500
```

The failure is not primarily a DNS or TCP connectivity problem.

The observable divergence occurs during application processing.

Another example:

```text id="j6m0q4"
DNS query
↓
No response
↓
Retry
↓
No TCP connection
```

The earliest observed failure is DNS resolution.

Always start troubleshooting from the earliest meaningful divergence.

## DNS and HTTP Statistics

Statistics can help identify patterns across a large capture.

Useful areas include:

```text id="r2v6z8"
Protocol hierarchy
Endpoints
Conversations
I/O graphs
DNS traffic
HTTP traffic
Expert information
```

Questions include:

```text id="b8d0f2"
Which hosts generate the most DNS traffic?

Which destinations receive the most HTTP traffic?

Which names are queried repeatedly?

Are there bursts of HTTP activity?

Which conversations are unusually long?
```

Use statistics to find interesting areas, then return to individual packets for evidence.

## Practical Exercise 1 — DNS-to-HTTP Correlation

Find:

```text id="x4c8m2"
DNS query
↓
DNS response
↓
TCP connection
↓
HTTP request
```

Record:

```text id="q6s0v4"
Hostname:
Returned IP:
Destination IP:
HTTP Host:
URI:
Response:
```

Determine how strongly the packets support the relationship.

## Practical Exercise 2 — DNS Failure

Find a hostname that fails to resolve.

Determine:

```text id="w2y8a6"
Query:
Response:
Response code:
Retries:
Was HTTP attempted?
```

Explain where the expected sequence stopped.

## Practical Exercise 3 — Multiple Addresses

Find a hostname with multiple A or AAAA records.

Determine:

```text id="e4g0k2"
Returned addresses:
Address actually used:
Other addresses attempted:
Successful connection:
```

## Practical Exercise 4 — CNAME Chain

Find a DNS response containing a CNAME.

Trace:

```text id="m6o2q8"
Requested hostname
↓
CNAME
↓
Final address
↓
Connection
```

## Practical Exercise 5 — HTTP Request Analysis

Find an HTTP request and document:

```text id="u0w4y6"
Source:
Destination:
Method:
Host:
URI:
User-Agent:
Status:
Response size:
```

Determine what additional context would be needed to assess whether the request is expected.

## Practical Exercise 6 — HTTP Redirect Chain

Find an HTTP transaction containing a redirect.

Document:

```text id="a8c2e4"
Initial request:
Initial response:
Redirect destination:
Follow-up request:
Final response:
```

## Practical Exercise 7 — HTTP Timing

Find a request with a noticeable response delay.

Measure:

```text id="g6i8k0"
Request time:
Response time:
Elapsed time:
TCP retransmissions:
Window behavior:
```

Determine whether packet-level evidence suggests a transport problem or whether the delay occurs while waiting for the application response.

## Practical Exercise 8 — DNS and HTTPS

Find a hostname that is resolved and then contacted using TLS.

Record:

```text id="n2p6r8"
DNS name:
DNS result:
Destination IP:
TLS server name when visible:
Connection timing:
```

Explain what is visible and what remains encrypted.

## Practical Exercise 9 — Repeated HTTP Activity

Find repeated requests from one host to the same destination.

Determine:

```text id="t4v8x0"
Request count:
Time interval:
Methods:
URIs:
Status codes:
Response sizes:
```

Compare the pattern against the host's expected role.

## Practical Exercise 10 — Security Evidence Timeline

Build a complete timeline containing:

```text id="z2b6d4"
DNS
↓
Connection
↓
HTTP/TLS
↓
Application request
↓
Response
↓
Termination
```

For every stage, mark:

```text id="h8j0l2"
Observed
Not observed
Encrypted
Uncertain
```

## Professional DNS/HTTP Evidence Record

Use this structure:

```text id="p4r6t8"
Incident:
Date/Time:
Host:
Capture point:

Hostname:

DNS server:

DNS query:

Record type:

DNS response:

Returned addresses:

CNAME chain:

TCP connection:

TLS:

HTTP Host:

HTTP method:

HTTP URI:

HTTP status:

Request timing:

Response timing:

Repeated behavior:

Baseline:

What the packet capture proves:

What remains unknown:

Additional evidence required:
```

## Sensitive Evidence Handling

When working with real authorized captures:

```text id="c0e2g4"
Treat PCAPs as sensitive.
```

They may contain:

* Internal addresses
* Usernames
* Cookies
* Tokens
* Credentials
* URLs
* Application data
* Personal information

When creating GitHub writeups or reports:

```text id="m6o8q0"
Redact credentials
Redact session tokens
Use documentation IP addresses
Avoid publishing sensitive URIs
Remove personal information
Do not commit confidential PCAPs
```

A professional security workflow protects the evidence as carefully as it analyzes it.

## Common Mistakes

### Mistake 1: Assuming DNS Must Appear Before Every Connection

Caching, static configuration, proxies, and earlier lookups can prevent DNS packets from appearing in the capture.

### Mistake 2: Assuming DNS Answer Equals Application Destination

The application may select among multiple addresses or communicate through an intermediary.

### Mistake 3: Treating an Unfamiliar HTTP URI as Malicious

URI appearance alone is insufficient.

### Mistake 4: Treating Port 80 or 443 as Proof of Application Identity

Ports are contextual clues.

### Mistake 5: Ignoring Connection Reuse

Multiple HTTP requests can share one TCP connection.

### Mistake 6: Ignoring HTTPS Encryption

The absence of visible HTTP requests does not mean no HTTP communication occurred.

### Mistake 7: Exposing Sensitive HTTP Data

Packet captures may contain credentials and tokens.

Redact sensitive information in documentation.

### Mistake 8: Assuming a DNS Name Maps Permanently to One IP

CDNs, load balancing, and DNS policies can return different addresses.

### Mistake 9: Treating Repeated DNS Queries as Automatically Suspicious

Application behavior and caching configuration can produce repeated queries.

### Mistake 10: Overstating Correlation

A DNS response and later connection to the same address are useful evidence, but context still matters.

## Professional DNS and HTTP Investigation Workflow

Use this workflow consistently:

```text id="q8s0u2"
1. Define the investigated host and time window.
2. Identify relevant DNS traffic.
3. Identify query names and record types.
4. Correlate responses with queries.
5. Record response codes and answers.
6. Track A, AAAA, and CNAME behavior.
7. Follow returned addresses into subsequent traffic.
8. Identify TCP conversations.
9. Identify TLS when applicable.
10. Identify HTTP requests when visible.
11. Record Host, method, URI, and response.
12. Measure timing.
13. Examine repeated communication.
14. Compare against normal behavior.
15. Correlate DNS, transport, and application stages.
16. Distinguish visible content from encrypted content.
17. Protect sensitive evidence.
18. Separate observations from interpretations.
19. Identify what additional evidence is required.
20. Document the investigation.
```

## Investigation Checklist

Before closing a DNS/HTTP investigation:

* [ ] Host identified
* [ ] Time window defined
* [ ] Capture point documented
* [ ] DNS query identified
* [ ] DNS response identified
* [ ] Response code checked
* [ ] A records checked
* [ ] AAAA records checked
* [ ] CNAME chain checked
* [ ] DNS timing checked
* [ ] DNS retries checked
* [ ] DNS server identified
* [ ] Returned address correlated with subsequent traffic
* [ ] TCP conversation identified
* [ ] TLS considered
* [ ] HTTP Host identified
* [ ] HTTP method identified
* [ ] URI identified when visible
* [ ] Response status identified
* [ ] Request/response timing checked
* [ ] Repeated behavior examined
* [ ] Baseline considered
* [ ] Sensitive information protected
* [ ] Observation separated from interpretation
* [ ] Capture limitations documented
* [ ] Additional evidence identified

## Completion Criteria

You should be able to independently investigate a DNS-to-application communication chain and answer:

```text id="w4y6a8"
What hostname was requested?

Who requested it?

Which resolver responded?

What record type was requested?

What answer was returned?

Was there a CNAME chain?

Which address was used?

Was IPv4 or IPv6 selected?

Did TCP establish?

Did TLS occur?

Was HTTP visible?

What Host and URI were requested?

What response was returned?

How long did each stage take?

Did the communication repeat?

Is the behavior consistent with the host's baseline?

What does the capture prove?

What remains unknown?

What additional evidence is required?
```

The core mental model is:

```text id="n0p2r4"
DNS
 ↓
Name
 ↓
Address
 ↓
Connection
 ↓
TLS / HTTP
 ↓
Request
 ↓
Response
 ↓
Behavior
 ↓
Context
```

The strongest investigations connect these layers without assuming that any single packet, hostname, URI, destination, or response code provides the complete explanation.
