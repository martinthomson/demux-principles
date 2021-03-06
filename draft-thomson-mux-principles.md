---
title: Principles for Multiplexing of UDP-based Protocols
abbrev: UDP Multiplexing Principles
docname: draft-thomson-mux-principles-latest
category: info
updates: 7983

ipr: trust200902
area: IAB
keyword: Internet-Draft

stand_alone: yes
pi: [notoc]

author:
 -
    ins: M. Thomson
    name: Martin Thomson
    organization: Mozilla
    email: martin.thomson@gmail.com


--- abstract

The existence of protocols that rely on multiplexing of different protocols
could be used to justify the imposition of constraints on the operation of
other protocols.  The existence of a multiplexed protocol does not inherently
justify constraints on protocols that participate or might participate in the
protocol.


--- middle

# Introduction

The use of multiple protocols that share a 5-tuple is possible in unordered
transport like UDP.  Multiplexing in this fashion creates a meta-protocol that
is comprised of the set of protocols that are multiplexed.

A specific example of such a meta-protocol is documented in RFC 7983
{{?RT-MUX=RFC7983}}.  This protocol comprises up to 5 different protocols: STUN
{{?STUN=RFC5389}}, ZRTP {{?ZRTP=RFC6189}}, DTLS {{?DTLS=RFC6347}}, TURN
channels {{?TURN=RFC5766}}, and RTP {{?RTP=RFC3550}}.  The meta-protocol that
is composed from this set of protocols is commonly deployed for use in
real-time communications.  Of particular note is the STUN protocol {{?STUN}},
which is specifically designed to be multiplexed with another protocol.

The existence of a multiplexed meta-protocol can be used to justify constraints
on the definition of new protocols, or the addition of changes to protocols that
participate.  These constraints should be considered in the context of the
protocol in use.  A protocol design for use outside of a multiplexed context
should not be unduly constrained by the limitations of a chosen multiplexing
scheme.


# Multiplexing is Almost Always Possible

Any protocol that includes integrity checks can be multiplexed with any other
protocol.  A sufficiently strong checksum or cryptographic authentication tag
allows arriving datagrams to be rejected by any protocol that the datagram was
not intended for with high probability.

New protocols often require confidentiality and integrity and so use a solution
like TLS {{?TLS=RFC5246}}.  Other protocols benefit greatly from being robust
against errors in the relatively weak checksum {{?CHECKSUM=RFC1071}} provided by
internet protocols {{?CHECKSUM-FAIL=DOI.10.1145/347059.347561}}.  Thus, the
inclusion of strong integrity checks is beneficial for any internet protocol,
which makes the protocol highly likely to be classified by a recipient as
intended.

Note:

: This does not mean that protocols are distinguishable to intermediaries.
  Protocols that include use keyed integrity checks will not be identifiable to
  entities that do not have access to keys.

Relying on an integrity check also probabilistic, meaning that shorter integrity
checks increase the chance that a datagram could be incorrectly classified by a
recipient.  While the probability of collision is negligible for a protocol that
uses an integrity check with 128 bits or more of redundancy, a shorter integrity
check could be vulnerable to collisions and mis-classification.

As long as no more than one participating protocol lacks an integrity check of
sufficient strength, protocols can be demultiplexed with high reliability.
However, this form of demultiplexing can be grossly inefficient, as every
datagram needs to be validated once for each potential protocol that is in use.


# Multiplexing Considerations

Practical multiplexing schemes rely on simpler and more direct differences
between protocols.  For instance, RFC 7983 {{?RT-MUX}} separates protocols based
on the value of the first octet.  While this scheme is cheap and effective, it
has the drawback of using a limited space of potential values.  Of the 256
possible values for the first octet, RFC 7983 consumes 132 values.  Adding more
protocols to the set that can be multiplexed would reduce the available space
further.

The design in RFC 7983 was initially opportunistic in nature.  The original set
of protocols that were selected included only DTLS, SRTP, and STUN.  Since then,
compatibility with that scheme has constrained the design of other protocols so
that they fit within this scheme.  This is not an indefinitely sustainable
posture.

The need to multiplex can create unwanted constraints on the designs of
protocols, particularly as protocols evolve.  For a protocol that is used
exclusively or predominantly in a multiplexed meta-protocol, this does not
present a significant burden, but protocols that have uses in other contexts are
different.

For instance, DTLS 1.3 {{?DTLS13=I-D.ietf-tls-dtls13}} defines a record layout
that uses the first octet in a very different fashion to earlier versions; this
design is constrained by the need to remain compatible with the scheme in RFC
7983 {{?RT-MUX}}.

Multiplexing should not be a primary consideration in design of new protocols
that might be multiplexed.  Similary, the design of extensions or revisions to
protocols should not be constrained by the potential for multiplexing.
Multiplexing considerations might be paid greater attention depending on how
likely it is that the protocol will be used together with other protocols.

For example, a new version of RTP {{?RTP}} might consider setting the version
field to 3.  While this space in the multiplexing scheme in {{?RT-MUX}} is
currently unallocated, changing the RTP version greatly reduces the space
available for other protocols.  Because RTP is frequently used in a multiplexed
context, changing the RTP header is best avoided.


## Alternative Multiplexing Designs

Protocol multiplexing works most efficiently when the protocol in use for any
given packet can be identified using only the contents of the packet.  Stateful
multiplexing - where multiplexing rules change based on protocol state - is
possible, but is best avoided.

A protocol that intends to multiplex several protocols can avoid this sort of
pressure by adding an explicit multiplexing protocol layer.  The simplest
multiplexing protocol is a shim of a small number of octets that is added to the
start or end of every datagram.  The value of these octets is then used to
identify the protocol.

For a scheme like that in {{?RT-MUX}}, it is possible to use a shim for only
some protocols, whether other protocols are multiplexed based on their inherent
observable properties.

Expanding the size of datagrams can have implications for protocol operation.
Trivially, it reduces the space in the path maximum transfer unit (PMTU)
{{?PMTUv4=RFC1191}}, {{?PMTUv6=RFC8201}} available to the multiplexed
meta-protocol.

More substantially, the addition of octets to a protocol datagram - especially
at the start - can also mean that the apparent format of datagrams changes.  If
a multiplexed meta-protocol adds octets to the start of payloads, this reduces
the degree to which the multiplexed meta-protocol can be identified correctly.
Some middleboxes have been shown to observe and discriminate based on the
content of flows, even when large parts of a flow is encrypted and therefore
inaccessible to the middlebox.  Octets added to the start of a datagram could
cause middleboxes to apply different treatment to a protocol when it is
multiplexed than when it is not.


# No Expectation of Successful Multiplexing

A multiplexing scheme that relies on certain properties of a protocol cannot
expect those properties to remain fixed as that protocol changes.

It's possible that the designers of a protocol will explicitly guarantee that
certain properties won't change over time, such as the invariants defined in
{{?QUIC-INVARIANTS=I-D.thomson-quic-invariants}}, in which case a multiplexing
scheme could be designed based on those guarantees.

Choosing a multiplexing scheme that relies on properties of a protocol remaining
constant can either impose unwanted constraints on the design of a protocol that
is selected for multiplexing, or it can cause some or all features of that
protocol to be unusable in the multiplexed context when that protocol changes.


# Security Considerations

The ability to use new protocol features can have a material impact on security
if access to updated or added security-related features is prevented by an
incompatibility with a chosen multiplexing scheme.


# IANA Considerations

This document has no IANA actions.


--- back
