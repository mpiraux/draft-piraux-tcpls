---
title: "TCPLS: Modern Transport Services with TCP and TLS"
abbrev: "TCPLS"
docname: draft-piraux-tcpls-latest
category: info

ipr: trust200902
area: Transport Area
workgroup: TODO Working Group
keyword: Internet-Draft

stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
 -
    name: Maxime Piraux
    organization: UCLouvain
    email: maxime.piraux@uclouvain.be
 -
    name: Olivier Bonaventure
    organization: UCLouvain
    email: olivier.bonaventure@uclouvain.be
 -
    name: Florentin Rochet
    organization: University of Edinburgh
    email: frochet@ed.ac.uk

normative:
  RFC8446:

informative:
  RFC4960:
  RFC6335:
  RFC7258:
  RFC8548:
  RFC8684:
  RFC9000:
  RFC7540:


--- abstract

This document specifies a protocol leveraging TCP and TLS to provide modern
transport services such as multiplexing, connection migration and multipath
in a secure manner.

--- middle

# Introduction

The TCP/IP protocol stack continuously evolves. In the early days, most
applications were interacting with the transport layer (mainly TCP, but
also UDP) using the socket API. This is illustrated in {{fig-intro-old}}.

~~~~~~~~~~~~~~~~~~~~~~~~~~~
+------------------------------+
|          Application         |
+------------------------------+
|            TCP/UDP           |
+------------------------------+
|             IPv4             |
+------------------------------+
~~~~~~~~~~~~~~~~~~~~~~~~~~~
{: #fig-intro-old title="The classical TCP/IP protocol stack"}

The TCP/IP stack has slowly evolved and the figure above does not
anymore describe current Internet applications. IPv6 is now widely
deployed next to IPv4 in the network layer. In the transport layer,
protocols such as SCTP {{RFC4960}} or DCCP {{RFC6335}} and TCP
extensions including Multipath TCP {{RFC8684}} or tcpcrypt {{RFC8548}}
have been specified. The security aspects of the TCP/IP protocol suite
are much more important today than in the past {{RFC7258}}.  Many
applications rely on TLS {{RFC8446}} and their stack is
similar to the one shown in {{fig-intro-today}}.

~~~~~~~~~~~~~~~~~~~~~~~~~~~
+------------------------------+
|          Application         |
+------------------------------+
|             TLS              |
+------------------------------+
|             TCP              |
+------------------------------+
|         IPv4 and IPv6        |
+------------------------------+
~~~~~~~~~~~~~~~~~~~~~~~~~~~
{: #fig-intro-today title="Today's TCP/IP protocol stack"}

Recently, the IETF went one step further in improving the transport
layer with the QUIC protocol {{RFC9000}}. QUIC is a new secure transport
protocol primarly designed for HTTP/3. It includes the reliability and
congestion control features that are part of TCP and integrates the
security features of TLS 1.3 {{RFC8446}}. This close integration between
the reliability and security features brings a lot of benefits in QUIC.
QUIC runs above UDP to be able to pass through most middleboxes and to be
implementable in user space.

~~~~~~~~~~~~~~~~~~~~~~~~~~~
+------------------------------+
|          Application         |
+------------------------------+
|..........                    |
|   TLS   |   QUIC   ..........|
|..........          |   UDP   |
+------------------------------+
|         IPv4 and IPv6        |
+------------------------------+
~~~~~~~~~~~~~~~~~~~~~~~~~~~
{: #fig-intro-quic title="QUIC protocol stack"}

In this document, we revisit how TCP and TLS 1.3 can be used to
provide modern transport services to applications. We apply a similar principle
and combine TCP and TLS 1.3 in a protocol that we call TCPLS.
TCPLS leverages the security features of TLS 1.3 like QUIC. In addition,
TCPLS reuses the existing high performance TCP stacks and TCP's wider
ability to pass through middleboxes.

~~~~~~~~~~~~~~~~~~~~~~~~~~~
+------------------------------+
|          Application         |
+------------------------------+
|..........                    |
|   TLS   |   TCPLS  ..........|
|..........          |   TCP   |
+------------------------------+
|         IPv4 and IPv6        |
+------------------------------+
~~~~~~~~~~~~~~~~~~~~~~~~~~~
{: #fig-intro-tcpls title="TCPLS in the TCP/IP protocol stack"}

In this document, we will use the term TLS/TCP to refer to the TLS 1.3
protocol running over a TCP connection. We reserve the word TCPLS for
the protocol that is proposed in this document.

This document is organised as follows. First, {{overview}} gives an overview
of TCPLS. Then, {{services}} describes the modern transport services provided by
TCPLS. Finally, {{format}} describes the format of TCPLS and its TLS Extensions
introduced in this document.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# TCPLS Overview {#overview}

In order for TCPLS to be widely compatible with middleboxes that inspect TCP
segments and TLS records, TCPLS does not modify the TCP connection establishment
and only adds a TLS extension to the TLS handshake. {{fig-overview-handshake}}
illustrates the opening of a TCPLS session which starts with the TCP
3-way handshake, followed by the TLS handshake. In
the Extensions of the ClientHello and in the server EncryptedExtensions, the
tcpls_hello TLS Extension is introduced to announce the support of TCPLS.

~~~~~~~~~~~~~~~~~~~~~~~~~~~
Client                                   Server
 |                    SYN                    |
 |------------------------------------------>|
 |                  SYN+ACK                  |
 |<------------------------------------------|
 |    ACK, TLS ClientHello + tcpls_hello     |
 |------------------------------------------>|
 |  TLS ServerHello, TLS EncryptedExtensions |
 |                       + tcpls_hello, ...  |
 |<------------------------------------------|
 |               TLS Finished                |
 |------------------------------------------>|
 |                                           |
~~~~~~~~~~~~~~~~~~~~~~~~~~~
{: #fig-overview-handshake title="Starting a TCPLS session"}

TCP/TLS offers a single encrypted bytestream service to the application. To
achieve this, TLS records are used to encrypt and secure chunks of the
application bytestream and are then sent through the TCP bytestream. TCPLS
leverages TLS records in a different way. It allows exchanging control and
application data inside TLS records with TCPLS frames, forming the basic
units of TCPLS. A TLS record can be composed of several frames, which cannot
span several records.

TCPLS is not restricted to using a single TCP connection to exchange these
frames, as it introduces Head-of-Line blocking between the frames and limit the
use of the protocol. This documents also specifies how several TCP connections
can be joined to a TCPLS session. This document illustrates how this feature
can be leveraged to provide Connection Migration and Multipath capabilities.

# Modern Transport Services {#services}

Application requirements and the devices they run on evolve over time. In
the early days, most applications involved single-file transfer and ran on
single-homed computers with a fixed-line network. Today, applications such as
HTTP require to exchange multiple objects, with different priorities, on
devices that can move from one access network to another and that often
have multiple access networks available.

TCPLS draws on the lessons learned from the design of SCTP, MPTCP
and QUIC to provide three modern transport services atop TCP and TLS to
applications in a secure manner.

## Multiplexing

TCPLS expands the service provided by TCP with streams. Streams are
independent bidirectional bytestreams that can be used by applications to
concurrently convey several objects over a TCPLS session.
Streams can be opened by the client and by the server.

Streams are identified by a 32-bit unsigned integer. The parity of this number
indicates the initiator of the stream. The client opens even-numbered
streams while the server opens odd-numbered streams. Streams are opened in
sequence, e.g. a client that has opened stream 0 will use stream 2 has the
next one.

Data of the streams is exchanged using Stream frames, for which their format is
described in {{stream-frame}}. Each Stream frame carries a chunk of data of
a given stream. Applications can close a stream and set its end, which
senders encode in the Stream frame.

Similarly to HTTP/2 {{RFC7540}}, conveying several streams on a single TCP
connection introduces Head-of-Line (HoL) blocking between the streams. To
alleviate this, TCPLS provides means to the application to choose the degree
of HoL blocking resilience it needs for its application objects. These means
are described in the following sections.

## Connection Migration

TCPLS enables the application to add new TCP connections to the TCPLS
session. The simplest use of an other TCP connection is to fall over it in
the advent of a network failures or due to application policy. This section
describes the mecanisms needed to achieve this.

### Joining TCP connections

The TCPLS server can provide tokens to the client in order to join new TCP
connections to the TCPLS session. {{join-connections}} illustrates a client
and server first establishing a new TCPLS session as described in {{overview}}.
Then the server sends a token over this connection using the New Token
frame. The token has a sequence number (i.e. 1) and a value (i.e. "abc").
The client uses this token to open a new TCP connection and initiates the
TCPLS handshake. It adds the token inside the TCPLS Join TLS extension in the
ClientHello.

~~~~~~~~~~~~~~~~~~~~~~~~~~~
             <-1.TCPLS Handshake->
       .---------------------------------.
       |            <-2.New Token(1,abc) |
       v                                 v
+--------+                            +--------+
| Client |                            | Server |
+--------+                            +--------+
       ^                                 ^
       | 3.TCPLS Handshake + Join(abc)-> |      Legend:
       .---------------------------------.        --- TCP connection
~~~~~~~~~~~~~~~~~~~~~~~~~~~
{: #join-connections title="Joining a new TCP connection"}

When receiving a TCPLS Join Extension, the server validates the token and
associate the TCP connection to the TCPLS session. TCP connections of TCPLS
session are identified by a 32-bit unsigned integer called Connection ID. The
first connection of a session has a value of 0. When joining a new connection,
the sequence number of the token, i.e. 1 in our example, become the
Connection ID of the connection.

### Record protection

When adding new TCP connections to a TCPLS session, an endpoint does not
complete the TLS handshake. TCPLS provides a nonce construction for TLS
record protection that is used for all connections of a session. This
reduces the cryptographic cost of adding connections. The endpoints SHOULD send
realistic but random TLS messages to form an apparent complete TLS handshake
to middleboxes.

In order to expand the TLS session to multiple connections, TCPLS adds a
record sequence number space per connection that is maintained independently at
both sides. Each record sent over a TCPLS session is identified by the
Connection ID of its connection and its record sequence number. Each record
nonce is constructed as defined in {{fig-record-nonce}}.

~~~
N                  N-32                   64                    0
+---------------------------------------------------------------+
|                    client/server_write_iv                     |
+---------------------------------------------------------------+
         XOR                                        XOR
+-------------------+                      +--------------------+
|   Connection ID   |                      | Conn. record sequ. |
+-------------------+                      +--------------------+
~~~
{: #fig-record-nonce title="TCPLS TLS record nonce construction"}

This construction guarantees that every TLS record sent over the TLS session
is protected with a unique nonce. As in TLS 1.3, the per-connection record
sequence is implicit.

### Record Acknowledgements

The receiver frequently acknowledges the records received using the ACK
frame. Records are acknowledged after the record protection has been
successfully removed. This enables the sender to know which records have been
received. TCPLS enables the endpoint to send acknowledgments for a TCP
connection over any connections, e.g. not only the receiving connection.

### Migration

To migrate from a given TCP connection, an endpoint stops transmitting
over this TCP connection and sends the following frames on other TCP
connections. It leverages the acknowledgements to retransmit the frames of
TLS records that have not been yet acknowledged.

When an endpoint abortfully closes a TCP connection, its peer leverages the
acknowlegments to retransmit the TLS records that were not acknowlegded.

## Multipath

When the endpoints have opened several TCP connections, they can send frames
over the connections. By sending frames of a given stream on two or more
connections, an endpoint can benefit from the aggregated bandwidth of the
connections.

# TCPLS Protocol {#format}

## TCPLS TLS Extensions

This document specifies two TLS extensions used by TCPLS. The first,
tcpls_hello, is used to announce the support of TCPLS. The second,
tcpls_join, is used to join a TCP connection to a TCPLS session.

### TCPLS Hello

The "tcpls_hello" extension is used by the client and the server to announce
their support of TCPLS. The extension contains no value. When it is present
in both the ClientHello and the EncryptedExtensions, the endpoints MUST use
TCPLS after completing the TLS handshake.

### TCPLS Join

~~~~
struct {
    opaque token<32>;
} Join;
~~~~

The "tcpls_join" extension is used by the client to join the TCP connection
on which it is sent to a TCPLS session. The extension contains a Token
provided by the server. The client MUST NOT send more than one
tcpls_join extension in its ClientHello. When receiving a ClientHello with
this extension, the server checks that the token is valid and joins the TCP
connection to the corresponding TCPLS session. When the token is not valid,
the server MUST abort the handshake with an illegal_parameter alert.

By controlling the amount of tokens given to the client, the server can
control the number of active TCP connections of a session. The server SHOULD
replenish the tokens

## TCPLS Frames

TCPLS uses TLS Application Data records to exchange TCPLS frames. After
decryption, the record payload consists of a sequence of TCPLS frames. A
frame is a Type-Value unit, starting with a byte indicating its frame type
followed by type-specific fields. {{tcpls-frame-types}} lists the frames
specified in this document.

| Type value | Frame name        | Rules | Definition                  |
|:-----------|:------------------|:------|:----------------------------|
| 0x00       | Padding           | A     | {{padding-frame}}           |
| 0x01       | Ping              |       | {{ping-frame}}              |
| 0x02-0x03  | Stream            |       | {{stream-frame}}            |
| 0x04       | ACK               | A     | {{ack-frame}}               |
| 0x05       | New Token         |  S    | {{new-token-frame}}         |
| 0x06       | New Address       |       | {{new-address-frame}}       |
| 0x07       | Connection Failed |       | {{connection-failed-frame}} |
{: #tcpls-frame-types title="TCPLS frames"}

The "Rules" column in {{tcpls-frame-types}} indicates special requirements
regarding certain frames.

A:

: Non-ack-eliciting. Receiving this frame does not elicit the sending of an
acknowledgment.

S:

: Server only. This frame MUST NOT be sent by the client.

### Padding frame

This frame has no semantic value. It can be used to mitigate traffic
analysis on the TLS records of a TCPLS session. The Padding frame has no
content.

~~~
Padding frame {
    Type (8) = 0x00,
}
~~~
{: #padding-frame-format title="Padding frame format"}

### Ping frame

This frame is used to elicit an acknowledgment from its peer. It has no
content. When an endpoint receives a Ping frame, it acknowledges the TLS
record that contains this frame. This frame can be used to check that the
peer can receive TLS records over a particular TCP connection.

~~~
Ping frame {
    Type (8) = 0x01,
}
~~~
{: #ping-frame-format title="Ping frame format"}

### Stream frame

This frame is used to carry chunks of data of a stream.

~~~
Stream frame {
    Type (7) = 0x01,
    FIN (1),
    Stream ID (32),
    Offset (64),
    Length (16),
    Stream Data (...),
}
~~~
{: #stream-frame-format title="Stream frame format"}

FIN:

: The last bit of the frame type bit indicates that this Stream frame
ends the stream when its value is 1. The end of the stream is at the sum of the
Offset and Length fields of this frame.

Stream ID:

: A 32-bit unsigned integer indicating the ID of the stream this frame
relates to.

Offset:

: A 64-bit unsigned integer indicating the offset in bytes of the carried
data in the stream.

Length:

: A 16-bit unsigned integer indicating the length of the Stream Data field.

### ACK frame

This frame is sent by the receiver to acknowledge the receipt of TLS records on
a particular TCP connection of the TCPLS session. Although the reliability
of the data exchange on a connection is handled by TCP,

~~~
ACK frame {
    Type (8) = 0x04,
    Connection ID (32),
    Highest Sequence Received (64),
}
~~~
{: #ack-frame-format title="ACK frame format"}

Connection ID:

: A 32-bit unsigned integer indicating the TCP connection for which the
acknowledgment was sent.

Highest Sequence Received:

: A 64-bit unsigned integer indicating the highest TLS record sequence
number received on this TCP connection.

### New Token frame

This frame is used by the server to provide tokens to the client. Each token
can be used to join a new TCP connection to the TCPLS session, as described
in TODO backref. Clients MUST NOT send New Token frames.

~~~
New Token frame {
    Type (8) = 0x05,
    Sequence (8),
    Token (256),
}
~~~
{: #new-token-frame-format title="New Token frame format"}

Sequence:

: A 8-bit unsigned integer indicating the sequence number of this token

Token:

: A 32-byte opaque value that can be used as a token by the client.

### New Address frame

This frame is used by an endpoint to communicate a new address to its peer.

~~~
New Address frame {
    Type (8) = 0x06,
    Address ID (8),
    IP Version (8),
    Address (32..128),
}
~~~
{: #new-address-frame-format title="New Address frame format"}

Address ID:

: A 8-bit unsigned integer indicating the ID of the address.

IP Version:

: A 8-bit unsigned integer indicating the IP version of the address.

Address:

: The address value

### Connection Failed frame

This frame is used by the receiver to inform the sender that a TCP
connection has failed, for instance due to a timeout or the receipt of a
spurious RST. The frame has a per-connection sequence to distinguish new
signals from delayed ones. Each endpoint keeps track of the largest
sequence number received for each connection. When receiving a Connection
Failed frame that does not increase the largest sequence number of its
connection, the endpoint MUST ignore the frame.

~~~
Connection Failed frame {
    Type (8) = 0x07,
    Connection ID (32),
    Sequence (8),
}
~~~
{: #connection-failed-format title="Connection Failed format"}

Connection ID:

: A 32-bit unsigned integer indicating the ID of the connection that failed.

Sequence:

: A 8-bit unsigned integer encoding the sequence of Connection Failed frame
sent for the connection.

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.

TODO IANA actions for TLS Extensions
TODO IANA actions for TCPLS frames

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
