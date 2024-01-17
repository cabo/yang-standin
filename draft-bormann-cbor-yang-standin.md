---
v: 3

docname: draft-bormann-cbor-yang-standin-latest
title: "Stand-in Tags for YANG-CBOR"
date:

submissiontype: IETF
category: std
consensus: true

area: "Applications and Real-Time"
workgroup: "Concise Binary Object Representation Maintenance and Extensions"
keyword:
 - efficient YANG
venue:
  group: "CBOR (Concise Binary Object Representation Maintenance and Extensions)"
  type: "Working Group"
  mail: "cbor@ietf.org"
  github: "cabo/yang-standin"

author:
  - name: Carsten Bormann
    org: Universität Bremen TZI
    street: Postfach 330440
    city: Bremen
    code: D-28359
    country: Germany
    phone: +49-421-218-63921
    email: cabo@tzi.org
  - name: Maria Matejka
    org: CZ.NIC
    street: Milesovska 1136/5
    city: Praha
    code: 13000
    country: Czechia
    email:
    - maria.matejka@nic.cz
    - mq@jmq.cz

normative:
  RFC9254: yang-cbor
  I-D.schoenw-netmod-rfc6991-bis: legacy-bis
  RFC5952:
  RFC8943: date
  STD94: cbor
  RFC6021: yang-types

informative:


--- abstract

YANG (RFC 7950) is a data modeling language used to model
configuration data, state data, parameters and results of Remote
Procedure Call (RPC) operations or actions, and notifications.

YANG-CBOR (RFC 9254) defines encoding rules for YANG in the Concise Binary
Object Representation (CBOR) (RFC 8949).
While the overall structure of YANG-CBOR is encoded in an efficient,
binary format, YANG itself has its roots in XML and therefore
traditionally encodes some information such as date/times and IP
addresses/prefixes in a verbose text form.

This document defines how to use existing CBOR tags for this kind of
information in YANG-CBOR as a "stand-in" for the text-based
information that would be found in the original form of YANG-CBOR.


--- middle

# Introduction

(see abstract)


# Conventions and Definitions

The terminology of {{-yang-cbor}} applies.

Legacy representation:
: The (often text-based) representation for a YANG data item as used
  in YANG-XML, YANG-JSON, and (unchanged) YANG-CBOR.

Stand-in tag:
: A CBOR tag that can supply the information that is equivalent to a
  legacy representation in a more efficient format (e.g., using binary
  data).

Round Trip:
: A pair of conversions from a legacy representation to a stand-in tag
  and back to a legacy representation.
  By definition of these conversions, a round trip needs to retain the
  semantic information of the original legacy representation.

Unambiguous Round Trip:
: A Round Trip that provides exactly the same legacy representation
  (not just semantically equivalent).
  The stand-in tag is also said to "unambiguously stand in" for the
  legacy representation.

{::boilerplate bcp14-tagged}

# Stand-In Tags

This document defines two sets of stand-in tags.
Where information starts out in a legacy representation, these tags
are only used when an Unambiguous Round Trip can be achieved.

## `ietf-yang-types`: Tag 1 (Date/Time) and Tag 100 (Date)

{{Section 3 of -legacy-bis}} defines the following types in `ietf-yang-types`:

YANG type | base type | specification | stand-in
date-and-time | string | {{-yang-types}} | tag 1
date-with-zone-offset | string | {{-legacy-bis}} | (none)
date-no-zone | string | {{-legacy-bis}} | tag 100
{: title="Legacy representations in ietf-yang-types"}

Tag 1 ({{Section 3.4.2 of RFC8949@-cbor}}) can unambiguously stand in for all `date-and-time` values that:

* do not specify a time zone (note that {{-legacy-bis}}
uses the legacy "`-00:00`" format for time-zone-free date-times)
* are not an inserted leap second (23:59:60 or 23:59:61)
* do not have trailing zeroes in the fractional part of the seconds.
* do not have fractional parts of the seconds with a precision that
  cannot be represented in floating-point tag content in a tag 1.

All other `date-and-time` values stay in legacy representation.

Tag 1 uses an integer tag content for all `date-and-time` values
without fractional seconds and a floating-point tag content for values
that have fractional seconds given.

Tag 100 {{-date}} can unambiguously stand in for all `date-no-zone` values.

## `ietf-inet-types`: Tags 54 and 52 (IP addresses and prefixes)

{{Section 4 of -legacy-bis}} defines in `ietf-inet-types`:

YANG type | base type | specification | stand-in
ip-address | union | {{-yang-types}} | (see union)
ipv4-address | string | {{-yang-types}} | tag 52
ipv6-address | string | {{-yang-types}} | tag 54
ip-address-no-zone | union | RFC 6991 | (see union)
ipv4-address-no-zone | ipv4-address | RFC 6991 | tag 52
ipv6-address-no-zone | ipv6-address | RFC 6991 | tag 54
ip-address-link-local | union | {{-legacy-bis}} | (see union)
ipv4-address-link-local | ipv4-address | {{-legacy-bis}} | tag 52
ipv6-address-link-local | ipv6-address | {{-legacy-bis}} | tag 54
ip-prefix | union | {{-yang-types}} | (see union)
ipv4-prefix | string | {{-yang-types}} | tag 52
ipv6-prefix | string | {{-yang-types}} | tag 54
ip-address-and-prefix | union | {{-legacy-bis}} | (see union)
ipv4-address-and-prefix | string | {{-legacy-bis}} | tag 52
ipv6-address-and-prefix | string | {{-legacy-bis}} | tag 54
{: title="Legacy representations in ietf-yang-types"}

TO DO: Define usage of tags 54 and 52 for the cases we actually
support.
Addresses that have zones given cannot use tag 54/52.
Addresses with leading zeros cannot use tag 54/52.
Addresses that do not comply with {{RFC5952}} cannot use tag 54.

TO DO: Check how the unions in {{-yang-types}} and {{-legacy-bis}} interact
with this.  E.g., the union ip-address needs to be parsed to decide
between tag 54 and tag 52.

# Using Stand-In Tags

## The Role of the Schema

One approach of introducing stand-in tags in a produced would be to
match any text string it produces against the format of the legacy
representations and put in a stand-in tag instead.
We call this the schema-agnostic approach.

It is probably more efficient to trigger producing a stand-in tag from
schema information available while producing the YANG-CBOR.
However, that entangles the use of stand-in tags with the need schema
information; the algorithm for deciding about the use of the stand-in
tag needs to have enough schema information available to make this
decision.

Not all values of a schema type can be represented by a tag such that
an unambiguous round trip is achieved.  This document aims at an
unambiguous round trip.
ISSUE: Can we maybe live with round trips that aren't unambiguous?

# Negotiation

Introducing stand-in tags in YANG-CBOR requires some form of consent
between the producer and the consumer of YANG-CBOR information:

* A producer that creates YANG-CBOR containing stand-in tags needs to
  know whether the consumer supports stand-in tags, and, possibly,
  which specific stand-in tags it supports.  We speak about the
  _capability_ of a consumer to consume stand-in tags.
  A producer MUST NOT employ stand-in tags unless it knows about the
  capabilities of the consumer.
  A consumer SHOULD indicate its capabilities for consuming stand-in tags.

* A consumer may not want to implement certain legacy text-based
  representations where more efficient (and easy to implement)
  stand-in tags are available.  This places a _requirement_ on the
  producer (which needs to have the _capability_ to produce YANG-CBOR
  where those stand-in tags are used, in place of legacy
  representations).
  A producer MUST NOT employ legacy representations where stand-in
  tags are _required_ by the consumer.
  A consumer that has requirements for only receiving stand-in tags in
  place of legacy representations, MUST indicate this to the producer.

ISSUE: Where do we put those two aspects of negotiation?

* NETCONF negotiation
* yang-library
* media-type parameters
* ?

# Security Considerations

TODO Security


# IANA Considerations

## stand-in tags?

ISSUE: Do we want to have a separate registry for stand-in tags?

They already are CBOR tags and thus in the in the registry, but might
get lost in the bulk of that (and are only identified as YANG-CBOR
stand-in Tags in the specification).

## media-type parameters

ISSUE: Should the use of stand-in tags be mentioned in the various
YANG-CBOR-based media types (as a media type parameter)?
Compare how application/yang-data+cbor can use id=name/id=sid to
indicate another encoding decision.

--- back

# Acknowledgments
{:unnumbered}

TODO acknowledge.
