# DNS and Name Resolution

## Objective

DNS problems are often reported as application or connectivity failures:

```text
"The website is not opening."
"The server cannot be reached by name."
"The application cannot connect."
"IP address works, hostname does not."
"Sometimes the hostname works and sometimes it does not."
```

Wireshark allows you to reconstruct the name-resolution process and determine where it stops.

The goal of this workflow is to answer:

* Did the client generate a DNS query?
* Which DNS server received it?
* What name was queried?
* What record type was requested?
* Did the DNS server respond?
* What response code was returned?
* Did the response contain useful records?
* Was the response delayed or repeated?
* Did the client retry?
* Did the application use the returned address?
* Did the problem occur with IPv4, IPv6, or both?
* Is the observed behavior consistent with the reported failure?

The central workflow is:

```text
User symptom
    ↓
Identify hostname and time
    ↓
Identify client
    ↓
Identify DNS server
    ↓
Observe query
    ↓
Observe response
    ↓
Check response code
    ↓
Check answer records
    ↓
Check timing and retries
    ↓
Check subsequent connection
    ↓
Compare successful vs failed resolution
    ↓
Determine earliest supported failure
```

DNS analysis should remain evidence-driven.

A DNS packet can show what was requested and what response was returned. It does not automatically explain why a resolver produced that result.

## DNS as Part of the Connectivity Workflow

A hostname-based connection normally contains several stages:

```text
Application
    ↓
Hostname
    ↓
DNS resolution
    ↓
IP address
    ↓
Network communication
    ↓
TCP/UDP
    ↓
TLS
    ↓
Application protocol
```

For example:

```text
https://example.test
        ↓
DNS query for example.test
        ↓
DNS response
        ↓
203.0.113.50
        ↓
TCP connection to 203.0.113.50:443
        ↓
TLS
        ↓
HTTP
```

If DNS fails, the later stages may never occur.

If DNS succeeds but TCP fails, DNS is not necessarily the root of the connectivity problem.

This distinction is fundamental.

## First Define the Name-Resolution Problem

Before filtering packets, record:

| Item                 | Question                                |
| -------------------- | --------------------------------------- |
| Client               | Which host is experiencing the problem? |
| Hostname             | What name is being resolved?            |
| DNS server           | Which resolver is being used?           |
| Time                 | When did the failure occur?             |
| Record type          | A, AAAA, CNAME, MX, TXT, etc.?          |
| Expected result      | What should DNS return?                 |
| Actual result        | What did it return?                     |
| Follow-up connection | Did the client connect afterward?       |
| Capture point        | Where was the packet capture taken?     |

This prevents unrelated DNS traffic from becoming part of the investigation.

## Identify the DNS Traffic

DNS commonly uses:

```text
UDP/53
```

TCP can also be used for DNS.

Modern environments may additionally use encrypted DNS mechanisms such as:

```text
DNS over TLS
DNS over HTTPS
```

These can change what is visible in a packet capture.

For traditional DNS, a useful starting filter is:

```text
dns
```

You can then narrow by address:

```text
dns && ip.addr == 192.0.2.10
```

Or focus on DNS traffic involving a known server:

```text
dns && ip.addr == 192.0.2.53
```

These are examples using documentation addresses.

The important skill is not memorizing the filters. It is identifying the communication you need to inspect.

## Identify the Client

Determine which host initiated the DNS query.

For a typical request:

```text
Client → DNS Server
```

the source address identifies the querying host at the capture point.

Record:

```text
Source IP:
Destination IP:
Source port:
Destination port:
Query name:
Query type:
```

For IPv6, inspect:

```text
ipv6.src
ipv6.dst
```

Do not assume IPv4 merely because the application eventually connects to an IPv4 address.

## Identify the DNS Server

Determine which system answered the query.

It may be:

* Local router
* Corporate resolver
* ISP resolver
* Public resolver
* Recursive resolver
* Internal DNS server
* DNS forwarder

The IP address visible in the packet tells you which endpoint participated in the observed transaction.

The packet alone may not establish the entire DNS architecture behind that endpoint.

For example:

```text
Client → Internal DNS Resolver
```

The internal resolver may itself contact other DNS infrastructure that is outside the capture point.

## DNS Query and Response

A basic DNS exchange looks like:

```text
Client → DNS Server
Query: example.test A

DNS Server → Client
Response: example.test A 203.0.113.50
```

The first task is to associate the response with the correct query.

Wireshark exposes DNS transaction information that helps correlate requests and responses.

The workflow is:

```text
Query
  ↓
Transaction
  ↓
Response
  ↓
Answer
```

Do not analyze DNS responses without first determining which query they answer.

## Query Name

The queried hostname is one of the most important fields.

Ask:

```text
What exact name was requested?
```

For example:

```text
www.example.test
api.example.test
mail.example.test
```

Check whether:

* The expected hostname was requested
* The name was truncated or malformed
* Multiple related names were queried
* The client appended search-domain components
* Different hostnames were attempted

A user may report:

```text
"example.test is not working."
```

while the client is actually querying:

```text
example.test.internal
```

That distinction can matter.

## Record Types

Different DNS record types answer different questions.

Common examples:

| Record | Purpose              |
| ------ | -------------------- |
| A      | IPv4 address         |
| AAAA   | IPv6 address         |
| CNAME  | Canonical-name alias |
| MX     | Mail exchange        |
| NS     | Name server          |
| TXT    | Text data            |
| PTR    | Reverse lookup       |

For application connectivity, A and AAAA records are especially important.

A hostname can return:

```text
A
```

without returning:

```text
AAAA
```

or vice versa.

The application may behave differently depending on which address family is available.

## A and AAAA Analysis

A typical dual-stack lookup may involve:

```text
Client → DNS: A query
Client ← DNS: IPv4 address

Client → DNS: AAAA query
Client ← DNS: IPv6 address
```

The application may then attempt IPv6 and IPv4 connections.

When investigating intermittent or asymmetric connectivity, determine:

```text
Was an A record returned?
Was an AAAA record returned?
Which address did the client use?
Did one address family succeed while the other failed?
```

This can reveal problems that are invisible if the investigation considers only IPv4.

## CNAME Chains

A response may contain a CNAME:

```text
service.example.test
        ↓
CNAME
        ↓
backend.example.test
        ↓
A
        ↓
203.0.113.50
```

Do not treat the CNAME as the final connection address automatically.

Follow the chain and identify the actual address records returned.

The important question is:

```text
What address did the client ultimately receive for the requested service?
```

## DNS Response Codes

DNS responses contain a status indicating how the query was handled.

Common outcomes include:

```text
NOERROR
NXDOMAIN
SERVFAIL
REFUSED
```

### NOERROR

`NOERROR` indicates that the DNS response completed without a DNS-level error status.

It does not necessarily mean the requested name had a useful address record.

Inspect the answer section.

### NXDOMAIN

`NXDOMAIN` indicates that the queried domain name does not exist according to the responding DNS server's authoritative information or resolution path.

Do not translate it automatically into:

```text
"The entire DNS server is broken."
```

It is a specific DNS response.

### SERVFAIL

`SERVFAIL` indicates that the server could not successfully complete the resolution.

Possible underlying reasons can include:

* Upstream resolution problems
* DNSSEC-related issues
* Resolver failure
* Temporary infrastructure problems
* Configuration problems

The packet establishes the response status, not necessarily the underlying cause.

### REFUSED

`REFUSED` indicates that the server refused to perform the requested operation.

Possible explanations depend on the DNS server configuration and request context.

Again, distinguish:

```text
Observed:
DNS response status = REFUSED

Hypothesis:
Resolver policy or access-control behavior may be involved.
```

## No Response vs Error Response

These are different situations.

### No response

```text
Query
Query
Query
...
```

No response is observed.

Possible explanations include:

* DNS server unavailable
* Packet loss
* Network path problem
* Filtering
* Incorrect destination
* Response outside capture visibility
* Capture timing limitations

### Error response

```text
Query
Response: SERVFAIL
```

This gives stronger evidence about what the DNS server returned.

The investigation should move toward understanding the response and its context rather than treating it as a missing-packet problem.

## DNS Retries

Repeated queries can indicate that the client or resolver did not receive a satisfactory response.

Example:

```text
Query
      ↓
No response
      ↓
Query
      ↓
No response
      ↓
Query
```

Record:

* Number of attempts
* Time between attempts
* Whether the destination stayed the same
* Whether the query contents changed
* Whether any response eventually appeared

Repeated DNS traffic is evidence of retry behavior.

It is not, by itself, proof of the root cause.

## DNS Timing

Timing can be just as important as response content.

Compare:

```text
Query → Response
```

with:

```text
Query → long delay → Response
```

and:

```text
Query → timeout → retry
```

A successful but very slow DNS response can still create an application timeout or noticeable user delay.

Record the approximate query-to-response delay.

For repeated failures, determine whether the delay is:

```text
Consistent
Increasing
Variable
Only present for one server
Only present for one record type
```

## DNS and Application Correlation

A DNS investigation should not stop at the DNS response.

Determine what happened next.

Example:

```text
DNS query
↓
DNS response: 203.0.113.50
↓
TCP SYN to 203.0.113.50:443
```

This establishes that the client used the returned address for a subsequent connection attempt.

Another pattern:

```text
DNS query
↓
DNS response
↓
No subsequent connection
```

Possible explanations include:

* Application did not attempt the connection
* Application abandoned the request
* Another address was selected
* Connection occurred through another interface
* Relevant traffic is outside the capture
* Application behavior prevented the next stage

Do not assume that every DNS response must immediately produce a TCP connection.

## DNS Failure Example: NXDOMAIN

Suppose the capture shows:

```text
Client → DNS:
Query: app.example.test A

DNS → Client:
Response: NXDOMAIN
```

No TCP connection follows.

Evidence:

```text
The resolver returned NXDOMAIN.
No subsequent connection to an address for that hostname is observed.
```

Interpretation:

```text
The requested hostname did not resolve through the observed DNS server.
```

The next investigation question may be:

```text
Was the hostname correct?
Was the client using the correct DNS server?
Is the record expected to exist?
Is the observed DNS server authoritative or recursive?
```

## DNS Failure Example: SERVFAIL

Suppose:

```text
Query
↓
SERVFAIL
↓
Retry
↓
SERVFAIL
```

The evidence shows repeated unsuccessful DNS resolution.

Do not immediately conclude:

```text
"DNS is down."
```

Instead investigate:

```text
Which resolver responded?
Did other names resolve?
Did another DNS server behave differently?
Is the failure specific to one domain?
Did the response timing change?
```

## DNS Failure Example: No Response

Suppose:

```text
Client → DNS server: Query
Client → DNS server: Query
Client → DNS server: Query
```

No response is visible.

Start with:

```text
Is the DNS server address correct?
Is the query leaving the client?
Is the capture point before or after the expected response path?
Are other DNS queries succeeding?
Does another DNS server respond?
```

The absence of a response is an observation, not a complete diagnosis.

## Compare Working and Failing DNS

One of the strongest workflows is:

```text
Working:
Query → Response → Application connection

Failing:
Query → no response → Retry → timeout
```

Compare:

* Same hostname
* Same client
* Same DNS server
* Same record type
* Similar time
* Same network path

Then identify the first difference.

## Multiple DNS Servers

A client may use multiple DNS servers.

You may observe:

```text
Client → DNS Server A
Client → DNS Server B
```

If one responds while another does not, compare them.

Questions:

```text
Does the client retry against another server?
Do both return the same answer?
Do they return different answers?
Does one respond significantly slower?
Does one return an error?
```

This can explain intermittent resolution behavior.

## Different Answers From Different Resolvers

Suppose:

```text
Resolver A → 203.0.113.10
Resolver B → 203.0.113.20
```

Do not automatically label one as incorrect.

Possible explanations include:

* Load balancing
* Geographic DNS
* Split-horizon DNS
* Caching differences
* CDN behavior
* Internal vs external DNS views
* DNS policy

The packet evidence shows that the resolvers returned different answers.

Understanding why requires additional network or DNS context.

## Split-Horizon DNS

Organizations may intentionally return different answers depending on where the client is located.

For example:

```text
Internal client
    ↓
Internal DNS
    ↓
Private address

External client
    ↓
External DNS
    ↓
Public address
```

If a user says:

```text
"It works from home but not from the office."
```

DNS differences may be part of the explanation.

Compare the observed queries and answers from each environment when captures are available.

## DNS Caching

Not every application request necessarily results in a DNS packet.

A cached answer may be used locally.

Therefore:

```text
Application request
↓
No DNS packet
↓
Connection to previously resolved address
```

does not prove that DNS was skipped incorrectly.

It may simply mean the answer was already cached.

When investigating DNS behavior, determine whether the relevant lookup was actually performed during the capture window.

## Search Domains

A hostname typed by a user may be transformed by resolver configuration.

For example, a short name:

```text
server1
```

may result in queries such as:

```text
server1.example.test
server1.corp.example.test
```

depending on the resolver configuration.

When unexpected DNS traffic appears, inspect the exact queried names rather than relying on what the user typed.

## Reverse DNS

Reverse DNS maps addresses back to names.

IPv4 reverse lookups use PTR records under:

```text
in-addr.arpa
```

IPv6 uses:

```text
ip6.arpa
```

Reverse lookups can appear during:

* Diagnostic tools
* Logging
* Security products
* Application behavior
* Network management

A reverse DNS query does not necessarily indicate that the application is trying to reach the resulting hostname.

Determine which process or protocol behavior triggered the lookup when that information is available.

## DNS Over TCP

DNS can use TCP, including cases such as:

* Larger DNS responses
* Truncated UDP responses followed by TCP
* Zone transfers
* Other protocol-specific conditions

A common pattern can be:

```text
UDP DNS query
↓
Response indicates truncation
↓
TCP connection to DNS server
↓
DNS query over TCP
↓
Response
```

When analyzing DNS, do not assume every transaction uses UDP.

## Encrypted DNS

Modern systems can use encrypted DNS mechanisms.

Examples include:

```text
DNS over TLS
DNS over HTTPS
```

Traditional DNS fields may no longer be visible in the same way.

You may instead observe:

```text
TLS connection
HTTPS connection
Encrypted application traffic
```

In such cases, Wireshark may still provide metadata and transport information, but the actual queried hostname may not be visible without appropriate decryption or additional endpoint evidence.

This is an important visibility limitation.

## DNS and TLS SNI

For encrypted application traffic, the hostname may sometimes be visible through TLS metadata such as SNI, depending on the protocol version and deployment.

Do not confuse:

```text
DNS query name
```

with:

```text
TLS server name indication
```

They are related pieces of information but occur at different protocol stages.

## DNS and IPv6 Preference

A hostname may resolve to both:

```text
A
AAAA
```

The client may attempt IPv6 first.

A useful investigation is:

```text
DNS:
AAAA → 2001:db8::50
A → 203.0.113.50

Connection:
IPv6 attempt → fails
IPv4 attempt → succeeds
```

The user may experience:

```text
Slow connection
Intermittent connection
Application fallback delay
```

The DNS records themselves may be valid.

The problem may instead be with the IPv6 path.

## DNS-Based Intermittent Behavior

Intermittent DNS problems can be difficult because successful and failed responses may alternate.

Example:

```text
Query → Response A
Query → Response A
Query → timeout
Query → Response A
Query → timeout
```

Investigate:

* Which resolver handled each query
* Query timing
* Response timing
* Source/destination addresses
* Record contents
* Whether different servers were involved
* Whether the application used different returned addresses

Do not average the behavior into:

```text
"DNS is mostly working."
```

Identify the conditions associated with failure.

## DNS Investigation With Statistics

Wireshark statistics can help answer:

```text
How much DNS traffic exists?
Which endpoints participate?
Which DNS servers are contacted?
Which names appear repeatedly?
Are there unusual bursts?
```

Useful areas include:

* Protocol hierarchy
* Endpoints
* Conversations
* DNS packet filtering
* I/O graphs
* Expert information

Statistics should support the packet-level investigation rather than replace it.

## DNS Investigation Workflow

Use this repeatable process:

```text
1. Define the hostname problem.
2. Identify the affected client.
3. Identify the expected DNS server.
4. Locate DNS traffic.
5. Identify the exact query.
6. Identify the record type.
7. Match the response to the query.
8. Check response code.
9. Inspect answer records.
10. Measure response timing.
11. Check retries.
12. Check whether another DNS server was used.
13. Check A vs AAAA behavior.
14. Follow the returned address into the next connection.
15. Compare with a successful resolution.
16. Document what is proven and what remains uncertain.
```

## Practical Exercise 1 — Basic DNS Query

Find a normal DNS transaction.

Record:

```text
Client:
DNS server:
Query name:
Record type:
Response code:
Answer:
Approximate response time:
```

Then identify the next network connection using the returned address if visible.

## Practical Exercise 2 — NXDOMAIN

Find a query that receives an `NXDOMAIN` response.

Determine:

```text
Requested name:
Record type:
DNS server:
Response code:
Did a subsequent connection occur?
```

Explain why an NXDOMAIN response is different from no response.

## Practical Exercise 3 — SERVFAIL

Find a `SERVFAIL` response.

Determine:

```text
Query:
Server:
Response:
Timing:
Retry behavior:
```

Then identify what additional evidence would be required to determine why the resolver returned `SERVFAIL`.

## Practical Exercise 4 — DNS Timeout

Find a transaction where the client retries a DNS query.

Record:

```text
Number of queries:
Time between attempts:
Did a response eventually appear?
Did the application proceed?
```

Do not assume the reason for the timeout without additional evidence.

## Practical Exercise 5 — A vs AAAA

Find a hostname that returns both A and AAAA records.

Determine:

```text
IPv4 result:
IPv6 result:
Which address family was attempted first?
Did both succeed?
Did one fail?
```

## Practical Exercise 6 — CNAME

Find a DNS response containing a CNAME.

Trace:

```text
Requested name
      ↓
CNAME
      ↓
Final address record
```

Record the final address used by the client if visible.

## Practical Exercise 7 — Multiple DNS Servers

Find a capture where the client contacts more than one DNS server.

Compare:

```text
Server A:
Response:
Timing:

Server B:
Response:
Timing:
```

Determine whether their behavior differs.

## Practical Exercise 8 — DNS and Application Correlation

Find:

```text
DNS query
↓
DNS response
↓
TCP or UDP connection
```

Determine whether the connection used an address returned by DNS.

## Practical Exercise 9 — DNS Failure vs Application Failure

Find a case where DNS succeeds but the application still fails.

Document:

```text
DNS result:
Returned address:
Transport result:
Application result:
Earliest observed failure:
```

This exercise reinforces the distinction between name resolution and application connectivity.

## Practical Exercise 10 — Encrypted DNS Visibility

If your lab contains encrypted DNS traffic, determine:

```text
What DNS metadata is visible?
What information is encrypted?
Can the queried name be directly identified?
What additional evidence would be required?
```

Do not infer DNS content from unrelated encrypted traffic without evidence.

## Professional DNS Evidence Record

Use this structure for investigations:

```text
Incident:
Date/Time:
Client:
Hostname:
DNS server:
Capture point:

Expected resolution:

Observed query:
Record type:

Observed response:
Response code:
Answer:

Query/response timing:

Retries:

Additional DNS servers:

A result:

AAAA result:

CNAME chain:

Subsequent connection:

Earliest observed failure:

Evidence:

Interpretation:

Remaining uncertainty:

Additional evidence required:
```

This format keeps the investigation reproducible.

## Common Mistakes

### Mistake 1: Calling Every Hostname Failure a DNS Failure

A hostname may resolve correctly while the subsequent TCP, TLS, or application transaction fails.

Always follow the complete sequence.

### Mistake 2: Ignoring the Response Code

A response is not automatically successful.

Inspect the DNS status and answer section.

### Mistake 3: Treating No Response as NXDOMAIN

These are completely different observations.

```text
No response:
Nothing observed from the resolver.

NXDOMAIN:
A DNS response explicitly indicates the queried name does not exist.
```

### Mistake 4: Ignoring A and AAAA Differences

IPv4 and IPv6 can produce different connection behavior.

Always determine which address family the client actually uses.

### Mistake 5: Ignoring CNAMEs

The queried name may not directly contain the final address.

Follow the resolution chain.

### Mistake 6: Assuming the First DNS Server Is the Only One

Clients and resolvers can use multiple DNS servers.

Inspect the actual traffic.

### Mistake 7: Assuming Every DNS Query Is Visible

Caching and encrypted DNS can prevent traditional DNS packets from appearing.

### Mistake 8: Treating Resolver Answers as Automatically Correct or Incorrect

A resolver may intentionally return different addresses based on DNS architecture.

Interpret answers in context.

### Mistake 9: Ignoring Timing

A DNS server can return a valid answer slowly enough to cause an application-level problem.

### Mistake 10: Confusing DNS With TLS

A hostname visible in TLS metadata does not mean a DNS query was observed.

These are separate protocol stages.

## Professional DNS Investigation Checklist

Before closing a DNS investigation:

* [ ] Client identified
* [ ] Hostname identified
* [ ] DNS server identified
* [ ] Capture point understood
* [ ] Query identified
* [ ] Record type identified
* [ ] Response correlated with query
* [ ] Response code checked
* [ ] Answer section inspected
* [ ] CNAME chain checked when applicable
* [ ] A record checked
* [ ] AAAA record checked
* [ ] Response timing measured
* [ ] Retries checked
* [ ] Multiple DNS servers considered
* [ ] Search-domain behavior considered when relevant
* [ ] Caching considered
* [ ] Encrypted DNS considered
* [ ] Subsequent connection checked
* [ ] Successful comparison performed when available
* [ ] Capture limitations documented
* [ ] Evidence separated from hypotheses
* [ ] Remaining uncertainty documented

## Completion Criteria

You should be able to independently investigate a DNS-related complaint and answer:

```text
What hostname was requested?

Who requested it?

Which DNS server handled it?

What record type was requested?

Did the resolver respond?

What response code was returned?

What answer was provided?

How long did resolution take?

Was the query retried?

Were multiple DNS servers involved?

Were A and AAAA results different?

Did a connection follow the DNS response?

Did the application actually use the returned address?

Where did the first meaningful failure occur?

What does the capture prove?

What remains uncertain?
```

The key mental model is:

```text
Hostname
   ↓
DNS Query
   ↓
DNS Response
   ↓
Address Selection
   ↓
Connection Attempt
   ↓
Transport
   ↓
TLS
   ↓
Application
```

DNS troubleshooting becomes much easier when the analyst stops asking:

```text
"Is DNS broken?"
```

and instead asks:

```text
"What did the client ask,
what did the resolver return,
how quickly did it respond,
and what happened next?"
```
