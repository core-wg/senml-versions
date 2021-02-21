---
stand_alone: true
ipr: trust200902
docname: draft-ietf-core-senml-versions-latest
keyword: Internet-Draft
cat: std
updates: 8428
wg: CoRE
pi:
  toc: 'yes'
  tocompact: 'yes'
  tocdepth: '3'
  tocindent: 'yes'
  symrefs: 'yes'
  sortrefs: 'yes'
  comments: 'yes'
  inline: 'yes'
  strict: 'no'
  compact: 'yes'
  subcompact: 'no'
title: >
  SenML Features and Versions
abbrev: >
  SenML Features and Versions
date: 2021-02-12
author:
-
  ins: C. Bormann
  name: Carsten Bormann
  org: Universitaet Bremen TZI
  street: Postfach 330440
  city: Bremen
  code: D-28359
  country: Germany
  phone: +49-421-218-63921
  email: cabo@tzi.org

normative:
  RFC8428: senml
  RFC8126: ianacons
  IANA.senml:
  RFC8798: units
  C:
    title: Information technology — Programming languages — C
    author:
      org: International Organization for Standardization
    date: 2018-06
    target: https://www.iso.org/standard/74528.html
    seriesinfo:
      ISO/IEC: 9899:2018, Fourth Edition
  Cplusplus:
    title: Programming languages — C++
    author:
      org: International Organization for Standardization
    date: 2020-12
    target: https://www.iso.org/standard/79358.html
    seriesinfo:
      ISO/IEC: 14882:2020, Sixth Edition
informative:
  RFC7493: i-json

--- abstract

This short document updates RFC 8428, Sensor Measurement Lists
(SenML), by specifying the use of independently selectable "SenML
Features" and mapping them to SenML version numbers.

--- to_be_removed_note_Discussion_Venues

Discussion of this document takes place on the
CORE Working Group mailing list (core@ietf.org),
which is archived at
<https://mailarchive.ietf.org/arch/browse/core/>.

Source for this draft and an issue tracker can be found at
<https://github.com/core-wg/senml-versions>.

--- middle


# Introduction {#intro}


The Sensor Measurement Lists (SenML) specification {{RFC8428}} provides a version
number that is initially set to 10, without further
specification on the way to make use of different version numbers.

The traditional idea of using a version number for evolving an
interchange format presupposes a linear progression of that format.
A more likely form of evolution of SenML is the addition of
independently selectable _features_
that can be added to the base version (version 10) in a fashion that
these are mostly independent of each other.  A recipient of a SenML pack can check the
features it implements against those required by the pack, processing the
pack only if all required features are provided in the implementation.

This short document specifies the use of SenML Features and maps
them to SenML version number space, updating {{RFC8428}}.

## Terminology

{::boilerplate bcp14}

Where bit arithmetic is explained, this document uses the notation
familiar from the programming language C {{C}}, including the `0b`
prefix for binary numbers defined in Section 5.13.2 of the C++
language standard {{Cplusplus}}, except that superscript notation
(example for two to the power of 64: 2<sup>64</sup>) denotes
exponentiation; in the plain text version of this draft, superscript
notation is rendered by C-incompatible surrogate notation as seen in
this example.

# Feature Codes and the Version number

The present specification defines "SenML Features", each identified by a "feature
name" (a text string) and a "feature code", an unsigned integer less
than 53.

The specific version of a SenML pack is composed of a set of
features.
The SenML version number (`bver` field) is then a bitmap of these
features, specifically the sum of, for each feature present, two taken
to the power of the feature code of that feature.

~~~ math
version = \sum_{fc=0}^{52} present(fc) ⋅ 2^{fc}
~~~

where present(fc) is 1 if the feature with the feature code `fc` is
present, 0 otherwise.

## Discussion

Representing features as a bitmap within a number is quite efficient as long as
feature codes are sparingly allocated (see also {{iana}}).

Compatibility with the existing SenML version number, 10 decimal
(0b1010), requires reserving four of the lower-most bit positions {{resv}}.
There is an upper limit to the range of the integer numbers that can
be represented in all SenML representations: practical JSON limits
this to 2<sup>53</sup>-1 {{-i-json}}.
This means the feature codes 4 to 52 are available, one of which is
taken by {{secu}}, leaving 48 for allocation.
(The current version 10 (with all other feature codes unset) can be
visualized as `0b00000000000000000000000000000000000000000000000001010`.)
For a lifetime of this scheme of several decades, approximately two feature codes
per year or less should be allocated.
(More boutique features can always be communicated by must-understand
fields, see {{Section 4.4 of RFC8428}}.)

Most representations visible to engineers working with SenML will use
decimal numbers, e.g. 26 (0b11010, 0x1a) for a version that adds the
"Secondary Units" feature ({{secu}}).  This is sightly unwieldy, but
will be quickly memorized in practice.

# Features: Reserved0, Reserved1, Reserved2, Reserved3 {#resv}

For SenML Version 10 as described in {{RFC8428}}, the feature codes 0 to 3 are already in use.
Reserved1 (1) and Reserved3 (3) are always present
and the features Reserved0 (0) and Reserved2 (2) are always absent,
yielding a version number of 10 if no other feature is in use.
These four reserved feature codes are not to be used with any more specific
semantics except in a specification that updates the present specification.

# Feature: Secondary Units {#secu}

The feature "Secondary Units" (code number 4) indicates that secondary
unit names {{-units}} MAY be be used in the "u" field of SenML Records, in addition to the
primary unit names already allowed by {{RFC8428}}.

Note that the most basic use of this feature simply sets the SenML
version number to 26 (10 + 2<sup>4</sup>).

# Security Considerations {#seccons}

The security considerations of {{-senml}} apply.
This specification provides structure to the interpretation of the
SenML version number, which poses no additional security
considerations except for some potential for surprise that version
numbers do not simply increase linearly.

# IANA Considerations {#iana}

IANA is requested to create a new subregistry "SenML features" within the SenML
registry {{IANA.senml}}, with the registration policy "specification required" {{RFC8126}}
and the columns:

* Feature code (an unsigned integer less than 53)
* Feature name (text)
* Specification

The initial content of this registry is as follows:

| Feature code | Feature name    | Specification |
|            0 | Reserved0       | RFCthis       |
|            1 | Reserved1       | RFCthis       |
|            2 | Reserved2       | RFCthis       |
|            3 | Reserved3       | RFCthis       |
|            4 | Secondary Units | RFCthis       |
{: #feat title="Features defined for SenML at the time of writing"}

As the number of features that can be registered has a hard limit (48
codes left at the time of writing), the designated expert is
specifically instructed to maintain a frugal regime of code point
allocation, keeping code points available for SenML Features that are
likely to be useful for non-trivial subsets of the SenML ecosystem.
Quantitatively, the expert could for instance steer the allocation to
not allocate more than 10 % of the remaining set per year.

--- back

# Acknowledgements {#acks}
{: numbered="no"}

Ari Keranen proposed to use the version number as a bitmap and
provided further input on this specification.
