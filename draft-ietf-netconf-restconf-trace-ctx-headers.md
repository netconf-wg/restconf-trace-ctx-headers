---
docname: draft-ietf-netconf-restconf-trace-ctx-headers-latest
title:  RESTCONF Extension to Support Trace Context Headers
abbrev: RESTCONF Trace Context Headers
category: std
date: 2024-03-03

ipr: trust200902
submissiontype: IETF
consensus: true
v: 06
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
    fullname: Kristian Larsson
    organization: Deutsche Telekom AG
    email: kll@dev.terastrm.net

 -
    fullname: Jan Lindblad
    organization: Cisco Systems
    email: jlindbla@cisco.com

normative:
  RFC2119:
  RFC8040:
  RFC8174:
  RFC8446:
  RFC8525:

  I-D.draft-ietf-netconf-trace-ctx-extension:

  W3C-Trace-Context:
    title: W3C Recommendation on Trace Context
    target: https://www.w3.org/TR/2021/REC-trace-context-1-20211123/
    date: 2021-11-23

--- abstract

This document defines an extension to the RESTCONF protocol in order to support Trace Context propagation as defined by the W3C.

--- middle

# Introduction

Network automation and management systems commonly consist of multiple
sub-systems and, together with the network devices they manage, they effectively form a distributed system.  Distributed tracing is a methodology implemented by tracing tools to track, analyze and debug operations such as configuration transactions, across multiple distributed systems.

The W3C has defined two HTTP headers (traceparent and tracestate) in {{W3C-Trace-Context}} for context propagation that are useful for distributed systems like the ones defined in section 4 of {{?RFC8309}}.

According to the W3C specification, each operation is uniquely identified by a "trace-id" field, and carries multiple metadata fields about the operation.  Propagating this Trace Context between systems provides a coherent view of the entire operation as carried out by all involved systems.

In {{I-D.draft-ietf-netconf-trace-ctx-extension}}, the NETCONF protocol extension is defined and we will re-use several of the YANG and XML objects defined in that document for RESTCONF. Please refer to that document for additional context and example applications.

## Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT","SHOULD","SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.

# RESTCONF Extensions

A RESTCONF server that implements the Trace Context propagation mechanism defined in this document MUST support the Trace Context traceparent header as defined in {{W3C-Trace-Context}}.

A RESTCONF server SHOULD support the Trace Context tracestate header as defined in {{W3C-Trace-Context}}.

## Error Handling

A RESTCONF server SHOULD follow the "Processing Model for Working with Trace Context" as specified in {{W3C-Trace-Context}}.  Based on this processing model, it is NOT RECOMMENDED to reject an RPC because of the Trace Context header values.

If a server decides to reject an RPC because of the Trace Context header values, the server MUST return a RESTCONF rpc-error with the following values:

      error-tag:      operation-failed
      error-type:     protocol
      error-severity: error

 Additionally, the error-info tag SHOULD contain a relevant details about the error.

 Finally, the sx:structure defined in {{I-D.draft-ietf-netconf-trace-ctx-extension}} SHOULD be present in any error message from the server.

## Trace Context Header Versioning

The RESTCONF protocol extension described in this document refers to the {{W3C-Trace-Context}} Trace Context capability. The W3C traceparent and tracestate headers include the notion of versions. It would be desirable for a RESTCONF client to be able to discover the one or multiple versions of these headers supported by a server.

{{I-D.draft-ietf-netconf-trace-ctx-extension}} defines a pair YANG modules that SHOULD be included in the YANG library per {{RFC8525}} of the RESTCONF server supporting the RESTCONF Trace Context extension that will refer to the headers' supported versions.

# Security Considerations

The related document {{I-D.draft-ietf-netconf-trace-ctx-extension}} defines two YANG modules that are used when implementing the Trace Context concept, regardless of YANG-based protocol.  These modules are completely empty, and therefore have very limited security considerations. Their purpose is only to indicate which Trace Context header versions the server supports using YANG Library {{RFC8525}}.

The traceparent and tracestate headers make it easier to track and correlate the flow of requests and their downstream effect on other systems.  This is indeed the whole point with these headers.  This knowledge may be used by unauthorized entities to infer a map of a managed network.

All advice mentioned in the {{W3C-Trace-Context}} under the Privacy Considerations and Security Considerations also apply to this document.

The lowest RESTCONF layer is HTTPS, and the mandatory-to-implement secure transport is TLS [RFC8446].

# IANA Considerations

This document has no IANA actions.

# Acknowledgments

The authors would like to acknowledge the valuable implementation feedback from Christian Rennerskog and Per Andersson.  Many thanks to Raul Rivas Felix, Alexander Stoklasa, Luca Relandini and Erwin Vrolijk for their help with the demos regarding integrations.  The help and support from Med Boucadair, Jean Quilbeuf and Benoît Claise has also been invaluable to this work.

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

{{W3C-Trace-Context}} specifies that a vendor may validate the tracestate header and that invalid headers may be discarded.[Error handling](#error-handling), states that servers may return an error. Let's assume that an implementation follows that behavior.

Example of a badly formated tracestate header using {{RFC8040}} example (Appendix B.2.1), in which a server receives the following:

      POST /restconf/data/example-jukebox:jukebox/library HTTP/1.1
      Host: example.com
      Content-Type: application/yang-data+json
      traceparent: 00-405062f633be64ee006089dfca95a153-e021f9e263aad8e2-01
      tracestate: SomeBadFormatHere

      {
        "example-jukebox:artist" : [
          {
            "name" : "Foo Fighters"
          }
        ]
      }

To which the server responds with an error message:

     HTTP/1.1 400 Bad Request
     Date: Thu, 20 Jun 2024 20:56:30 GMT
     Server: example-server
     Content-Type: application/yang-data+json

     { "ietf-restconf:errors" : {
         "error" : [
            {
              "error-type" : "protocol",
              "error-tag" : "operation-failed",
              "error-severity" : "error",
              "error-message" :
              "Context traceparent header incorrectly formatted",
              "error-info": {
                "ietf-trace-context:trace-context-error-info" : {
                  "ietf-trace-context:meta-name" : "tracestate",
                  "ietf-trace-context:meta-value" :
                  "SomeBadFormatHere",
                  "ietf-trace-context:error-type" :
                  "ietf-trace-context:bad-format"
                }
              }
            }
         ]
       }
     }

# Changes (to be deleted by RFC Editor)
## From version 05 to 06
- More missing edits

## From version 04 to 05
- Removed unused references and terminology

## From version 03 to 04
- Abbreviation change
- "ietf-trace-contex:trace-context-error-info" should have been a container in example

## From version 02 to 03
- Added abbreviations to terminology
- error messages are SHOULD to align with W3C handling.
- Addapted example to YANG module changes in reference.

## From version 01 to 02
- Added WGLC comments
- Changed namespaces and module name
- Fix error in error response
- Comments from Med Boucadair
- Removed markdown formatting of tracestate and traceparent, as toolchain could not handle this properly
- Removed references to RFC8341 (NACM) as the passage in the security considerations no longer need it
- Rearranged text in introduction to include referenes in a more natural order
- Removed several references to "we" and replaced with more neutral language
- Clarified that everything described as MUST requirements in this document only apply to RESTCONF implementations that implement this document. Other RESTCONF implementations do not need to care about this document, it's an optional extension
- Clarified that the YANG modules used by this document is defined by the sibling document for NETCONF
- Lots of updated wording based on review feedback

## From version 00 to -01
- Added Security considerations
- Added Acknowledgements
- Added several Normative references
- Added links to latest document on github
- Added RESTCONF example for success and error
- Modified Error Handling to reflect better W3C alignment based on implementation feedback
- Firmed up error handling and YANG-library to MUST-requirements

## From version 00 to draft-ietf-netconf-restconf-trace-ctx-headers-00
- Adopted by NETCONF WG
- Moved repository to NETCONF WG
- Changed build system to use martinthomson's excellent framework
- Ran make fix-lint to remove white space at EOL etc.
- Added this change note. No other content changes
