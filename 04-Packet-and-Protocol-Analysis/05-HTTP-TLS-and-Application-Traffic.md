# HTTP, TLS, and Application Traffic Analysis

## Objective

This file builds the ability to analyze application-layer traffic in Wireshark, with a focus on HTTP, TLS, and commonly encountered application protocols.

The goal is not to memorize every protocol field.

The goal is to answer practical questions such as:

* What application is communicating?
* Who is communicating with whom?
* What request was made?
* What response was returned?
* Did the server return an error?
* Was the client redirected?
* Did the TLS connection complete?
* What TLS metadata is visible?
* Where did an application failure occur?
* Is the traffic encrypted?
* What can and cannot be concluded from encrypted traffic?
* Is the observed behavior normal, unusual, or simply unexplained by the available evidence?

The core workflow remains:

```text
Question
    ↓
Relevant application traffic
    ↓
Filter
    ↓
Inspect request/response or handshake
    ↓
Follow the conversation/stream
    ↓
Correlate timing and lower-layer behavior
    ↓
Interpret
    ↓
Document evidence and limitations
```

## Application-Layer Mental Model

Application traffic sits above the transport layer.

A simplified model is:

```text
Application
    ↓
HTTP / TLS / DNS / SSH / SMTP / SMB / etc.
    ↓
TCP or UDP
    ↓
IP
    ↓
Ethernet / Wi-Fi
```

For example:

```text
Browser
  ↓
HTTP
  ↓
TLS
  ↓
TCP
  ↓
IP
  ↓
Ethernet
```

Not every application protocol uses TLS, TCP, or the same transport.

Examples:

```text
HTTP       → commonly TCP
HTTPS      → HTTP over TLS over TCP
DNS        → commonly UDP, sometimes TCP
DHCP       → UDP
SSH        → TCP
SMTP       → TCP, commonly with TLS depending on configuration
SMB        → TCP
SNMP       → commonly UDP
NTP        → UDP
```

Do not identify an application protocol solely from its port number.

A port is evidence.

The packet contents, protocol dissection, conversation behavior, and surrounding context provide stronger evidence.

## Protocol Identification

Wireshark can identify protocols using multiple sources of information.

Useful evidence includes:

* port numbers
* packet contents
* protocol signatures
* dissectors
* conversation behavior
* TLS handshake metadata
* application-layer fields
* protocol hierarchy

For example:

```text
TCP destination port 443
```

suggests HTTPS/TLS-related traffic.

It does not prove that the traffic is HTTPS.

Similarly:

```text
TCP destination port 22
```

suggests SSH.

It does not prove that the application is actually SSH.

Always distinguish:

```text
Expected protocol based on port
```

from:

```text
Observed protocol based on packet evidence
```

## HTTP Fundamentals

HTTP uses a request/response model.

A simplified exchange is:

```text
Client → HTTP Request → Server
Client ← HTTP Response ← Server
```

A request can contain information such as:

* method
* request URI
* host
* headers
* cookies
* body
* protocol version

A response can contain:

* status code
* headers
* cookies
* content type
* content length
* body
* protocol version

The exact visible fields depend on the HTTP version, capture point, encryption, and available packet data.

## HTTP Methods

Common HTTP methods include:

| Method | Typical purpose                                             |
| ------ | ----------------------------------------------------------- |
| GET    | Retrieve a resource                                         |
| POST   | Submit data or trigger an action                            |
| HEAD   | Retrieve response metadata without the normal response body |
| PUT    | Create or replace a resource                                |
| DELETE | Request deletion of a resource                              |
| PATCH  | Partially modify a resource                                 |

Do not interpret a method by itself.

For example:

```text
POST
```

only tells you the request method.

You still need to inspect:

* destination
* host
* URI
* headers
* body if visible
* response
* timing
* surrounding requests

## HTTP Request Analysis

When investigating an HTTP request, start with:

```text
Who sent it?
Where did it go?
What method was used?
What host was requested?
What URI was requested?
What headers are present?
Was a request body sent?
What response followed?
```

Useful Wireshark fields can include:

```text
http.request
http.request.method
http.host
http.request.uri
http.request.full_uri
http.user_agent
http.cookie
http.content_type
http.content_length
```

Field availability can vary depending on the HTTP version and capture.

A practical investigation might begin with:

```text
http.request
```

Then narrow the result:

```text
http.request.method == "GET"
```

or:

```text
http.request.method == "POST"
```

or:

```text
http.host
```

The important skill is not memorizing fields.

It is learning to move from:

```text
Broad question
```

to:

```text
Specific observable field
```

## HTTP Host and URI

The host and URI often provide important application context.

For example:

```text
Host: example.internal
URI: /login
```

can tell you that the client requested a login resource from a particular application.

Look for:

* host
* URI path
* query parameters
* HTTP method
* response status
* redirects
* repeated requests

Be careful with sensitive information.

HTTP may expose:

* session identifiers
* authentication data
* cookies
* personal information
* API parameters
* internal paths

Only inspect or handle traffic you are authorized to analyze.

## HTTP Status Codes

HTTP response status codes provide a useful first-level interpretation.

### 2xx

Generally indicates successful processing.

Examples:

```text
200 OK
201 Created
204 No Content
```

### 3xx

Generally indicates redirection or cache-related behavior.

Examples:

```text
301 Moved Permanently
302 Found
304 Not Modified
307 Temporary Redirect
308 Permanent Redirect
```

### 4xx

Generally indicates a client-side request or authorization-related problem.

Examples:

```text
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
405 Method Not Allowed
408 Request Timeout
429 Too Many Requests
```

### 5xx

Generally indicates a server-side processing failure.

Examples:

```text
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
```

These categories are useful starting points, not complete explanations.

For example:

```text
500
```

does not tell you why the server failed.

You must correlate the response with:

* request
* previous requests
* TCP behavior
* timing
* redirects
* repeated attempts
* other application traffic

## HTTP Request and Response Correlation

A common investigation pattern is:

```text
Request
  ↓
Server processing
  ↓
Response
```

When inspecting an HTTP response, determine which request caused it.

Look for:

* request sequence
* TCP stream
* timestamps
* request URI
* host
* response status
* packet relationships

Following the TCP stream is often useful when the HTTP exchange spans many packets.

## Following an HTTP Stream

When you find an interesting HTTP packet, use the packet context to inspect the associated stream.

Typical workflow:

```text
Interesting HTTP packet
        ↓
Identify TCP conversation
        ↓
Follow TCP Stream
        ↓
Review client/server exchange
        ↓
Return to packet-level evidence
```

Do not rely exclusively on the reconstructed stream.

The stream provides context.

Individual packets provide:

* timestamps
* sequence numbers
* retransmissions
* packet lengths
* TCP flags
* lower-layer evidence

A strong investigation uses both.

## HTTP Redirect Analysis

Redirects can explain unexpected application behavior.

Example:

```text
GET /login
    ↓
302
    ↓
Location: /auth
    ↓
GET /auth
    ↓
200
```

A redirect chain may be normal.

It may also reveal:

* incorrect configuration
* authentication flow
* HTTP-to-HTTPS transition
* repeated redirects
* redirect loops
* unexpected destinations

When investigating redirects, record:

```text
Original request
→ Response status
→ Location target
→ Follow-up request
→ Final response
```

## HTTP Error Investigation

For a visible HTTP error:

```text
HTTP 404
HTTP 403
HTTP 500
HTTP 502
HTTP 503
HTTP 504
```

do not immediately conclude that the network is broken.

Determine where the failure occurred.

A useful workflow is:

```text
Client request
    ↓
TCP connection established?
    ↓
HTTP request transmitted?
    ↓
Server response received?
    ↓
HTTP status code
    ↓
Response timing
    ↓
Repeated attempts?
```

Examples:

```text
TCP connection succeeds
HTTP request succeeds
Server returns 404
```

This points toward an application/resource issue rather than a basic TCP connectivity failure.

Another case:

```text
TCP connection succeeds
HTTP request sent
Long delay
504 response
```

requires further investigation of the application path, gateway, upstream service, and timing.

## HTTP Timing

Timing is often more useful than simply looking at packet contents.

Questions include:

```text
How long between request and response?
How long between TCP connection and HTTP request?
How long between redirects?
Are repeated requests becoming slower?
Does the server respond immediately with an error?
```

Use packet timestamps and stream context.

Do not call a response "slow" without considering:

* network latency
* server processing
* client behavior
* connection establishment
* retransmissions
* application design
* capture location

## HTTP/2

Modern web traffic may use HTTP/2 instead of HTTP/1.1.

HTTP/2 changes the representation of application traffic.

Important concepts include:

* streams
* frames
* multiplexing
* headers
* concurrent requests
* connection reuse

Multiple logical requests can share one TCP connection.

Therefore, a single TCP stream does not necessarily represent one application request.

When analyzing HTTP/2, think in terms of:

```text
TCP connection
    ↓
HTTP/2 session
    ↓
Multiple logical streams
    ↓
Requests and responses
```

Do not assume:

```text
one TCP connection = one HTTP request
```

## TLS Fundamentals

TLS provides security services such as:

* confidentiality
* integrity
* authentication

A simplified HTTPS model is:

```text
HTTP
  ↓
TLS
  ↓
TCP
  ↓
IP
```

After TLS is established, application data is normally encrypted.

This creates an important analytical boundary:

```text
Visible metadata
        ≠
Visible application content
```

You may still be able to determine a great deal without decrypting the traffic.

## TLS Handshake

A simplified TLS handshake can include:

```text
ClientHello
    ↓
ServerHello
    ↓
Certificate
    ↓
Key exchange / handshake messages
    ↓
Encrypted application data
```

The exact handshake differs by TLS version and configuration.

Do not assume that every handshake contains exactly the same messages.

The important investigative questions are:

```text
Did the handshake begin?
Did the server respond?
What TLS version was negotiated?
What cipher suite was selected?
Was a certificate presented?
Was a server name visible?
Did the handshake complete?
Did an alert occur?
Did encrypted application data follow?
```

## ClientHello

The ClientHello can expose useful metadata.

Depending on the TLS version and capture, you may see:

* supported versions
* cipher suites
* extensions
* server name indication
* supported groups
* signature algorithms
* session-related information

A useful field for visible SNI is:

```text
tls.handshake.extensions_server_name
```

SNI can provide application context even when HTTP content itself is encrypted.

SNI availability depends on protocol version, configuration, and capture visibility.

Do not assume that absence of SNI proves the absence of a particular application.

## ServerHello

The ServerHello represents the server's response to the TLS negotiation.

It can provide evidence about:

* negotiated TLS version
* selected cipher suite
* selected parameters

The exact fields vary by TLS version.

The useful question is:

```text
What did the client offer?
What did the server select?
```

## TLS Versions

Common TLS versions encountered in modern environments include:

```text
TLS 1.2
TLS 1.3
```

Older protocol versions may appear in historical or legacy environments.

When analyzing a TLS session, record the observed version rather than assuming the expected version.

A useful investigation question is:

```text
Did the client and server successfully negotiate a mutually supported version?
```

## Cipher Suites

Cipher suite information can help characterize a TLS session.

Do not treat a cipher suite name as the entire security assessment.

Record:

* negotiated cipher suite
* TLS version
* relevant handshake metadata
* certificate information
* connection behavior

Interpret the information in context.

## Certificates

TLS certificates can provide evidence such as:

* subject
* issuer
* validity period
* public-key information
* certificate chain information
* extensions

A certificate is evidence about the TLS endpoint and authentication configuration.

Do not automatically conclude:

```text
Certificate exists = certificate is valid
```

or:

```text
Certificate name matches = application is trustworthy
```

Certificate validation depends on factors such as:

* hostname validation
* trust chain
* validity period
* client trust store
* certificate usage
* application behavior

Wireshark can show certificate information available in the captured handshake, but it does not necessarily reproduce the exact validation decision made by the client.

## TLS Alerts

TLS alerts can indicate handshake or protocol problems.

When investigating an alert, determine:

```text
Which side sent it?
When was it sent?
What happened immediately before it?
Did the TCP connection close?
Was application data exchanged?
Was the alert repeated?
```

Do not treat an alert as a complete root-cause explanation.

Correlate it with:

* ClientHello
* ServerHello
* certificate messages
* supported versions
* TCP behavior
* timing
* connection termination

## TLS Handshake Failure Workflow

When HTTPS fails, use this sequence:

```text
1. Identify the client and server.
2. Confirm TCP connectivity.
3. Locate ClientHello.
4. Check for ServerHello.
5. Check negotiated TLS information.
6. Inspect certificate-related messages if visible.
7. Look for TLS alerts.
8. Check TCP resets or retransmissions.
9. Compare timing.
10. Determine the earliest observable failure.
```

The key principle is:

> Find the earliest failure supported by packet evidence.

## Encrypted Application Traffic

Even when application content is encrypted, useful information can remain visible.

Depending on the protocol and capture point, you may observe:

* source and destination
* ports
* timestamps
* packet lengths
* TCP flags
* retransmissions
* TLS version
* cipher information
* SNI
* certificate metadata
* handshake behavior
* connection duration
* traffic direction
* packet frequency

You may be able to infer:

```text
A client established a TLS connection to a server.
```

You may not be able to infer:

```text
The exact URL requested.
```

unless the relevant information is otherwise visible or the traffic can be legitimately decrypted.

This distinction is essential for professional analysis.

## TLS Traffic Without Decryption

A practical workflow is:

```text
Identify endpoint pair
        ↓
Identify TCP stream
        ↓
Locate TLS handshake
        ↓
Inspect visible metadata
        ↓
Measure timing
        ↓
Observe packet sizes and direction
        ↓
Check alerts/retransmissions/resets
        ↓
State what is observable
        ↓
State what remains unknown
```

Do not invent application-level conclusions from encrypted payloads.

## Decryption and Key Material

Wireshark can analyze decrypted TLS traffic when appropriate key material and protocol conditions make decryption possible.

For authorized lab or troubleshooting environments, this can provide additional visibility.

However:

```text
Encrypted capture
```

and:

```text
Decryptable capture
```

are not equivalent.

Modern TLS designs can make passive decryption impossible without appropriate session secrets or other supported mechanisms.

Always document whether your analysis is based on:

```text
Encrypted metadata
```

or:

```text
Decrypted application content
```

## SSH Traffic

SSH commonly uses TCP and encrypts its application data.

Wireshark may still provide useful information about:

* endpoints
* TCP connection
* SSH identification exchange when visible
* connection timing
* packet sizes
* retransmissions
* resets
* connection duration

After encryption is established, command-level content is normally not visible from an ordinary capture.

Therefore:

```text
SSH connection observed
```

does not mean:

```text
SSH commands observed
```

## FTP Traffic

FTP commonly separates control and data behavior.

When analyzing FTP, look for:

* control connection
* commands
* responses
* authentication exchanges
* data connections
* transfer timing

Plain FTP can expose application-level information directly.

This makes protocol analysis straightforward but also creates significant confidentiality concerns.

Only inspect authorized traffic.

## SMTP Traffic

SMTP is used for mail transfer.

Useful analysis questions include:

```text
Who is communicating?
Which SMTP commands are visible?
Was authentication attempted?
Did the server accept or reject the message?
Was TLS negotiated?
What response codes were returned?
```

Depending on encryption and configuration, application content may be unavailable.

## SMB Traffic

SMB is commonly encountered in Windows and enterprise environments.

Useful investigation areas include:

* client/server relationships
* SMB dialect information
* session establishment
* authentication behavior
* file/service operations when visible
* errors
* timing
* repeated failures

SMB analysis can become complex quickly.

Start with:

```text
Who?
To whom?
When?
Which SMB version?
Which operation?
What response?
What error?
```

Then correlate with TCP and authentication traffic.

## LDAP and Kerberos

LDAP is commonly associated with directory services.

Kerberos is commonly used for authentication in Active Directory environments.

For practical Wireshark analysis, focus on:

* endpoints
* request/response behavior
* authentication-related exchanges
* errors
* timing
* repeated requests
* protocol relationships

Do not attempt to interpret every protocol field before understanding the overall communication flow.

## SNMP

SNMP is commonly used for network management and monitoring.

Useful questions include:

```text
Which device is querying?
Which device is responding?
What operation is being performed?
Are requests repeated?
Are errors returned?
What timing pattern exists?
```

Depending on the SNMP version and configuration, sensitive values may or may not be visible.

## NTP

NTP is used for time synchronization.

When analyzing NTP, inspect:

* client/server relationship
* request/response behavior
* timing
* repeated requests
* unusual destinations
* failures

Time synchronization can also matter during forensic investigations because inaccurate system clocks can affect event correlation.

## Application-Layer Correlation

Do not analyze application protocols in isolation.

For example:

```text
DNS
 ↓
TCP
 ↓
TLS
 ↓
HTTP/application traffic
```

A website failure may involve several layers.

A practical investigation can move through:

```text
DNS resolution
    ↓
TCP connection
    ↓
TLS handshake
    ↓
HTTP request
    ↓
HTTP response
```

If the failure occurs before HTTP, the HTTP layer cannot explain the failure.

## Website Failure Workflow

When a user reports:

```text
"The website does not work."
```

do not immediately filter on HTTP.

Use a layered workflow:

```text
1. Identify the client.
2. Identify the intended destination.
3. Check DNS resolution.
4. Check TCP connection establishment.
5. Check TLS handshake if HTTPS is used.
6. Check HTTP request.
7. Check HTTP response.
8. Check redirects.
9. Check timing and retransmissions.
10. Identify the earliest observable failure.
```

Example:

```text
DNS succeeds
TCP succeeds
TLS succeeds
HTTP request succeeds
HTTP 503 returned
```

The evidence shows that basic network connectivity and TLS establishment occurred before the visible application-level failure.

## Slow Web Application Workflow

For a slow application:

```text
1. Identify the user request.
2. Identify the relevant TCP stream.
3. Measure connection establishment.
4. Measure TLS handshake if applicable.
5. Measure request-to-response timing.
6. Check retransmissions.
7. Check delayed responses.
8. Check redirects.
9. Compare repeated requests.
10. Determine where the delay is observed.
```

Separate:

```text
Network delay
```

from:

```text
Server/application processing delay
```

as far as the capture permits.

Do not automatically blame the network.

## HTTP 4xx/5xx Investigation

For an application error:

```text
Find error response
    ↓
Identify corresponding request
    ↓
Inspect URI/method
    ↓
Inspect timing
    ↓
Check previous requests
    ↓
Check redirects
    ↓
Check TCP health
    ↓
Determine earliest relevant evidence
```

Example:

```text
GET /api/data
    ↓
500 Internal Server Error
```

This is evidence of an HTTP server-side error response.

It is not, by itself, proof of a particular backend failure.

## TLS Failure Investigation

For a TLS failure:

```text
TCP handshake
    ↓
ClientHello
    ↓
Server response?
    ↓
Negotiation?
    ↓
Certificate?
    ↓
TLS alert?
    ↓
TCP termination?
```

Record the exact observed sequence.

For example:

```text
TCP handshake succeeds
ClientHello sent
No ServerHello observed
TCP retransmissions occur
Connection resets
```

This is stronger evidence than simply writing:

```text
HTTPS is broken.
```

## HTTP and TLS Filtering

Useful starting filters can include:

```text
http
```

```text
http.request
```

```text
http.response
```

```text
http.request.method == "GET"
```

```text
http.request.method == "POST"
```

```text
http.response.code >= 400
```

```text
tls
```

```text
tls.handshake
```

```text
tls.handshake.type == 1
```

```text
tls.handshake.extensions_server_name
```

Use broad filters first when exploring an unfamiliar capture.

Then narrow them as your question becomes more specific.

## Combining Application and Transport Filters

Application-layer analysis becomes more powerful when combined with transport evidence.

Examples:

```text
http && tcp.analysis.retransmission
```

```text
tls && tcp.flags.reset == 1
```

```text
http.response.code >= 500 && tcp
```

The purpose is not to create complicated filters.

The purpose is to answer a specific question.

For example:

```text
Are HTTP errors occurring on connections that also show TCP retransmissions?
```

The filter supports the question.

It does not answer the question automatically.

## Security Analysis of HTTP Traffic

When analyzing authorized traffic, look for evidence such as:

* unexpected destinations
* unusual HTTP methods
* suspicious URIs
* repeated failed requests
* unusual user agents
* unexpected redirects
* sensitive information transmitted without expected protection
* unusual request frequency
* unexpected external communication

Do not label traffic malicious from a single unusual field.

Use multiple pieces of evidence:

```text
Destination
+
Timing
+
Protocol
+
URI
+
Request pattern
+
Response behavior
+
Context
```

## Security Analysis of TLS Traffic

Encrypted traffic limits content visibility, but useful metadata may remain.

Investigate:

* unexpected destinations
* unusual TLS endpoints
* unusual connection frequency
* unexpected SNI values
* certificate anomalies
* repeated failed handshakes
* unusual handshake patterns
* resets after handshake
* long-lived connections
* unusual traffic volumes

Avoid statements such as:

```text
This encrypted connection is malicious.
```

unless the evidence actually supports that conclusion.

Instead document:

```text
The host established repeated TLS connections to an unexpected destination.
The application payload was encrypted and could not be inspected.
The observed evidence is insufficient to determine the application content.
```

## Protocol Recognition Exercise

Use a supplied or generated capture containing several application protocols.

For each conversation, determine:

```text
Source
Destination
Transport
Port
Observed protocol
Evidence for identification
```

Create a table:

| Source   | Destination | Transport | Port | Protocol | Identification Evidence |
| -------- | ----------- | --------- | ---: | -------- | ----------------------- |
| Client A | Server A    | TCP       |  443 | TLS      | TLS handshake           |
| Client A | Server B    | TCP       |   22 | SSH      | SSH protocol evidence   |
| Client B | DNS Server  | UDP       |   53 | DNS      | DNS query/response      |
| Client C | Mail Server | TCP       |   25 | SMTP     | SMTP exchange           |

The point is to practice identifying protocols from evidence rather than ports alone.

## Practical Exercise 1: HTTP Request Analysis

Find an HTTP request.

Determine:

```text
Source
Destination
Method
Host
URI
User-Agent
Response code
```

Then follow the associated TCP stream.

### Success Criteria

You can explain the complete request/response exchange without relying only on the packet list.

## Practical Exercise 2: HTTP Error Investigation

Find an HTTP 4xx or 5xx response.

Determine:

```text
Request
Response
Status code
Timing
TCP stream
Previous related requests
```

Then explain the earliest observable failure.

## Practical Exercise 3: Redirect Chain

Find a request that produces a redirect.

Document:

```text
Initial URI
Redirect status
Location
Follow-up request
Final response
```

Determine whether the redirect chain appears complete or unusual.

## Practical Exercise 4: TLS Handshake

Find a TLS handshake.

Record:

```text
Client
Server
TLS version
Cipher information
SNI if visible
Certificate information if visible
Handshake completion
```

Then identify whether encrypted application data follows.

## Practical Exercise 5: TLS Failure

Find a TLS connection that does not complete successfully.

Determine:

```text
ClientHello present?
Server response?
TLS alert?
TCP reset?
Retransmission?
Timing?
```

Write a short evidence-based explanation.

## Practical Exercise 6: Encrypted Traffic Analysis

Choose an encrypted application conversation.

Without decrypting it, determine:

```text
Endpoints
Port
Protocol
TLS metadata
Connection duration
Packet count
Traffic direction
Approximate packet-size behavior
```

Then write two sections:

```text
What I can determine
What I cannot determine
```

This is an important professional skill.

## Practical Exercise 7: Slow Web Request

Find a web request with noticeable delay.

Determine:

```text
TCP connection time
TLS handshake time if applicable
Request timestamp
Response timestamp
Retransmissions
Final response
```

Decide where the observable delay occurs.

Do not assign a root cause unless the capture supports it.

## Practical Exercise 8: Multi-Layer Website Investigation

Investigate one complete website transaction.

Trace:

```text
DNS
 ↓
TCP
 ↓
TLS
 ↓
HTTP
```

For each layer, record:

```text
What happened?
What evidence supports it?
Was there a problem?
```

This exercise connects the protocol-analysis files into one workflow.

## Practical Exercise 9: Application Protocol Identification

Use an unfamiliar capture.

Do not begin with port numbers.

First inspect:

```text
Protocol hierarchy
Packet dissection
Conversation behavior
Application-layer signatures
```

Then identify several protocols.

For each one, explain why you identified it.

## Practical Exercise 10: Security-Oriented Application Analysis

Use an authorized lab capture containing HTTP and TLS traffic.

Investigate:

```text
Unexpected destinations
Repeated requests
Unusual URIs
Redirects
Failed authentication-related requests
TLS metadata
Connection frequency
```

Separate:

```text
Observation
```

from:

```text
Interpretation
```

and:

```text
Conclusion
```

Do not make a stronger claim than the evidence supports.

## Application Traffic Investigation Checklist

### Identification

* [ ] Identify client.
* [ ] Identify server.
* [ ] Identify transport protocol.
* [ ] Identify application protocol.
* [ ] Confirm protocol using packet evidence.

### HTTP

* [ ] Identify request method.
* [ ] Identify host.
* [ ] Identify URI.
* [ ] Inspect relevant headers.
* [ ] Identify response status.
* [ ] Correlate request and response.
* [ ] Follow the relevant stream when useful.
* [ ] Inspect redirects.
* [ ] Inspect timing.
* [ ] Check repeated requests.

### TLS

* [ ] Locate ClientHello.
* [ ] Locate ServerHello.
* [ ] Identify negotiated TLS version.
* [ ] Identify cipher information.
* [ ] Check SNI if visible.
* [ ] Inspect certificate information if visible.
* [ ] Check for alerts.
* [ ] Check whether encrypted application data follows.
* [ ] Check TCP behavior.
* [ ] Document encryption limitations.

### Troubleshooting

* [ ] Check DNS first when relevant.
* [ ] Check TCP establishment.
* [ ] Check TLS establishment.
* [ ] Check application request.
* [ ] Check application response.
* [ ] Check timing.
* [ ] Check retransmissions.
* [ ] Identify the earliest observable failure.

### Security

* [ ] Identify unusual destinations.
* [ ] Check unusual request patterns.
* [ ] Check unexpected redirects.
* [ ] Check repeated failures.
* [ ] Inspect visible sensitive information carefully.
* [ ] Check TLS metadata.
* [ ] Separate observation from interpretation.
* [ ] State analysis limitations.

## Professional Reasoning Pattern

A strong application-layer investigation should look like this:

```text
Question:
Why did the web request fail?

Evidence:
DNS resolution succeeded.
TCP connection established.
TLS handshake completed.
HTTP GET was sent.
Server returned HTTP 503.

Interpretation:
The capture shows successful lower-layer connectivity and TLS establishment before the application returned a service-unavailable response.

Conclusion:
The earliest visible failure is at the HTTP/application layer.

Limitation:
The capture does not establish why the server generated the 503 response.
```

This is much stronger than:

```text
The network is fine and the server is broken.
```

The second statement goes beyond the available evidence.

## Common Mistakes

### Mistake 1: Assuming Port 443 Means HTTPS

Port 443 is a useful clue.

It is not proof.

### Mistake 2: Assuming HTTP Errors Are Network Errors

An HTTP 404, 500, or 503 is an application-layer observation.

Investigate the lower layers separately.

### Mistake 3: Treating TLS as Completely Invisible

TLS encrypts application content, but metadata can remain visible.

### Mistake 4: Assuming Visible TLS Metadata Reveals Everything

SNI, certificates, timing, and packet sizes do not reveal the full application conversation.

### Mistake 5: Blaming TCP for Every Slow Application

An application can be slow even when TCP is operating normally.

### Mistake 6: Treating One Packet as the Whole Story

Application protocols are conversations.

Correlate packets, streams, timestamps, and responses.

### Mistake 7: Confusing Observation With Interpretation

```text
Observation:
HTTP 503 returned.

Interpretation:
The application/service was unavailable at that moment.

Unsupported conclusion:
The backend database is definitely down.
```

Only the first two are directly supported by the observed exchange.

## Completion Criteria

You are ready to move forward when you can independently:

* identify common application protocols
* distinguish protocol identification from port-based assumptions
* analyze HTTP requests and responses
* interpret HTTP status codes
* follow HTTP conversations
* analyze redirects
* investigate HTTP errors
* measure application timing
* identify TLS handshakes
* identify TLS versions and cipher information
* inspect SNI when visible
* inspect certificate metadata when available
* identify TLS alerts and failures
* analyze encrypted traffic without overclaiming
* correlate application traffic with TCP behavior
* recognize common application protocols
* investigate website failures layer by layer
* investigate slow web applications
* distinguish network failures from application failures
* document what is known and unknown
* perform security-oriented application traffic analysis responsibly

The key skill is not memorizing HTTP or TLS fields.

The key skill is being able to move from:

```text
Application problem
```

to:

```text
DNS
→ TCP
→ TLS
→ HTTP
→ Response
→ Timing
→ Evidence
→ Conclusion
```

without skipping layers or making unsupported assumptions.
