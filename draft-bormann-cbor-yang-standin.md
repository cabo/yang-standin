---
v: 3

docname: draft-bormann-cbor-yang-standin-latest
title: "Stand-in Tags for YANG-CBOR"
date:

stream: IETF
category: std
consensus: true

area: "Applications and Real-Time"
workgroup: "CBOR (Concise Binary Object Representation) Maint. and Ext."
keyword:
 - efficient YANG
venue:
  group: "CBOR (Concise Binary Object Representation) Maintenance and Extensions"
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
  RFC9164: cbor-ip
  I-D.ietf-netmod-rfc6991-bis: legacy-bis
  RFC5952:
  RFC8943: date
  I-D.ietf-cbor-time-tag: extended-time
  STD94: cbor
  RFC6021: yang-types
  RFC9562: uuid
  IANA.cbor-tags:

informative:
  RFC9557: ixdtf
  RFC9542: mac-address
  I-D.bormann-cbor-notable-tags: notable

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

Encoder:
: The party which generates (sends) CBOR data described by YANG.

Intermediate Encoder:
: An encoder which isn't the original author of the data, converting it
  from legacy representation.

Aggressive Intermediate Encoder:
: An intermediate encoder that might choose to discard some
  information of a legacy representation in order to be able to use a
  stand-in tag.
  Such a choice may be based on knowledge of the Decoder's handling of
  such information (e.g, to accommodate intolerant decoders), or it
  may be a general characteristic of the service provided by the
  intermediate encoder (e.g., in order to serve as a legacy-eschewing
  encoder).

Legacy-Eschewing Encoder:
: An encoder that does not generate legacy representations in places
  where a stand-in tag might instead be used.
  An intermediate encoder may need to be aggressive to achieve this.

Decoder:
: The party which receives and parses CBOR data described by YANG.

Intolerant Decoder:
: A decoder that does not accept legacy representations in places
  where a stand-in tag might instead be used.
  Such a decoder is designed to interoperate only with an
  legacy-eschewing encoder.

Intermediate Decoder:
: A decoder which isn't the final recipient of the data, converting it
  to legacy representation.

Data Transfer:
: A series of actions, generally beginning by data origination, encoding,
  continuing by optional intermediate transcoding, sending and receiving,
  and finally decoding and consuming.

Round Trip:
: Part of a data transfer between an encoder generating CBOR data with
  stand-in tags and a decoder parsing the data.

Legacy Round Trip:
: A Round Trip where the encoder is an intermediate encoder or the decoder is
  an intermediate decoder and any of these converts from or to the
  legacy representation.

Unambiguous Round Trip:
: A Legacy Round Trip that provides exactly the same legacy representation
  (not just semantically equivalent).
  The stand-in tag is also said to "unambiguously stand in" for the
  legacy representation.

{::boilerplate bcp14-tagged-bcp14}

# Stand-In Tags

This document defines two sets of stand-in tags.
Where information starts out in a legacy representation, these tags
are only used when an Unambiguous Round Trip can be achieved.

## `ietf-yang-types`: Tag 1 (Date/Time) and Tag 100 (Date)

{{Section 3 of -legacy-bis}} defines the following types in `ietf-yang-types`:

YANG type | base type | specification | stand-in
date-and-time | string | {{-yang-types}} | tag 1
date-no-zone | string | {{-legacy-bis}} | tag 100
{: title="Legacy date and date/time representations in ietf-yang-types"}

Tag 1 ({{Section 3.4.2 of RFC8949@-cbor}}) can unambiguously stand in for all `date-and-time` values that:

* do not specify a time zone (note that {{-legacy-bis}}
uses the legacy "`-00:00`" format for time-zone-free date-times)
* are not an inserted leap second (23:59:60 or 23:59:61)
* do not have trailing zeroes in the fractional part of the seconds.
* do not have fractional parts of the seconds with a precision that
  cannot be represented in floating-point tag content in a tag 1.

Tag 1 uses an integer tag content for all `date-and-time` values
without fractional seconds and a floating-point tag content for values
that have fractional seconds given.

Tag 100 {{-date}} can unambiguously stand in for all `date-no-zone` values.

## `ietf-yang-types`: Tag 1001 (Extended Date and Time)

{{Section 3 of -legacy-bis}} defines the following types in `ietf-yang-types`:

YANG type | base type | specification | stand-in
date-and-time | string | {{-yang-types}} | tag 1001
date-no-zone | string | {{-legacy-bis}} | tag 1001
date | string | {{-legacy-bis}} | tag 1001
time | string | {{-legacy-bis}} | tag 1001
time-no-zone | string | {{-legacy-bis}} | tag 1001
timeticks | uint32 | {{-yang-types}} | tag 1002
timestamp | uint32 | {{-yang-types}} | tag 1001
hours32        | int32 | {{-legacy-bis}} | tag 1002
minutes32      | int32 | {{-legacy-bis}} | tag 1002
seconds32      | int32 | {{-legacy-bis}} | tag 1002
centiseconds32 | int32 | {{-legacy-bis}} | tag 1002
milliseconds32 | int32 | {{-legacy-bis}} | tag 1002
microseconds32 | int32 | {{-legacy-bis}} | tag 1002
microseconds64 | int64 | {{-legacy-bis}} | tag 1002
nanoseconds32  | int32 | {{-legacy-bis}} | tag 1002
nanoseconds64  | int64 | {{-legacy-bis}} | tag 1002
{: title="Legacy representations in ietf-yang-types"}

Tag 1001 {{-extended-time}} can unambigously stand for all the aforementioned
types of values.

If the encoder supports tags 1001 and 1002, it MUST NOT use tags 100 and 1 as stand-ins.
Intolerant decoders should specify which kind of tags they expect.


## `ietf-yang-types`: Tags 37 (UUID) and CPA113 (hex-string) {#hex-tags}

{{Section 3 of -legacy-bis}} defines the following types in `ietf-yang-types`:

| YANG type    | base type | specification | stand-in   |
| uuid         | string    | {{-legacy-bis}} | tag 37     |
| hex-string   | string    | {{-legacy-bis}} | tag CPA113 |
| mac-address  | string    | {{-yang-types}} | tag CPA113 |
| phys-address | string    | {{-yang-types}} | tag CPA113 |
{: #tab-hex title="Legacy UUID and colon-separated hexadecimal representations in ietf-yang-types"}

These types are hexadecimal representations of byte strings, adorned
in various ways.

`uuid` stands for a 16-byte byte string ({{Section 4 of -uuid}}),
represented in hexadecimal with ASCII minus/hyphen characters added in
specific positions.
Tag 37 (see also Section 7 of {{-notable}}) can be used as a binary
stand-in for this adorned hexadecimal representation.
According to the description of `uuid` in {{Section 3 of -legacy-bis}},
"the canonical representation uses lowercase characters".
For consistency with this specification, an intermediate decoder of a
tag 37 stand-in MUST use lowercase characters in the uuid hex string
generated.

`hex-string`, and the similar, but more specific types `mac-address`
and `phys-address`, stand for byte strings in various lengths (exactly
6 bytes for `mac-address`, variable-length for the others),
represented in hexadecimal with ASCII colon characters added between
the representations of each of the bytes.
This specification defines tag number CPA113 {{iana-113}} to be an additional
"Expected Later Encoding" tag (similar to tag 23, see {{Section 3.4.5.2
of RFC8949@-cbor}}), except that the expected encoding of CPA113
includes colons and uses lowercase hex digits.

The following example implementation of the transformation in a
decoder shows the use of lowercase hex characters (`%02x` as opposed
to `%02X`) and the insertion of colon characters between the
hex-represented bytes:

~~~ ruby
def tag_cpa113_to_legacy(s)
  s.bytes.map{|x| "%02x" % x}.join(":")
end
~~~

Note: {{Section 2.4 of RFC9542}} defines tag number 48 for MAC
addresses.
This could be used in place of tag CPA113, but only for MAC addresses,
not for other byte strings of a similar form.
This specification therefore requests IANA to assign a new CBOR tag that can be
used as a stand-in for all instances of colon-separated text strings
of hexadecimally represented bytes, as shown in {{tab-hex}}.

Note Related tags have not been defined so far for tag 21 or 22
defined alongside tag 23, as YANG has a base type "binary" that is
encoded in base64 classic in YANG-XML and YANG-JSON, but already
encoded in a binary byte string in YANG-CBOR; use cases that might
actually use base type "string" for base64-encoded data in YANG have
not been considered.  However, tag 21 or 22 could be used as stand-in
tags if that is useful for some specific YANG model not considered
here.

[^cpa]

[^cpa]: RFC-Editor: This document uses the CPA (code point allocation)
      convention described in [I-D.bormann-cbor-draft-numbers].  For
      each usage of the term "CPA", please remove the prefix "CPA"
      from the indicated value and replace the residue with the value
      assigned by IANA; perform an analogous substitution for all other
      occurrences of the prefix "CPA" in the document.  Finally,
      please remove this note.

## `ietf-inet-types`: Tags 54 and 52 (IP addresses and prefixes)

{{Section 4 of -legacy-bis}} defines in `ietf-inet-types`:

YANG type | base type | specification | stand-in
ip-address | union | {{-yang-types}} | (see union)
ipv6-address | string | {{-yang-types}} | tag 54
ipv4-address | string | {{-yang-types}} | tag 52
ip-address-no-zone | union | RFC 6991 | (see union)
ipv6-address-no-zone | ipv6-address | RFC 6991 | tag 54
ipv4-address-no-zone | ipv4-address | RFC 6991 | tag 52
ip-address-link-local | union | {{-legacy-bis}} | (see union)
ipv6-address-link-local | ipv6-address | {{-legacy-bis}} | tag 54
ipv4-address-link-local | ipv4-address | {{-legacy-bis}} | tag 52
ip-prefix | union | {{-yang-types}} | (see union)
ipv6-prefix | string | {{-yang-types}} | tag 54
ipv4-prefix | string | {{-yang-types}} | tag 52
ip-address-and-prefix | union | {{-legacy-bis}} | (see union)
ipv6-address-and-prefix | string | {{-legacy-bis}} | tag 54
ipv4-address-and-prefix | string | {{-legacy-bis}} | tag 52
{: title="Legacy representations in ietf-yang-types"}

An intermediate encoder MAY normalize IPv6 addresses and prefixes that do not comply with {{RFC5952}}
but can be converted into the stand-in representation.
For example, IPv6 address written as 2001:db8:: is the same as 2001:0db8::0:0 and both would
be converted to `54(h'20010db8000000000000000000000000')`, anyway only the
first one complies with {{RFC5952}}. The encoder MAY refuse to convert the
latter one.

If the schema specifies
`ip-prefix`, an intermediate encoder MAY normalize prefixes with non-zero bits after the prefix end.
For example, if the legacy representation of `ipv6-prefix` is 2001:db8:1::/40, the encoder
may either refuse it as malformed or convert it to 2001:db8::/40 and represent
as `54([40, h'20010db8'])`.

The encoder implementation should be clear about which normalizations are employed and how.

Adapted examples from {{-cbor-ip}}:

Stand-in representation of IPv6 address 2001:db8:1234:deed:beef:cafe:face:feed
is `54(h'20010db81234deedbeefcafefacefeed')`.

CBOR encoding of stand-in (19 bytes):

~~~ cbor-pretty
D8 36                                  # tag(54)
   50                                  # bytes(16)
      20010DB81234DEEDBEEFCAFEFACEFEED
~~~

CBOR encoding of legacy representation (40 bytes):

~~~ cbor-pretty
78 26                                   # text(38)
   323030313A6462383A313233343A646565643A626565663A636166653A666163653A66656564
~~~

Stand-in representation of IPv6 prefix 2001:db8:1234::/48 is
`54([48, h'20010db81234'])`.

CBOR encoding of stand-in (12 bytes):

~~~ cbor-pretty
D8 36                 # tag(54)
   82                 # array(2)
      18 30           # unsigned(48)
      46              # bytes(6)
         20010DB81234 # " \u0001\r\xB8\u00124"
~~~

CBOR encoding of legacy representation (19 bytes):

~~~ cbor-pretty
72                                      # text(18)
   323030313A6462383A313233343A3A2F3438 # "2001:db8:1234::/48"
~~~

Stand-in representation of IPv6 link-local address fe80::0202:02ff:ffff:fe03:0303/64%eth0 is
`54([h'fe8000000000020202fffffffe030303', 64, 'eth0'])`.

CBOR encoding of stand-in (27 bytes):

~~~ cbor-pretty
D8 36                                   # tag(54)
   83                                   # array(3)
      50                                # bytes(16)
         FE8000000000020202FFFFFFFE030303
      18 40                             # unsigned(64)
      44                                # bytes(4)
         65746830                       # "eth0"
~~~

CBOR encoding of legacy representation (40 bytes):

~~~ cbor-pretty
78 26                                   # text(38)
   666538303A3A303230323A303266663A666666663A666530333A303330332F36342565746830
~~~

TO DO: adapt more examples from {{-cbor-ip}}

TO DO: Check how the unions in {{-yang-types}} and {{-legacy-bis}} interact
with this.  E.g., the union ip-address needs to be parsed to decide
between tag 54 and tag 52.

## Union handling

When the schema specifies a union data type for a node, there are
additional requirements on the encoder and decoder.

An encoder which is fully aware of data semantics MUST use the appropriate
data type, even though it isn't formally specified by the schema.

If an intermediate encoder doesn't fully understand the data semantics,
it needs to find out which type the data actually is to choose the right stand-in.
If more types are possible, it MAY choose any of these which allow for an Unambiguous Round Trip,
otherwise it SHOULD keep the legacy representation.

If a decoder receives data for a union-typed node, it MUST accept any data type
of the union, even though it may violate additional constraints outside the schema.

# Using Stand-In Tags

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
  stand-in tags are available, i.e., it may use an intolerant decoder.
  This places a _requirement_ on the
  producer to use a legacy-eschewing encoder (which therefore needs to
  have the _capability_ to produce YANG-CBOR
  where those stand-in tags are used, in place of legacy
  representations).
  Where the consumer employs an intolerant decoder, stand-in tags are
  _required_ by the consumer: for interoperating with a producer's
  encoder, this MUST be legacy-eschewing, i.e. it MUST NOT employ
  legacy representations.
  A consumer that has requirements for only receiving stand-in tags in
  place of legacy representations, MUST indicate this to the producer.

## Intermediate transcoding

The simplest situation is when no intermediate encoders and decoders are
involved in the data transfer, therefore the round trip is not legacy.
In this case, no conversions are involved.

Producing a stand-in MUST be always triggered by schema usage. Intermediate encoders
MUST NOT encode stand-ins when no schema is available. This avoids problems with
detecting whether e.g. the input string "15:32:06" is actually a time or just
accidentally looking like a time.

## Defining Stand-In Usage outside the data path

Requiring modifications to a YANG model in order to use it with
stand-in tags would pose significant deployment hurdles to using
stand-in tags.

A YANG model may want to restrict the information content in such a
way that stand-in tags can always be used, e.g., by using date-no-zone
in place of date where that is applicable, or by excluding features of
a YANG data type that cannot be represented in a stand-in-tag.

The encoder and/or the decoder MAY also just declare their stand-in usage in
their documentation. Using this negotiation method as the only one is generally
not recommended.

## In-message negotiation

The encoder may append or prepend a stand-in declaration block to the
message or to a stream of messages, effectively declaring which specific
stand-ins are actually in use. This is a one-way stand-in declaration method
where the decoder has no choice over the used stand-ins, thus only the
`required` container is to be used.

If applicable, the encoder SHOULD use the media-type subparameter `standin` to
declare which variant of in-message negotiation it is using.

## Stream handshake negotiation

Both sides of the communication may do a handshake, exchanging their stand-in
declaration blocks in order to align their encoders and decoders. The actual
realization of the handshake is out of the scope of this document, we only
specify the data structure used for it.

## Stand-in declaration block YANG module

~~~ yang
module cbor-yang-standin-declaration-block {
  yang-version 1.1;
  namespace "TODO";

  grouping standin {
    leaf tag {
      type uint64;
      description "CBOR tag value";
    }
    leaf version {
      type uint32;
      description "Version of this tag support, currently always zero";
    }
  }

  container required {
    uses standin;
    description "Stand-in tags always required to be used";
  }

  container supported {
    uses standin;
    description "Stand-in tags supported but not strictly required";
  }
}
~~~

# Security Considerations

TODO Security

# IANA Considerations

## New CBOR Tags {#iana-113}

In the registry "{{cbor-tags (CBOR Tags)<IANA.cbor-tags}}" {{IANA.cbor-tags}},
IANA is requested to assign the tag in {{tab-new-tags}}.

| Tag    | Data Item   | Semantics                                                                            | Reference                              |
| CPA113 | byte string | Expected Later Encoding: colon-separated hexadecimal representation of a byte string | draft-bormann-yang-standin, {{hex-tags}} |
{: #tab-new-tags title="New CBOR Tag Defined by this Specification"}


## stand-in tags?

ISSUE: Do we want to have a separate registry for stand-in tags?

They already are CBOR tags and thus in the registry, but might
get lost in the bulk of that (and are only identified as YANG-CBOR
stand-in Tags in the specification).

I think we should create some registry saying "this tag may encode that types"
and vice versa, the problem may be how to define this registry well. --Maria

## media-type parameters

IANA is requested to register a media-type subparameter `standin`
with allowed values of `appended` and `prepended`.

## Standin declaration block SID allocation

IANA is requested to register a block of 50 SIDs for the
`cbor-yang-standin-declaration-block` module in the IETF YANG-SID Ranges
Registry as per RFC 9595.

TODO: use proper wording, create an actual SID file.

--- back

# Acknowledgments
{:unnumbered}

TODO acknowledge.
