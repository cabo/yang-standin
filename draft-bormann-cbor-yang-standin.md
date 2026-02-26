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
  RFC9581: extended-time
  STD94: cbor
  RFC6021: yang-types
  RFC9562: uuid
  RFC7951: yang-json
  RFC9595: yang-cbor-sid
  IANA.cbor-tags:

informative:
  RFC9557: ixdtf
  RFC9542: mac-address
  I-D.bormann-cbor-notable-tags: notable
  BCP215: yang-tree

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

Decoder:
: The party which receives and parses CBOR data described by YANG.

{::boilerplate bcp14-tagged-bcp14}

# Stand-In Tags

This document defines a new content type and encoding of YANG-modeled
data, based on {{-yang-cbor}}. Whenever a stand-in tag is defined
for a specific type, the encoding as specified in this document
may be used. Otherwise, the encoding is as specified in {{-yang-cbor}}.

This section defines several standard stand-in tags for general use.

## `ietf-yang-types`: Tag 1 (Date/Time) and Tag 100 (Date) {#tag-date-time-basic}

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

## `ietf-yang-types`: Tag 1001 (Extended Date and Time) {#tag-date-time-extended}

{{Section 3 of -legacy-bis}} defines the following types in `ietf-yang-types`:

YANG type | base type | specification | stand-in
date-and-time | string | {{-yang-types}} | tag 1001
date-no-zone | string | {{-legacy-bis}} | tag 1001
date | string | {{-legacy-bis}} | tag 1001
time | string | {{-legacy-bis}} | tag 1001
time-no-zone | string | {{-legacy-bis}} | 1001
{: title="Legacy representations in ietf-yang-types"}

The tag 1001 {{-extended-time}} can unambigously stand for all the
aforementioned types of values. Values of type `time` and `time-no-zone`
are encoded with a date of 1 January 1970.

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
   # "2001:db8:1234:deed:beef:cafe:face:feed"
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
   # "fe80::0202:02ff:ffff:fe03:0303/64%eth0"
~~~

Stand-in representation of IPv4 address 192.0.2.1 is `52(h'c0000201')`.

CBOR encoding of stand-in (7 bytes):

~~~ cbor-pretty
D8 34          # tag(52)
   68          # bytes(4)
      C0000201
~~~

CBOR encoding of legacy representation (10 bytes):

~~~ cbor-pretty
69                    # text(9)
   3139322e302e322e31 # "192.0.2.1"
~~~

Stand-in representation of IPv4 prefix 192.0.2.0/24 is `52([24, h'c0000200'])`.

CBOR encoding of stand-in (10 bytes):

~~~ cbor-pretty
D8 34             # tag(52)
   82             # array(2)
      18 18       # unsigned(24)
      48          # bytes(4)
         C0000200
~~~

CBOR encoding of legacy representation (13 bytes):

~~~ cbor-pretty
6C                          # text(12)
   3139322e302e322e302f3234 # "192.0.2.0/24"
~~~

Stand-in representation of IPv4 address combined with prefix 192.0.2.1/24 is `52([h'c0000201', 24])`.

CBOR encoding of stand-in (10 bytes):

~~~ cbor-pretty
D8 34             # tag(52)
   82             # array(2)
      48          # bytes(4)
         C0000201
      18 18       # unsigned(24)
~~~

CBOR encoding of legacy representation (13 bytes):

~~~ cbor-pretty
6C                          # text(12)
   3139322e302e322e312f3234 # "192.0.2.1/24"
~~~

## Union handling

When the schema specifies a union data type for a node, there are
additional requirements on the encoder and decoder. The encoder MUST
be fully aware of data semantics and use the appropriate data type
and encoding. The decoder MUST use the data type information for further
processing, in a similar way as specified in {{Section 6.10 of -yang-json}}.

# Using Stand-In Tags

Document encoded using stand-in tags is not compatible with a document
encoded according to only {{-yang-cbor}}. It's also not easily possible
to specify the actual usage of stand-in tags globally, as types
may get e.g. subtyped or unionized.

It has been already specified in {{-yang-cbor}} that item names may be
replaced by schema identifiers, and that replacement table (SID file),
specified in {{-yang-cbor-sid}}, needs to be distributed alongside the
YANG modules for the encoded data to make any sense.

This document specifies an additional file, a Standin file, which shall
be distributed in a similar way to how SID files are distributed, whenever
possible. The Standin files SHOULD maintain the same lifecycle as SID
files to help interoperability.

## Standin File Format

Standin files are used to specify which exact standins are used in the
described encoding. The following tree diagram {{-yang-tree}} provides
an overview of the data model:

~~~yang-tree
module: ietf-cbor-standin-file

TODO

~~~

The standin file specifies the encoding for all nodes of a specific type,
or for a singular sid. The encoding of the standin file SHOULD be CBOR with SIDs

~~~yang
module ietf-cbor-standin-file {
  yang-version 1.1;
  namespace "urn:ietf:params:xml:ns:yang:ietf-cbor-standin-file";
  prefix standin;

  ... TODO formal headers

  identity value-encoding {
    description "Base identity for standin value encoding";
  }

  identity default {
    base value-encoding;
    description "Encoding without any standin tag.";
    reference "{{-yang-cbor}}";
  }

  identity tag-date-time-basic {
    base value-encoding;
    description "Use representation with tags 1 and 100 to encode date and time";
    reference "{{tag-date-time-basic}}";
  }

  identity tag-date-time-extended {
    base value-encoding;
    description "Use representation with tag 1001 to encode date and time";
    reference "{{tag-date-time-extended}}";
  }

  identity tag-uuid {
    base value-encoding;
    description "Use representation with tag 37 to encode UUIDs";
    reference "{{tag-uuid}}";
  }

  ...

  identity ipv6-address {
    base value-encoding;
    description "Use representation with tag 54 to encode IPv6 addresses";
  }

  ...

  container standin-file {
  description "contents of the standin file";

  leaf module-name {
    type yang:yang-identifier;
    mandatory true;
    description
      "Name of the YANG module associated with this
      standin file.";
  }

  leaf module-revision {
    type revision-identifier;
    description
      "Revision of the YANG module associated with this standin
      file.
      This leaf is not present if no revision statement is
      defined in the YANG module.";
  }

  leaf sid-file-version {
    type sid-file-version-identifier;
    default 0;
    description
      "Optional leaf that specifies the version number of the
      adjacent '.sid' file which this file depends on.";
  }

  leaf description {
    type string;
    description
      "Free-form meta-information about the standin file.";
  }

  list encoding {
    config false;
    choice encoding-variant {
      case by-type {
        list types {
          key "type";
          leaf type {
            type string;
            description "Type name to be encoded by the specifed standin";
          }
        }
      }
      case by-sid {
        list sids {
          key "sid";
          leaf sid {
            type sid;
            description "SID to be encoded by the specified standim";
//            TODO: Can this be a leafref?
          }
        }
      }
    }
    leaf standin {
      type identityref;
      base standin:value-encoding;
      description "Which value encoding to use";
    }
    }
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


## media-type parameters

IANA is requested to register a value `standin` for media-type subparameter `encoding`,
specifying that some values have been encoded using standins.

## Standin declaration block SID allocation

IANA is requested to register a block of 50 SIDs for the
`ietf-cbor-standin-file` module in the IETF YANG-SID Ranges
Registry as per RFC 9595.

--- back

# Acknowledgments
{:unnumbered}

TODO acknowledge.
