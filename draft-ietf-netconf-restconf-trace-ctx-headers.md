---
docname: draft-ietf-netconf-restconf-trace-ctx-headers-latest
title:  RESTCONF Extension to support Trace Context Headers
abbrev: rc_trace
category: std
date: 2024-11-07

ipr: trust200902
submissiontype: IETF
consensus: true
v: 02
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
  arch: https://mailarchive.ietf.org/arch/browse/netmod/
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
  RFC8341:
  RFC8446:
  RFC8525:

  I-D. draft-ietf-netconf-trace-ctx-extension-02:

  W3C-Trace-Context:
    title: W3C Recommendation on Trace Context
    target: https://www.w3.org/TR/2021/REC-trace-context-1-20211123/
    date: 2021-11-23


--- abstract

This document extends the RESTCONF protocol in order to support trace context propagation as defined by the W3C.

--- middle

# Introduction

Network automation and management systems commonly consist of multiple
sub-systems and together with the network devices they manage, they effectively form a distributed system.  Distributed tracing is a methodology implemented by tracing tools to follow, analyze and debug operations, such as configuration transactions, across multiple distributed systems.  An operation is uniquely identified by a trace-id and through a trace context, carries some metadata about the operation.  Propagating this "trace context" between systems enables forming a coherent view of the entire operation as carried out by all involved systems.

The W3C has defined two HTTP headers (traceparent and tracestate) for context propagation that are useful for distributed systems like the ones defined in {{?RFC8309}}. The goal of this document is to adopt this W3C specification for the RESTCONF protocol.

This document does not define new HTTP extensions but makes those defined in {{W3C-Trace-Context}} optional headers for the RESTCONF protocol.

In {{I-D.draft-ietf-netconf-trace-ctx-extension-02}}, the NETCONF protocol extension is defined and we will re-use several of the YANG and XML objects defined in that document for RESTCONF. Please refer to that document for additional context and example applications.

## Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT","SHOULD","SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.

Additionally, the document utilizes the following abbreviations:

OTLP:
: OpenTelemetry protocol as defined by {{OpenTelemetry}}

M.E.L.T:
: Metrics, Events, Logs and Traces

gNMI:
: gRPC Network Management Interface, as defined by {{gNMI}}

# RESTCONF Extensions

A RESTCONF server SHOULD support trace context traceparent header as defined in {{W3C-Trace-Context}}.

A RESTCONF server SHOULD support trace context tracestate header as defined in {{W3C-Trace-Context}}.

## Error Handling

The RESTCONF server SHOULD follow the "Processing Model for Working with Trace Context" as specified in {{W3C-Trace-Context}}.

If the server rejects the RPC because of the trace context headers, the server MAY return an rpc-error with the following values:

      error-tag:      operation-failed
      error-type:     protocol
      error-severity: error

 Additionally, the error-info tag SHOULD contain a relevant details about the error.

 Finally, the sx:structure defined in {{I-D.draft-ietf-netconf-trace-ctx-extension-02}} SHOULD be present in any error message from the server.

## Trace Context header versionning

This extension refers to the {{W3C-Trace-Context}} trace context capability. The W3C traceparent and trace-state headers include the notion of versions. It would be desirable for a RESTCONF client to be able to discover the one or multiple versions of these headers supported by a server. We would like to achieve this goal avoiding the deffinition of new RESTCONF capabilities for each headers' version.

{{I-D.draft-ietf-netconf-trace-ctx-extension-02}} defines a pair YANG modules that SHOULD be included in the YANG library per {{RFC8525}} of the RESTCONF server supporting the RESTCONF Trace Context extension that will refer to the headers' supported versions. Future updates of this document could include additional YANG modules for new headers' versions.

# Security Considerations

There are no YANG modules specified in this document, even though the functionality described herein relates to the network management protocol RESTCONF [RFC8040].  This is because the only functionality described are additional HTTP headers, and those cannot be described using YANG.There are still some security considerations worth mentioning, however.

The traceparent and tracestate headers make it easier to track the flow of requests and their downstream effect on other systems.  This is indeed the whole point with these headers.  This knowledge could also be of use to bad actors that are working to build a map of the managed network.

The lowest RESTCONF layer is HTTPS, and the mandatory-to-implement secure transport is TLS [RFC8446].

The Network Configuration Access Control Model (NACM) [RFC8341] provides the means to restrict access for particular NETCONF or RESTCONF users to a preconfigured subset of all available NETCONF or RESTCONF protocol operations and content.

All privacy and security consideration in {{W3C-Trace-Context}} apply to this document.

# IANA Considerations

This document has no IANA actions.

# Acknowledgments

The authors would like to acknowledge the valuable implementation feedback from Christian Rennerskog and Per Andersson.  Many thanks to Raul Rivas Felix, Alexander Stoklasa, Luca Relandini and Erwin Vrolijk for their help with the demos regarding integrations.  The help and support from Jean Quilbeuf and Benoît Claise has also been invaluable to this work.

--- back

# Example RESTCONF calls

All examples from {{RFC8040}} Appendix B could be recreated in this seciton by adding the new header described in this document. We selected one example from that document as reference.

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

{{W3C-Trace-Context}} specifies that vendor MAY validate the tracestate header and that invalid headers MAY be discarded. In Section [Error handling](#error-handling), it is stated that servers MAY return an error. Let's assume that is our implementation.

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
             "Context traceparent header incorrectly formatted",
             "error-info": {
               "ietf-trace-context:meta-name" : "tracestate",
               "ietf-trace-context:meta-value" :
               "SomeBadFormatHere",
               "ietf-trace-context:error-type" :
               "ietf-trace-context:bad-format"
             }
           }
         ]
       }
     }

# Changes (to be deleted by RFC Editor)

## From version 01 to 02
- Added WGLC comments
- Changed namespaces and module name

## From version 00 to -01
- Added Security considerations
- Added Acknowledgements
- Added several Normative references
- Added links to latest document on github
- Added RESTCONF example for success and error
- Modified Error Handling to reflect better W3C alignment based on implementation feedback

## From version 00 to draft-ietf-netconf-restconf-trace-ctx-headers-00
- Adopted by NETCONF WG
- Moved repository to NETCONF WG
- Changed build system to use martinthomson's excellent framework
- Ran make fix-lint to remove white space at EOL etc.
- Added this change note. No other content changes

## TODO:
