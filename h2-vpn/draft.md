---
title: Virtual Private Networking over HTTP/2
abbrev: H2-VPN
docname: draft-nottingham-h2-vpn-00
date: 2015
category: info

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
    uri: http://www.mnot.net/

normative:
  RFC0791:
  RFC2460:
  RFC2119:
  RFC7540:

informative:


--- abstract

This document describes a set of extensions that allow an IP-based Virtual Private Network to be created over a HTTP/2 connection.

--- middle

# Introduction

Virtual Private Networks have become a common means of assuring that traffic is carried over a
protected network. However, current VPN technologies can be discriminated from non-VPN traffic,
leading to them being blocked in some networks.

As HTTP/2 is deployed more, it becomes an attractive substrate for carrying a VPN, due to both its
broad use, its multiplexed, low-overhead framing layer, its typical use of encryption, and the
large number of HTTP endpoints on the Internet.


## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT",
"RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in
{{RFC2119}}.


# Extensions for VPN over HTTP/2

## The SETTINGS_VPN HTTP/2 Setting

The SETTINGS_VPN HTTP/2 setting (0xTBD) indicates that the sender is willing to use create VPNs over the HTTP/2 connection that it occurs within. 


## The IP HTTP/2 Frame Type

IP frames (type=0xTBD) convey individual IP packets in the Virtual Private Network associated with the stream ID.

IP frames MAY also contain padding. Padding can be added to IP frames to obscure the size of messages. Padding is a security feature; see {{RFC7540}}, Section 10.7.

~~~
 +---------------+
 |Pad Length? (8)|
 +---------------+-----------------------------------------------+
 |                           IP Data (*)                       ...
 +---------------------------------------------------------------+
 |                           Padding (*)                       ...
 +---------------------------------------------------------------+
~~~

The IP frame contains the following fields:

* Pad Length: An 8-bit field containing the length of the frame padding in units of octets. This field is conditional (as signified by a "?" in the diagram) and is only present if the PADDED flag is set.
* IP Data: Transport data. The amount of data is the remainder of the frame payload after subtracting the length of the other fields that are present.
* Padding: Padding octets that contain no application semantic value. Padding octets MUST be set to zero when sending. A receiver is not obligated to verify padding but MAY treat non-zero padding as a connection error (Section 5.4.1) of type PROTOCOL_ERROR.

The IP frame defines the following flags:

* END_STREAM (0x1): When set, bit 0 indicates that this frame is the last that the endpoint will send for the identified stream. Setting this flag causes the stream to enter one of the "half-closed" states or the "closed" state ({{RFC7540}}, Section 5.1).
* V6 (0x2): When set, bit 1 indicates that this frame is an IPv6 {{RFC2460}} frame; otherwise, it is an IPv4 {{RFC0791}} frame.
* PADDED (0x4): When set, bit 3 indicates that the Pad Length field and any padding that it describes are present.

IP frames MUST be associated with a stream. If an IP frame is received whose stream identifier
field is 0x0, the recipient MUST respond with a connection error (Section 5.4.1) of type
PROTOCOL_ERROR.

IP frames are subject to flow control and can only be sent when a stream is in the "open" or
"half-closed (remote)" state. The entire IP frame payload is included in flow control, including
the Pad Length and Padding fields if present. If an IP frame is received whose stream is not in
"open" or "half-closed (local)" state, the recipient MUST respond with a stream error ({{RFC7540}}
Section 5.4.2) of type STREAM_CLOSED.

The total number of padding octets is determined by the value of the Pad Length field. If the
length of the padding is the length of the frame payload or greater, the recipient MUST treat this
as a connection error ({{RFC7540}} Section 5.4.1) of type PROTOCOL_ERROR.

Note: An IP frame can be increased in size by one octet by including a Pad Length field with a
value of zero.


# VPN over HTTP/2 Operation

A sender MUST receive a SETTINGS_VPN setting from its peer before sending any IP frames. 


# IANA Considerations

TBD

# Security Considerations

TBD

--- back


# TODO

* Fill out security considerations
* Fill out IANA considerations
* Authentication
* Fully describe effects upon / interaction with state model
* More fully describe use cases
  * Multiplexing multiple VPNs
  * Onion routing as a complementary extension
* Describe general best practices for running a VPN over TCP
  * Hopefully by referencing other RFCs rather than reinventing
  * It'll get better with UDP-based HTTP