---

title: "Indicating Non-Availability of Dynamic Updates in the DNS"
abbrev: "Non-Availability of Dynamic Updates"
category: bcp

docname: draft-jabley-dnsop-missing-mname-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "Domain Name System Operations"
keyword:
 - dns
 - mname
venue:
  group: "Domain Name System Operations"
  type: "Working Group"
  mail: "dnsop@ietf.org"
  github: "ableyjoe/draft-jabley-dnsop-missing-mname"

author:
 -
    fullname: Joe Abley
    city: Amsterdam
    country: Netherlands
    organization: Cloudflare
    email: jabley@cloudflare.com
 -
    fullname: Peter Thomassen
    city: Berlin
    country: Germany
    organization: deSEC, Secure Systems Engineering
    email: peter@desec.io

normative:

informative:

--- abstract

The Start of Authority (SOA) Resource Record (RR) in the Domain
Name System (DNS) specifies various parameters related to the
handling of data in DNS zones.  These parameters are variously used
by authority- only servers, caching resolvers and DNS clients to
guide them in the way that data contained within particular zones
should be used.

One particular field in the SOA RR is known as MNAME, which is used
to specify the "Primary Master" server for a zone.  This is the
server to which Dynamic Updates are sent by clients.  Many zones
do not accept updates using the Dynamic Update mechanism, and any
such DNS UPDATE messages which are received provide no usual purpose.
For such zones it may be preferable not to receive updates from
clients at all.

This document proposes a convention by which a zone operator can
signal to clients that a particular zone does not accept Dynamic
Updates.

--- middle

# Introduction

{{!RFC2136}} specifies a mechanism for clients to update zones in
the DNS dynamically.  This mechanism is widely-deployed by many
end-station operating systems, where it is used (for example) to
update DNS records in response to a local change of IP address.

Many zones, however, do not accept dynamic updates from clients as
a matter of policy.  For such zones, specifying a DNS server name
in the MNAME field of an SOA record has no benefit, and in fact may
well cause unwanted traffic (DNS UPDATE messages) to be received
by the named server.

This document proposes a convention by which a zone operator can
signal to clients that a particular zone does not accept Dynamic
Updates.

# Terminology

{::boilerplate bcp14-tagged}

This document assumes familiarity with the terminology of the Domain
Name System as described in {{?RFC9499}}.

# Use of the MNAME Field

The Start of Authority (SOA) Resource Record (RR) is defined in
{{!RFC1035}}.  The MNAME field of the SOA RDATA is defined in that
document as "The &lt;domain-name&gt; of the name server that was
the original or primary source of data for this zone."

{{!RFC1035}} includes no specific guidance on the use of the MNAME
field, although the general tone in which SOA RDATA are discussed
suggests that its intended purpose was for the management of zone
transfers between authority-only servers.  There are no implementations
of authority-only servers known to the author which use the MNAME
field to manage or perform zone transfers, however; for bootstrapping
reasons, commonly-deployed implementations require master servers
to be specified explicitly, usually by address rather than name.

The MNAME field was subsequently referred to in {{!RFC1996}}, as
part of the definition of the term "Primary Master".  The server
specified in the MNAME field was, by default, to be excluded from
the set of servers to which DNS NOTIFY messages would be sent.

In {{!RFC2136}} the MNAME field was again used to provide a definition
for the term "Primary Master", in this case for the purpose of
identifying the server towards which dynamic updates for that zone
should be sent.

There have been no other references to the use of the MNAME in the
RFC series.

This document specifies a convention by which a zone operator may
include an empty MNAME field in order to deliberately specify that
there is no appropriate place for Dynamic Updates to be sent.

# Operations

Zone administrators who do not wish to receive Dynamic Updates from
clients for a particular zone may specify an empty MNAME field in
that zone's SOA RDATA.  The textual representation of an empty field
in the canonical representation of zone data is a single ".", as
illustrated below.

~~~ ascii-art {#soa}
   @       1800    IN      SOA     jabley.automagic.org. . (
                                        20080622    ; serial
                                        1800        ; refresh
                                        900         ; retry
                                        10800       ; expire
                                        1800 )      ; negative cache TTL
~~~

Dynamic Update clients who identify the Primary Master server as
the recipient of DNS UPDATE messages from the MNAME field in SOA
RDATA SHOULD interpret an empty MNAME field as an indication that
no attempt to send a DNS UPDATE message should be made for the zone
containing the SOA record.

# Impact on DNS NOTIFY

{{!RFC1996}} specifies that the Primary Server, which is derived
from the MNAME field of the SOA RDATA, be excluded from the set of
servers to which NOTIFY messages should be sent.

For zones whose SOA record contains an MNAME field which corresponds
to a server listed in the apex NS set, making the MNAME field empty
might well cause additional NOTIFY traffic.  If this is a concern,
the operators of the authority-only servers for the zone might
choose to specify an explicit notify list.

# Impact on DNS UPDATE

The goal of the convention specified in this document is to prevent
Dynamic Update clients from sending DNS UPDATE messages for particular
zones.  The use of an empty MNAME field is intended to prevent a
Dynamic Update client from finding a server to send DNS UPDATE
messages to.

# Unintended Consequences

Some concern has been raised in the past that an empty MNAME field
might result in unwanted traffic being sent to root servers, e.g.
for clients that interpret the MNAME as a zone name rather than a
hostname and direct traffic towards the root zone's nameservers.
However, no examples of this behaviour have been identified.

Use of an empty MNAME field is not new; cursory analysis of passive
DNS data demonstrates a robust volume of DNS responses with QTYPE=SOA
and empty SOA.MNAME for zones across a variety of top-level domains.
See {{quantify}} for discussion.

# Security Considerations

The convention described in this document provides no additional
security risks to DNS zone or server administrators.

Name servers which do not support Dynamic Updates for the zones
they host might experience a security benefit from reduced DNS
UPDATE traffic, the absence of that traffic provides additional
headroom in network bandwidth and server capacity for legitimate
query types.

Clients that normally send DNS UPDATE messages might see a security
benefit from not leaking the information contained within those
messages to nameservers that are not configured to receive them.

# IANA Considerations

This document makes no requests of the IANA.

--- back

# Observed Use of Empty MNAME fields in SOA Responses {#quantify}

A quick check using a variety of passive DNS datasets relating to
observed traffic on 2024-10-30 reveals asome volume of examples in
the real world. This data is not intended to say more than that,
but it perhaps suggests that a study with normalisation and a longer
time base might be useful to include in a future revision of this
draft.

|source  |counter |notes                   | {#realworld}
|----    |----    |----                    |
|com     |109328  |                        |
|net     |8854    |                        |
|org     |1792    |                        |
|czds    |964     |                        |
|imp     |634     | old gTLDs e.g. aero    |
|opencc  |111     | see openintel website  |


# Acknowledgments
{:numbered="false"}

Raffaele Sommese helped quantify existing observed use of SOA
responses with empty MNAME fields in a variety of passive DNS
datasets, as summarised briefly in {{quantify}}.
