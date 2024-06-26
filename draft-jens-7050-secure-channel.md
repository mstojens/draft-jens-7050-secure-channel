---
title: "Redefining Secure Channel for ipv4only.arpa IPv6 Prefix Discovery"
abbrev: "7050-secure-channel"
category: std
updates: "7050"

docname: draft-jens-7050-secure-channel-latest
submissiontype: IETF
v: 3
keyword:
 - secure channel
 - encrypted DNS
 - DNR
 - ipv4only.arpa

author:
 -
    fullname: Tommy Jensen
    organization: Microsoft
    email: tojens@microsoft.com

normative:

informative:


--- abstract

This document updates Discovery of the IPv6 Prefix Used for IPv6 Address Synthesis specification (RFC 7050) to redefine the term "secure channel" and 
modify requirements for nodes and DNS64 servers to use more recent developments
in DNS security.


--- middle

# Introduction

{{!RFC7050}} defines a mechanism for discovery of the current network's IPv6
address synthesis prefix that uses a DNS query for the ipv4only.arpa SUDN.
It further provides normative requirements for nodes to use a "secure channel"
when issuing this query in {{Section 3.1 of !RFC7050}}. {{Section 2.2 of !RFC7050}} defines "Secure channel" as follows:

{:quote}
>  a communication channel a node has between itself and
   a DNS64 server protecting DNS protocol-related messages from
   interception and tampering.  The channel can be, for example, an
   IPsec-based virtual private network (VPN) tunnel or a link layer
   utilizing data encryption technologies.

Since {{!RFC7050}} was published, there have been a number of developments
in secure DNS channels, including DoT {{!RFC7858}}, DoH {{!RFC8484}}, and DoQ
{{!RFC9250}}. These are more appropriate ways to provide a
secure channel that a node can use to gain trust in the "ipv4only.arpa" query being
answered by the network's designated DNS64 {{?RFC6147}} server. This document 
updates {{!RFC7050}} to redefine "secure channel" and specify requirements for networks
and client nodes to determine whether the channel between the DNS64 server and the 
client node can be considered secure for trusting the discovered IPv6
translation prefix.



# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document defines the following term:

Name-Validating Encrypted DNS Protocols:
   Any standardized encrypted DNS protocol that allows a DNS client to use a
   DNS server's claimed hostname assignment to securely verify that
   the server is designated for use with the specified encryption protocol.
   Examples as of the time of this writing include DoT
   {{!RFC7858}}, DoH {{!RFC8484}}, and DoQ {{!RFC9250}}, which all use TLS,
   either directly, indirectly through HTTPS, or through QUIC's use of the TLS
   handshake as described in {{?RFC9001}}.

# Explanation of Changes

## Redefining Secure Channel

The RFC7050 definition of "secure channel" was not uniformly supported
by vendors. In comparison, DNS servers at the time of this writing are widely
adopting support for Name-Validating Encrypted DNS Protocols, as are OS vendors
who act as the validating nodes. This means there is no need to provide
additional encryption at the link layer to provide validation of the DNS64
resolver or protection of the DNS traffic, which only places extra burdens on
network operators and client nodes alike. Using link-layer protection instead
of Name-Validating Encrypted DNS Protocols also deprives the DNS client of the
assurance that the server that it is communicating with has a valid claim to the
server hostname that is specified by system administrators in the client's
encrypted DNS configuration.

## Deprecating use of DNSSEC for DNS64 server validation

The original text in RFC7050 defined a DNSSEC-based mechanism for validating the
DNS64 resolver with necessary complication to determine the name needed for
validation. When Name-Validating Encrypted DNS Protocols are used, there is a
pre-defined name for the desired resolver, which removes this complication.

This name SHOULD be dynamically discovered from the network which advertises the
DNS64 server by using DNR {{!RFC9463}} so that no preconfiguration of nodes is
required to do proper server authentication of the DNS64 resolver. DNS
nodes SHOULD support DNR {{!RFC9463}} to facilitate this discovery.

As of this writing, there is wide support for Name-Validating Encrypted DNS
Protocols, whereas there is little support for DNSSEC validation by DNS clients,
which typically leave this responsibility to the recursive resolver.

## Relationship to RFC8880

{{?RFC8880}} updates {{!RFC7050}} to specify that the ipv4only.arpa zone MUST
be an insecure delegation. This document deprecates use of DNSSEC for validating
the resolver providing the prefix. This document and {{?RFC8880}} are therefore
complimentary, not in conflict, despite the apparent overlap due to both
revising the use of DNSSEC by {{!RFC7050}}.

# Text changes to RFC7050

## Redefining Secure Channel term

This document makes the following changes to {{Section 2.2 of !RFC7050}}:

OLD TEXT:

===

   Secure Channel: a communication channel a node has between itself and
   a DNS64 server protecting DNS protocol-related messages from
   interception and tampering.  The channel can be, for example, an
   IPsec-based virtual private network (VPN) tunnel or a link layer
   utilizing data encryption technologies.

===

NEW TEXT:

===

   Name-Validating Encrypted DNS Protocols: Any standardized encrypted DNS protocol that allows a DNS client to use a
   DNS server's claimed hostname assignment to securely verify that
   the server is designated for use with the specified encryption protocol.
   Examples as of the time of this writing include DoT
   {{!RFC7858}}, DoH {{!RFC8484}}, and DoQ {{!RFC9250}}, which all use TLS,
   either directly, indirectly through HTTPS, or through QUIC's use of the TLS
   handshake as described in {{?RFC9001}}.

   Secure Channel: A communication channel that a node has between itself and
   a DNS64 server that protects DNS protocol-related messages from
   interception and tampering. The channel uses a Name-Validating Encrypted DNS
   Protocol that is validated against a domain name associated with the configuration
   of the DNS64 resolver. The information that the client requires for DNS64-resolver
   server authentication may either be pre-configured or discovered using a mechanism 
   such as DNR {{?RFC9463}}.

===

## Deprecating use of DNSSEC for prefix discovery validation

### Redefining validation requirement

This document makes the following changes to {{Section 3.1 of !RFC7050}}:

OLD TEXT:

===

   To mitigate against attacks, the node SHOULD communicate with a
   trusted DNS64 server over a secure channel or use DNSSEC.  NAT64
   operators SHOULD provide facilities for validating discovery of
   Pref64::/n via a secure channel and/or DNSSEC protection.

   It is important to understand that DNSSEC only validates that the
   discovered Pref64::/n is the one that belongs to a domain used by
   NAT64 FQDN.  Importantly, the DNSSEC validation does not tell if the
   node is at the network where the Pref64::/n is intended to be used.
   Furthermore, DNSSEC validation cannot be utilized in the case of a
   WKP.

===

NEW TEXT:

===

   To mitigate against attacks, the node SHOULD communicate with a
   trusted DNS64 server over a secure channel. NAT64 operators SHOULD provide
   facilities for validating discovery of Pref64::/n via a secure channel,
   including support for one or more Name-Validating Encrypted DNS Protocols and
   the use of DNR {{!RFC9463}} to advertise the configuration that nodes need in
   order to connect to, and perform server authentication for, the DNS64 resolver.
   
   The node MUST use the name-validating functionality of its chosen Name-Validating
   Encrypted DNS Protocol and abort any connections to DNS64 resolvers that
   cannot pass this name validation. For example, clients using encrypted DNS protocols
   that use the TLS handshake will need to verify that the domain name of the DNS64
   server is present in the server's certificate and that the certificate is
   signed by a trust anchor that the node trusts.

===

### For the network

This document makes the following changes to {{Section 3.1.1 of !RFC7050}}:

OLD TEXT:

===

   If the operator has chosen to support nodes performing validation of
   discovered Pref64::/n with DNSSEC, the operator of the NAT64 device
   MUST perform the following configurations.

   1.  Have one or more fully qualified domain names for the NAT64
       translator entities (later referred to as NAT64 FQDN).  In the
       case of more than one Pref64::/n being used in a network, e.g.,
       for load-balancing purposes, it is for network administrators to
       decide whether a single NAT64's fully qualified domain name maps
       to more than one Pref64::/n, or whether there will be a dedicated
       NAT64 FQDN per Pref64::/n.

   2.  Each NAT64 FQDN MUST have one or more DNS AAAA resource records
       containing Pref64::WKA (Pref64::/n combined with WKA).

   3.  Each Pref64::WKA MUST have a PTR resource record that points to
       the corresponding NAT64 FQDN.

   4.  Sign the NAT64 FQDNs' AAAA and A resource records with DNSSEC.

===

NEW TEXT:

===

   The original version of RFC7050 defined a mechanism for securing the channel
   between a node and a DNS64 resolver by using DNSSEC. This is a complicated
   procedure that is discouraged in favor of the use of Name-Validating
   Encrypted DNS Protocols. Networks SHOULD plan to migrate nodes relying on this
   mechanism to using Name-Validating Encrypted DNS Protocols instead.

===

### For the node

This document makes the following changes to {{Section 3.1.2 of !RFC7050}}:

OLD TEXT:

===

   A node SHOULD prefer a secure channel to talk to a DNS64 server
   whenever possible.  In addition, a node that implements a DNSSEC
   validating resolver MAY use the following procedure to validate
   discovery of the Pref64::/n.

   1.  Heuristically find Pref64::/n candidates by making a AAAA
       resource record query for "ipv4only.arpa." by following the
       procedure in Section 3.  This will result in IPv6 addresses
       consisting of Pref64::/n combined with WKA, i.e., Pref64::WKA.
       For each Pref64::/n that the node wishes to validate, the node
       performs the following steps.

   2.  Send a DNS PTR resource record query for the IPv6 address of the
       translator (for ".ip6.arpa." tree), using the Pref64::WKA learned
       in step 1.  CNAME and DNAME results should be followed according
       to the rules in RFC 1034 [RFC1034], RFC 1035 [RFC1035], and RFC
       6672 [RFC6672].  The ultimate response will include one or more
       NAT64 FQDNs.

   3.  The node SHOULD compare the domains of learned NAT64 FQDNs to a
       list of the node's trusted domains and choose a NAT64 FQDN that
       matches.  The means for a node to learn the trusted domains is





Savolainen, et al.           Standards Track                    [Page 7]

RFC 7050                  Pref64::/n Discovery             November 2013


       implementation specific.  If the node has no trust for the
       domain, the discovery procedure is not secure, and the remaining
       steps described below MUST NOT be performed.

   4.  Send a DNS AAAA resource record query for the NAT64 FQDN.

   5.  Verify the DNS AAAA resource record contains Pref64::WKA
       addresses received at step 1.  It is possible that the NAT64 FQDN
       has multiple AAAA records, in which case the node MUST check if
       any of the addresses match the ones obtained in step 1.  The node
       MUST ignore other responses and not use them for local IPv6
       address synthesis.

   6.  Perform DNSSEC validation of the DNS AAAA response.

   After the node has successfully performed the above five steps, the
   node can consider Pref64::/n validated.

===

NEW TEXT:

===

   The original version of RFC7050 defined a mechanism for securing the channel
   between a node and a DNS64 resolver by using DNSSEC. This is a complicated
   procedure that is discouraged in favor of the use of Name-Validating
   Encrypted DNS Protocols. Nodes SHOULD NOT add support for the DNSSEC-based mechanism and
   SHOULD plan to replace this functionality with validation using Name-Validating
   Encrypted DNS Protocols.

===




# Security Considerations

This document modifies the mechanisms that nodes use to determine their connection
to the network's DNS64 resolver is secure. It shifts the verification from
IPsec to TLS or another mechanism that allows the client to verify the server's 
association with a domain name (via the use of Name-Validating Encrypted DNS
Protocols). This is expected to broaden the use of verification in the wild
given the greater likelihood, only increasing with time, that a DNS client will
implement support for a Name-Validating Encrypted DNS Protocol given this alternative
to the comparatively difficult task of implementing IPsec for its connection to
the DNS64 resolver.

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
