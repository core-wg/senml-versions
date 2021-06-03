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
date: 2021-05-09
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


The Sensor Measurement Lists (SenML) specification {{-senml}} provides a version
number that is initially set to 10, without further
specification on the way to make use of different version numbers.

The traditional idea of using a version number to indicate the
evolution of an interchange format generally assumes an incremental
progression of the version number as the format accretes additional
features over time.
However, in the case of SenML, it is expected that the likely evolution
will be for independently selectable capability _features_ to be added
to the basic specification that is indicated by version number 10.
To support this model, this document repurposes the single version
number accompanying a pack of SenML records so that it is interpreted
as a bitmap that indicates the set of features a recipient would need
to have implemented to be able to process the pack.

This short document specifies the use of SenML Features and maps
them to SenML version number space, updating {{-senml}}.

## Terminology

{::boilerplate bcp14}

Where bit arithmetic is explained, this document uses the notation
familiar from the programming language C {{C}}, including the `0b`
prefix for binary numbers defined in Section 5.13.2 of the C++
language standard {{Cplusplus}}, except that superscript notation
(example for two to the power of 64: 2<sup>64</sup>) denotes
exponentiation; in the plain text version of this draft, superscript
notation is rendered in paragraph text by C-incompatible surrogate
notation as seen in this example, and in display math by a crude
plaintext representation, as is the sum (Sigma) sign.

# Feature Codes and the Version number

The present specification defines "SenML Features", each identified by a "feature
name" (a text string) and a "feature code" (an unsigned integer less
than 53).

The specific version of a SenML pack is composed of a set of
features.
The SenML version number (`bver` field) is then a bitmap of these
features, specifically the sum of, for each feature present, two taken
to the power of the feature code of that feature.

~~~ math
version = \sum_{fc=0}^{52} present(fc)  ⋅  2^{fc}
~~~

where present(fc) is 1 if the feature with the feature code `fc` is
present, 0 otherwise.
(The expression 2<sup>fc</sup> can be implemented as `1 << fc` in C
and related languages.)

## Discussion

Representing features as a bitmap within a number is quite efficient as long as
feature codes are sparingly allocated (see also {{iana}}).

Compatibility with the existing SenML version number, 10 decimal
(0b1010), requires reserving four of the least significant bit
positions for the base version as described in {{resv}}.
There is an upper limit to the range of the integer numbers that can
be represented in all SenML representations: practical JSON limits
this to 2<sup>53</sup>-1 {{-i-json}}.
This means the feature codes 4 to 52 are available, one of which is
taken by the feature defined in {{secondary-units}}, leaving 48 for allocation.
(The current version 10 (with all other feature codes unset) can be
visualized as `0b00000000000000000000000000000000000000000000000001010`.)
For a lifetime of this scheme of several decades, approximately two feature codes
per year or fewer should be allocated.
Note that less generally applicable features can always be
communicated via fields labeled with names that end with the "_"
character ("must-understand fields"), see {{Section 4.4 of -senml}}.)

Most representations visible to engineers working with SenML will use
decimal numbers, e.g., 26 (0b11010, 0x1a) for a version that adds the
"Secondary Units" feature ({{secondary-units}}).  This is sightly unwieldy, but
will be quickly memorized in practice.

## Updating {{Section 4.4 of -senml}}

The last paragraph of {{Section 4.4 of -senml}} may be read to give
the impression that SenML version numbers are totally ordered, i.e.,
that an implementation that understands version n also always
understands all versions k < n.
If this ever was true for SenML versions before 10, it certainly is no
longer true with this specification.

Any SenML pack that sets feature bits beyond the first four will
lead to a version number that actually is greater than 10, so the
requirement in {{Section 4.4 of -senml}} will prevent false
interoperability with version 10 implementations.

Implementations that do implement feature bits beyond the first four,
i.e., versions greater than 10, will instead need to perform a bitwise
comparison of the feature bitmap as described in this specification
and ensure that all features indicated are understood before using the
pack.
E.g., an implementation that implements basic SenML (version number
10) plus only a future feature code 5, will accept version number 42,
but would not be able to work with a pack indicating version number
26 (base specification plus feature code 4).
(If the implementation *requires* feature code 5 without being
backwards compatible, it will accept 42, but not 10.)

# Features: Reserved0, Reserved1, Reserved2, Reserved3 {#resv}

For SenML Version 10 as described in {{-senml}}, the feature codes 0 to 3 are already in use.
Reserved1 (1) and Reserved3 (3) are always present
and the features Reserved0 (0) and Reserved2 (2) are always absent,
yielding a version number of 10 if no other feature is in use.
These four reserved feature codes are not to be used with any more specific
semantics except in a specification that updates the present specification.

# Feature: Secondary Units {#secondary-units}

The feature "Secondary Units" (code number 4) indicates that secondary
unit names {{-units}} MAY be used in the "u" field of SenML Records, in addition to the
primary unit names already allowed by {{-senml}}.

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

To facilitate the use of feature names in programs, the designated
expert is requested to ensure that feature names are usable as
identifiers in most programming languages, after lower-casing the
feature name in the registry entry and replacing whitespace with
underscores or hyphens, and that they also are distinct in this form.

The initial content of this registry is as follows:

| Feature code | Feature name    | Specification |
|            0 | Reserved0       | RFCthis       |
|            1 | Reserved1       | RFCthis       |
|            2 | Reserved2       | RFCthis       |
|            3 | Reserved3       | RFCthis       |
|            4 | Secondary Units | RFCthis, {{-units}} |
{: #feat title="Features defined for SenML at the time of writing"}

As the number of features that can be registered has a hard limit (48
codes left at the time of writing), the designated expert is
specifically instructed to maintain a frugal regime of code point
allocation, keeping code points available for SenML Features that are
likely to be useful for non-trivial subsets of the SenML ecosystem.
Quantitatively, the expert could for instance steer the allocation to
not allocate more than 10 % of the remaining set per year.

Where the specification of the feature code is provided in a document
that is separate from the specification of the feature itself (as with
feature code 4 above), both specifications should be listed.

--- back

# Acknowledgements {#acks}
{: numbered="no"}

{{{Ari Keränen}}} proposed to use the version number as a bitmap and
provided further input on this specification.
{{{Jaime Jiménez}}} helped clarify the document by providing a review.
{{{Elwyn Davies}}} provided a detailed GENART review, with directly
implementable text suggestions that now form part of this specification.

<!--  LocalWords:  selectable subregistry whitespace
 -->
