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
  RFC4960:
  RFC6335:
  RFC7258:
  RFC8446:
  RFC8548:
  RFC8684:
  RFC9000:

informative:


--- abstract

This document specifies a protocol leveraging TCP and TLS to provide modern
transport services such as multiplexing, connection migration and multipath
in a secure manner.

--- middle

# Introduction

The TCP/IP protocol stack continuously evolves. In the early days, most
applications were interacting with the transport layer (mainly TCP, but
also UDP) using the socket API. This is illustrated in {{fig-arch-old}}.

~~~~~~~~~~~~~~~~~~~~~~~~~~~
+------------------------------+
|          Application         |
+------------------------------+
|            TCP/UDP           |
+------------------------------+
|             IPv4             |
+------------------------------+
~~~~~~~~~~~~~~~~~~~~~~~~~~~
{: #fig-arch-old title="The classical TCP/IP protocol stack"}

The TCP/IP stack has slowly evolved and the figure above does not
anymore describe current Internet applications. IPv6 is now widely
deployed next to IPv4 in the network layer. In the transport layer,
protocols such as SCTP {{RFC4960}} or DCCP {{RFC6335}} and TCP
extensions including Multipath TCP {{RFC8684}} or tcpcrypt {{RFC8548}}
have been specified. The security aspects of the TCP/IP protocol suite
are much more important today than in the past {{RFC7258}}.  Many
applications rely on TLS {{RFC8446}} and their stack is
similar to the one shown in {{fig-arch-today}}.

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
{: #fig-arch-today title="Today's TCP/IP protocol stack"}

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
{: #fig-arch-quic title="QUIC protocol stack"}

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
{: #fig-arch-tcpls title="TCPLS in the TCP/IP protocol stack"}

In this document, we will use the term TLS/TCP to refer to the TLS 1.3
protocol running over a TCP connection. We reserve the word TCPLS for
the protocol that is proposed in this document.

This document is organised as follows.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Modern Transport Services

## Multiplexing

## Connection Migration

## Multipath

# TCPLS Protocol

## TCPLS Handshake

## TCPLS Frames

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
