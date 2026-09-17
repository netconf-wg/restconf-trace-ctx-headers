---
docname: draft-ietf-netconf-restconf-trace-ctx-headers-latest
title:  RESTCONF Extension to Support Trace Context Headers
abbrev: RESTCONF Trace Context Headers
category: std
date: 2026-09-17

ipr: trust200902
submissiontype: IETF
consensus: true
v: 11
area: Operations and Management
workgroup: NETCONF
keyword:
 - telemetry
 - distributed systems
 - opentelemetry
venue:
  group: NETCONF
  type: Working Group
  mail: netconf@ietf.org
  arch: https://mailarchive.ietf.org/arch/browse/netconf/
  github: https://github.com/netconf-wg/restconf-trace-ctx-headers
  latest: https://github.com/netconf-wg/restconf-trace-ctx-headers/blob/gh-pages/draft-ietf-netconf-restconf-trace-ctx-headers.txt

stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
 -
    fullname: Roque Gagliano
    organization: Cisco Systems
    street: Avenue des Uttins 5
    code: 1180
    city: Rolle
    country: Switzerland
    email: rogaglia@cisco.com

 -
    fullname: Christian Rennerskog
    organization: Cisco Systems
    email: crenners@cisco.com

 -
    fullname: Kristian Larsson
    organization: Deutsche Telekom AG
    email: kll@dev.terastrm.net

 -
    fullname: Jan Lindblad
    organization: All For Eco
    email: jan.lindblad+ietf@for.eco

normative:
  RFC2119:
  RFC8040:
  RFC8174:
  RFC8446:
  RFC8525:
  RFC9000:

  I-D.draft-ietf-netconf-trace-ctx-extension:

  W3C-Trace-Context:
    title: W3C Recommendation on Trace Context
    target: https://www.w3.org/TR/2021/REC-trace-context-1-20211123/
    date: 2021-11-23

--- abstract

This document defines an extension to the RESTCONF protocol to support Trace Context propagation as defined by the W3C.

--- middle

# Introduction

Network automation and management systems commonly consist of multiple subsystems and, together with the network devices they manage, effectively form a distributed system. Distributed tracing is a methodology implemented by tracing tools to track, analyze, and debug operations such as configuration transactions across multiple distributed systems.

The W3C has defined two HTTP headers, traceparent and tracestate, in {{W3C-Trace-Context}} for context propagation. These headers are useful for distributed systems such as those described in Section 4 of {{?RFC8309}}. While the traceparent header is portable and mandatory, the tracestate header is optional and is used to carry vendor-specific data in a set of key/value pairs.

According to the W3C specification, each operation is uniquely identified by a "trace-id" field and carries multiple metadata fields about the operation. Propagating this Trace Context between systems provides a coherent view of the entire operation as carried out by all involved systems.

In {{I-D.draft-ietf-netconf-trace-ctx-extension}}, the NETCONF protocol extension is defined, and we reuse several of the YANG and XML objects defined in that document for RESTCONF. Please refer to that document for additional context and example applications.

## Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.

# RESTCONF Extensions

A RESTCONF server that implements the Trace Context propagation mechanism defined in this document MUST support the Trace Context traceparent header as defined in {{W3C-Trace-Context}}.

A RESTCONF server MAY support the Trace Context tracestate header as defined in {{W3C-Trace-Context}}. Note that while the W3C Trace Context specification mandates that tracing tools forwarding traces MUST propagate both traceparent and tracestate headers (Section 2.3 of {{W3C-Trace-Context}}), RESTCONF servers may act as trace endpoints rather than forwarding intermediaries. Since the tracestate header carries vendor-specific opaque data, its support is intentionally made optional to accommodate implementations that do not require vendor-specific trace context propagation.

When interacting with these headers, the RESTCONF server follows the specifications of section 2.3 in {{W3C-Trace-Context}}, therefore a RESTCONF server MAY also participate in a trace by modifying the traceparent header and the relevant vendor-specific parts of the tracestate header, for example when acting as an intermediary that contributes its own span to the distributed trace.

## Error Handling

It is NOT RECOMMENDED to reject an RPC because of Trace Context header values.

If a server decides to reject an RPC because of Trace Context header values, the server MUST return a RESTCONF rpc-error with the following values:

      error-tag:      operation-failed
      error-type:     protocol
      error-severity: error

Additionally, the error-info tag SHOULD contain relevant details about the error.

Finally, the sx:structure defined in {{I-D.draft-ietf-netconf-trace-ctx-extension}} SHOULD be present in any error message from the server.

## Trace Context Header Versioning

The RESTCONF protocol extension described in this document refers to the {{W3C-Trace-Context}} Trace Context capability. The W3C traceparent and tracestate headers include the notion of versions. It would be desirable for a RESTCONF client to be able to discover the one or multiple versions of these headers supported by a server.

{{I-D.draft-ietf-netconf-trace-ctx-extension}} defines a pair YANG modules that SHOULD be included in the YANG library per {{RFC8525}} of the RESTCONF server supporting the RESTCONF Trace Context extension that will refer to the headers' supported versions.

# Security Considerations

The traceparent and tracestate headers make it easier to track and correlate the flow of requests and their downstream effects on other systems. This information may be used by unauthorized entities to infer a map of a managed network.

All advice mentioned in {{W3C-Trace-Context}} under Privacy Considerations and Security Considerations also applies to this document.

The RESTCONF protocol has to (1) use a secure transport layer (for example, TLS {{RFC8446}} and QUIC {{RFC9000}}) and (2) use mutual authentication.

# IANA Considerations

This document has no IANA actions.

# Acknowledgments

The authors would like to acknowledge the valuable implementation feedback from Per Andersson. Many thanks to Raul Rivas Felix, Alexander Stoklasa, Luca Relandini, and Erwin Vrolijk for their help with the demo integrations. The help and support from Med Boucadair, Jean Quilbeuf, and Benoît Claise have also been invaluable to this work.

--- back

# Example RESTCONF Calls

All examples from Appendix B of {{RFC8040}} could be recreated in this section by adding the new header described in this document. We selected one example from that document as reference.

## Successful creation of New Data Resources (from Appendix B.2.1 of {{RFC8040}})

To create a new "artist" resource within the "library" resource, a client might send the following request:

      POST /restconf/data/example-jukebox:jukebox/library HTTP/1.1
      Host: example.com
      Content-Type: application/yang-data+json
      traceparent: 00-405062f633be64ee006089dfca95a153-e021f9e263aad8e2-01
      tracestate: vendorname1=opaqueValue1,vendorname2=opaqueValue2

      {
        "example-jukebox:artist" : [
          {
            "name" : "Foo Fighters"
          }
        ]
      }

If the resource is created, the server might respond as follows:

      HTTP/1.1 201 Created
      Date: Thu, 26 Jan 2017 20:56:30 GMT
      Server: example-server
      Location: https://example.com/restconf/data/\
          example-jukebox:jukebox/library/artist=Foo%20Fighters
      Last-Modified: Thu, 26 Jan 2017 20:56:30 GMT
      ETag: "b3830f23a4c"
      traceparent: 00-405062f633be64ee006089dfca95a153-e021f9e263aad8e2-01
      tracestate: vendorname1=opaqueValue1,vendorname2=opaqueValue2

## Unsuccessful creation of New Data Resources (from Appendix B.2.1 of {{RFC8040}})

{{W3C-Trace-Context}} specifies that a vendor MAY validate the tracestate header and that the processing rules SHOULD be followed.

Example of a badly formatted tracestate header using the {{RFC8040}} example (Appendix B.2.1), in which a server receives a higher traceparent version 03:

      POST /restconf/data/example-jukebox:jukebox/library HTTP/1.1
      Host: example.com
      Content-Type: application/yang-data+json
      traceparent: 03-405062f633be64ee006089dfca95a153-e021f9e263aad8e2-01
      tracestate: SomeBadFormatHere

      {
        "example-jukebox:artist" : [
          {
            "name" : "Foo Fighters"
          }
        ]
      }

In this case, the server cannot parse the traceparent header and the response would be:

      HTTP/1.1 201 Created
      Date: Thu, 26 Jan 2017 20:56:30 GMT
      Server: example-server
      Location: https://example.com/restconf/data/\
          example-jukebox:jukebox/library/artist=Foo%20Fighters
      Last-Modified: Thu, 26 Jan 2017 20:56:30 GMT
      ETag: "b3830f23a4c"
      traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-00

Note that the API call was successful, but the traceparent header is new with its trace-flags set to 0 and the tracestate header was removed.

# Changes (to be deleted by RFC Editor)

## From version 10 to 11
- Grammar cleanups

## From version 09 to 10
- Updated security considerations based on the Shepherd review.
- Added Christian Rennerskog as a co-author.

## From version 08 to 09
- Explained MAY vs. W3C MUST for tracestate (OPSDir comment).
- Mentioned "participating in a trace" (modifying headers) (OPSDir comment).
- Fixed RFC 2119 boilerplate spacing to match the exact BCP 14 text.
- Updated the document date.

## From version 07 to 08
- Improved the error-handling example to show the most common scenario based on the W3C standard.
- Updated the dates.
- Several edits based on OpsDir comments.
- Added security considerations for QUIC and mutual authentication.

## From version 06 to 07
- More missing edits; updated dates.

## From version 05 to 06
- More missing edits.

## From version 04 to 05
- Removed unused references and terminology.

## From version 03 to 04
- Abbreviation change.
- "ietf-trace-contex:trace-context-error-info" should have been a container in the example.

## From version 02 to 03
- Added abbreviations to the terminology.
- Error messages are SHOULD to align with W3C handling.
- Adapted the example to YANG module changes in the reference.

## From version 01 to 02
- Added WGLC comments.
- Changed namespaces and module name.
- Fixed an error in the error response.
- Comments from Med Boucadair.
- Removed markdown formatting of tracestate and traceparent, as the toolchain could not handle it properly.
- Removed references to RFC 8341 (NACM), as the passage in the security considerations no longer needed it.
- Rearranged the text in the introduction to include references in a more natural order.
- Removed several references to "we" and replaced them with more neutral language.
- Clarified that everything described as MUST requirements in this document applies only to RESTCONF implementations that support this document; other RESTCONF implementations do not need to care about it, as it is an optional extension.
- Clarified that the YANG modules used by this document are defined by the sibling NETCONF document.
- Lots of updated wording based on review feedback.

## From version 00 to -01
- Added Security Considerations.
- Added Acknowledgments.
- Added several normative references.
- Added links to the latest document on GitHub.
- Added a RESTCONF example for success and error.

- Modified Error Handling to reflect better W3C alignment based on implementation feedback
- Firmed up error handling and YANG-library to MUST-requirements

## From version 00 to draft-ietf-netconf-restconf-trace-ctx-headers-00
- Adopted by NETCONF WG
- Moved repository to NETCONF WG
- Changed build system to use martinthomson's excellent framework
- Ran make fix-lint to remove white space at EOL etc.
- Added this change note. No other content changes
