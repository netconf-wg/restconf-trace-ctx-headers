---
docname: draft-ietf-netconf-restconf-trace-ctx-headers-latest
title:  RESTCONF Extension to support Trace Context Headers
abbrev: rc_trace
category: std
date: 2024-09-25

ipr: trust200902
submissiontype: IETF
consensus: true
v: 0
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
  github: netconf-wg/restconf-trace-ctx-headers
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
  RFC8341:
  RFC8446:
  RFC8525:

  I-D.draft-ietf-netconf-trace-ctx-extension-01:

  W3C-Trace-Context:
    title: W3C Recommendation on Trace Context
    target: https://www.w3.org/TR/2021/REC-trace-context-1-20211123/
    date: 2021-11-23


--- abstract

This document extends the RESTCONF protocol in order to support Trace Context propagation as defined by the W3C.

--- middle

# Introduction

Network automation and management systems commonly consist of multiple
sub-systems and together with the network devices they manage, they effectively form a distributed system.  Distributed tracing is a methodology implemented by tracing tools to follow, analyze and debug operations, such as configuration transactions, across multiple distributed systems.

The W3C has defined two HTTP headers (traceparent and tracestate) in {{W3C-Trace-Context}} for context propagation that are useful for distributed systems like the ones defined in {{?RFC8309}}.

According to the W3C specification, each operation is uniquely identified by a "trace-id" field, and carries multiple metadata fields about the operation.  Propagating this Trace Context between systems enables forming a coherent view of the entire operation as carried out by all involved systems.

The goal of this document is to adopt the W3C {{W3C-Trace-Context}} specification as optional headers for use with the RESTCONF protocol.

In {{I-D.draft-ietf-netconf-trace-ctx-extension-01}}, the NETCONF protocol extension is defined and many of the YANG and XML objects defined in that document are re-used for RESTCONF.  Please refer to that document for additional context and example applications.

## Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT","SHOULD","SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.

# RESTCONF Extensions

A RESTCONF server MUST support the Trace Context traceparent header as defined in {{W3C-Trace-Context}}.

A RESTCONF server SHOULD support the Trace Context tracestate header as defined in {{W3C-Trace-Context}}.

## Error Handling

The RESTCONF server SHOULD follow the "Processing Model for Working with Trace Context" as specified in {{W3C-Trace-Context}}.  Based on this processing model, it is NOT RECOMMENDED to reject an RPC because of the Trace Context header values.

If the server still decides to reject the RPC because of the Trace Context header values, the server MUST return a RESTCONF rpc-error with the following values:

      error-tag:      operation-failed
      error-type:     protocol
      error-severity: error

 Additionally, the error-info tag MUST contain relevant details about the error in the form of an sx:structure otlp-trace-context-error-info defined in ietf-netconf-otlp-context.yang from {{I-D.draft-ietf-netconf-trace-ctx-extension-01}}.

## Trace Context Header Versioning

This extension refers to the {{W3C-Trace-Context}} Trace Context capability. The W3C traceparent and tracestate headers include the notion of versions. It would be desirable for a RESTCONF client to be able to discover the one or multiple versions of these headers supported by a server.

To achieve this goal while avoiding the definition of new RESTCONF capabilities for each headers' version, {{I-D.draft-ietf-netconf-trace-ctx-extension-01}} defines a pair of YANG modules that MUST be included in the YANG library per {{RFC8525}} of the RESTCONF server supporting the RESTCONF Trace Context extension that will refer to the headers' supported versions. Future updates of this document could include additional YANG modules for new headers' versions.

# Security Considerations

The related document {{I-D.draft-ietf-netconf-trace-ctx-extension-01}} defines two YANG modules that are used when implementing the Trace Context concept, regardless of YANG-based protocol.  These modules are completely empty, and therefore have very limited security considerations. Their purpose is only to indicate which Trace Context header versions the server supports using YANG Library {{RFC8525}}.

Even though both YANG modules are empty, there are still some security considerations worth mentioning, however.  This is because the functionality described in this document is in the form of additional HTTP headers (which cannot be described using YANG) relating to the network management protocol RESTCONF [RFC8040].

The traceparent and tracestate headers make it easier to track the flow of requests and their downstream effect on other systems.  This is indeed the whole point with these headers.  This knowledge could also be of use to bad actors that are working to build a map of the managed network.

All advice mentioned in the {{W3C-Trace-Context}} under the Privacy Considerations and Security Considerations also apply to this document.

The lowest RESTCONF layer is HTTPS, and the mandatory-to-implement secure transport is TLS [RFC8446].

The Network Configuration Access Control Model (NACM) [RFC8341] provides the means to restrict access for particular NETCONF or RESTCONF users to a preconfigured subset of all available NETCONF or RESTCONF protocol operations and content.

# IANA Considerations

This document has no IANA actions.

# Acknowledgments

The authors would like to acknowledge the valuable implementation feedback from Christian Rennerskog and Per Andersson.  Many thanks to Raul Rivas Felix, Alexander Stoklasa, Luca Relandini and Erwin Vrolijk for their help with the demos regarding integrations.  The help and support from Jean Quilbeuf and Benoît Claise has also been invaluable to this work. Many thanks to Tom Petch and Med Boucadair for their reviews.

--- back

# Example RESTCONF calls

All examples from {{RFC8040}} Appendix B could be recreated in this section by adding the new header described in this document. We selected one example from that document as reference.

## Successful creation New Data Resources (from section B.2.1 in {{RFC8040}})

To create a new "artist" resource within the "library" resource, the client might send the following request:

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

## Unsuccessful creation New Data Resources (from section B.2.1 in {{RFC8040}})

{{W3C-Trace-Context}} specifies that vendor MAY validate the tracestate header and that invalid headers MAY be discarded. In the section about [Error handling](#error-handling), it is stated that servers MAY return an error. Let's assume that is our implementation.

Example of a badly formated tracestate header using {{RFC8040}} example B.2.1, which by following :

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

And the expected error message:

     HTTP/1.1 400 Bad Request
     Date: Tue, 20 Jun 2023 20:56:30 GMT
     Server: example-server
     Content-Type: application/yang-data+json

     { "ietf-restconf:errors" : {
         "error" : [
           {
             "error-type" : "protocol",
             "error-tag" : "operation-failed",
             "error-severity" : "error",
             "error-message" :
               "OTLP traceparent attribute incorrectly formatted",
             "error-info": {
               "ietf-netconf-otlp-context:meta-name" : "tracestate",
               "ietf-netconf-otlp-context:meta-value" :
                 "SomeBadFormatHere",
               "ietf-netconf-otlp-context:error-type" :
                 "ietf-netconf-otlp-context:bad-format"
             }
           }
         ]
       }
     }

# Changes (to be deleted by RFC Editor)

## From version 01 to -02
- Removed markdown formatting of tracestate and traceparent, as toolchain could not handle this properly
- Rearranged text in introduction to include referenes in a more natural order
- Removed several references to "we" and replaced with more neutral language
- Clarified that the YANG modules used by this document is defined by the sibling document for NETCONF

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
