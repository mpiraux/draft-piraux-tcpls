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
unit of TCPLS. A TLS record can be composed of several frames, which cannot
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
and QUIC to provide three modern transport services to applications in a
secure manner.

## Multiplexing

## Connection Migration

## Multipath

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
    opaque token<1..32>;
} Join;
~~~~

The "tcpls_join" extension is used by the client to join the TCP connection
on which it is sent to a TCPLS session. The extension contains a Token
provided by the server. The client MUST NOT send more than one
tcpls_join extension in its ClientHello. When receiving a ClientHello with
this extension, the server checks that the token is valid and joins the TCP
connection to the corresponding TCPLS session. When the token is not valid,
the server MUST abort the handshake with an illegal_parameter alert.

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
| 0x05       | New Token         |       | {{new-token-frame}}         |
| 0x06       | New Address       |       | {{new-address-frame}}       |
| 0x07       | Connection Failed |       | {{connection-failed-frame}} |
{: #tcpls-frame-types title="TCPLS frames"}

The "Rules" column in {{tcpls-frame-types}} indicates special requirements
regarding certain frames.

A:

: Non-ack-eliciting. Receiving this frame does not elicit the immediate sending
of an acknowledgment.

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

: The last bit of the frame type bit indicates whether this Stream frame
ends the stream. The end of the stream is at the sum of the Offset and
Length fields of this frame.

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
