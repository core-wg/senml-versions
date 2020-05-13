---
stand_alone: true
ipr: trust200902
docname: draft-ietf-core-senml-versions-latest
keyword: Internet-Draft
cat: std
updates: 8428
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
date: 2020-05-13
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
  I-D.ietf-core-senml-more-units: units


--- abstract

This short document updates RFC 8428, Sensor Measurement Lists
(SenML), by specifying the use of independently selectable "SenML
Features" and mapping them to SenML version numbers.

--- middle


# Introduction {#intro}


The Sensor Measurement Lists (SenML) specification {{RFC8428}} provides a version
number that is initially set to 10, without further
specification on the way to make use of different version numbers.

The traditional idea of using a version number for evolving an
interchange format presupposes a linear progression of that format.
A more likely form of evolution of SenML is the addition of
independently selectable "features"
that can be added to the base version (version 10) in a fashion that
these are mostly independent of each other.  A recipient of a SenML pack can check the
features it implements against those required by the pack, processing the
pack only if all required features are provided in the implementation.

This short document specifies the use of SenML Features and maps
them to SenML version number space, updating {{RFC8428}}.

{::boilerplate bcp14}

Where bit arithmetic is explained, this document uses
the notation familiar from the programming language C, except that
"\*\*" denotes exponentiation.

# Feature Codes and the Version number

The present specification defines "SenML Features", each identified by a "feature
name" (a text string) and a "feature code", an unsigned integer less
than 53.

The specific version of a SenML pack is composed of a set of
features.
The SenML version number (`bver` field) is then a bitmap of these
features, specifically the sum of, for each feature present, two taken
to the power of the feature code of that feature.

$$ version ~=~ \sum_{fc=0}^{52} ~~ present(fc) x 2^{fc} $$

where present(fc) is 1 if the feature with the feature code `fc` is
present, 0 otherwise.

# Features: Reserved0, Reserved1, Reserved2, Reserved3

For SenML Version 10 as described in {{RFC8428}}, the feature codes 0 to 3 are already in use.
Reserved1 (1) and Reserved3 (3) are always present
and the features Reserved0 (0) and Reserved2 (2) are always absent,
yielding a version number of 10 if no other feature is in use.
These four reserved feature codes are not to be used with any more specific
semantics except in a specification that updates the present specification.

# Feature: Secondary Units

The feature "Secondary Units" (code number 4) indicates that secondary
unit names {{-units}} MAY be be used in the "u" field of SenML Records, in addition to the
primary unit names already allowed by {{RFC8428}}.

Note that the most basic use of this feature simply sets the SenML
version number to 26 (10 + 2**4).

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
