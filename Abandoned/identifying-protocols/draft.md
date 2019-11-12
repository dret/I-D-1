---
title: Identifying Protocols
docname: draft-nottingham-identifying-protocols-00
date: 2017
category: bcp

ipr: trust200902
area: General
workgroup:
keyword: Internet-Draft

stand_alone: yes
pi: [toc, tocindent, sortrefs, symrefs, strict, compact, comments, inline]

author:
 -
    ins: M. Nottingham
    name: Mark Nottingham
    organization:
    email: mnot@mnot.net
    uri: https://www.mnot.net/

normative:
  RFC2119:

informative:


--- abstract

This document explores different means of identifying protocols, the reasons for doing so, and
defines best practices for their use.


--- note_Note_to_Readers

The issues list for this draft can be found at <https://github.com/mnot/I-D/labels/identifying-protocols>.

The most recent (often, unpublished) draft is at <https://mnot.github.io/I-D/identifying-protocols/>.

Recent changes are listed at <https://github.com/mnot/I-D/commits/gh-pages/identifying-protocols>.


--- middle

# Introduction

For a variety of reasons, it's often necessary to identify Internet protocols. This includes:

- Assuring that the endpoints positively support the intended protocol (e.g., to avoid cross-protocol attacks)
- Negotiating the use of a protocol (or protocol version)
- Configuring protocol details, such as transport parameters, or optional features
- Discriminating traffic based upon protocol, for policy enforcement or quality of service
- Establishing a name space for extensions and other protocol artefacts

Internet standards define a number of ways to identify a protocol. Less is said about the
appropriate use of these identifiers for various purposes. This document defines best practices for
the use of several kinds of protocol identifiers.


## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT",
"RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in
{{RFC2119}}.


# "Magic"

It's very common for protocols to specify an initial set of octets that are to be sent by endpoints
on a connection, so-called "magic."

Magic is best used to assure that the protocol being used is unambiguous, to prevent cross-protocol
attacks as well as plain misconfiguration.

For example, HTTP/2 {{?RFC7540}} uses a "connection preface" (Section 3.5) for this purpose; it
sends the string:

> PRI * HTTP/2.0\r\n\r\nSM\r\n\r\n

This was carefully designed to quickly and unambiguously break as many HTTP/1.1 servers as
possible, so that if a HTTP/2 client were to accidentally connect to a HTTP/1.1 server, it would
fail quickly and as reliably as possible. This aids error recovery.

Magic is less helpful for negotiating the protocol to be used. For example, the PROXY protocol
uses magic to preface metadata onto a connection. However, the client needs to be
configured to use the PROXY protocol on an endpoint-by-endpoint basis, since negotiation will fail
otherwise.

Since Magic only applies to the data stream it is used within, it is also less than useful for
negotiating alternative transports. However, a protocol can overload magic to perform some kinds of
configuration. This is effectively what the HTTP/2 SETTINGS frame does.





# Port Numbers

Transport protocols like TCP {{?RFC0793}} and UDP {{?RFC0768}} use port numbers to identify
protocols. Historically, port numbers have been used to identify the application in use, which maps
roughly to the protocol.

Generally, a protocol registers a "default" port that is to be used for it; this allows a client to
connect to a server for a given purpose only using that server's name.

However, there are cases where different applications use the same protocol on different ports
(e.g, SMTP on port 25, submission on port 587); conversely, different applications sometimes use
the same protocol on the same port (e.g., countless applications using HTTP on ports 80 and 443).

The binding between a port number and a protocol is also weak; just because HTTPS is allocated port
443, it isn't constrained to operate only on that port (and is commonly deployed on other ports as
well). Likewise, ports are sometimes used for purposes other than that allocated; again using 443
as an example, other protocols are known to use it for firewall traversal.

This means that while they serve their primary purpose of multiplexing multiple protocols onto the
same host well, port numbers have limited utility for the purposes of protocol identification
outlined above.

The most commonly cited case, discriminating traffic in the network, can only be described as "best
effort" generously, especially in these times of pervasive encryption {{?RFC7258}}. The port number
that traffic uses is no guarantee of the application or protocol in use. At best, it is an
indicator of *possible* application and/or protocol.


## Service Names

DNS SRV {{?RFC2782}} introduced Service Names as as complement to and extension of port numbers.
They are managed in the same registry {{?RFC0635}}:

    Service names are the unique key in the Service Name and Transport
    Protocol Port Number registry.  This unique symbolic name for a
    service may also be used for other purposes, such as in DNS SRV
    records.

That specification goes on to recommend:

> Because the port number space is finite (and therefore conservation is an important goal), the
> alternative of using service names instead of port numbers is RECOMMENDED whenever possible.

In other words, service names allow a host to dynamically nominate the port(s) to be used, rather
than relying upon a default port.

Being evolved from them, service names generally have the same granularity and scope of purpose as
port numbers; that is, they're fairly "high-level", occupying conceptual space somewhere between
coarsely-defined protocols and applications.

Service names have only been used with SRV to date. They are difficult to retrofit onto existing
protocols, since already-deployed clients will not support them, and therefore fail to work if they
are used.

So, stop asking for HTTP to support SRV. It's not going to happen.

Service names' conceptually "high" level and binding to SRV makes them useful for negotiating a protocol (or protocol suite), but less useful for negotiating a specific version or configuration of that protocol, since 


# URI Schemes

URI schemes {{?RFC3986}} occur as part of a resource identifier that's often (but not always) end
user-visible.

The primary purpose of URIs is identification.


# ALPN Protocol IDs

A newer form of protocol identification was introduced by ALPN {{?RFC7301}}, which establishes a
name space of Protocol Identifiers.

The original purpose of ALPN Protocol IDs was to identify the "next" protocol inside a TLS
connection, so that HTTP could negotiate the use of HTTP/2 instead of HTTP/1. HTTP/2 quickly
adopted this name space for other purposes, including the identification of protocols in
Alternative Services {{?RFC7838}}.

# IANA Considerations

This document has no requirements for IANA.

# Security Considerations


--- back
