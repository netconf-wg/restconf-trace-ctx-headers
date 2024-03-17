---
docname: draft-ietf-netconf-restconf-trace-ctx-headers-latest
title:  RESTCONF Extension to support Trace Context Headers
abbrev: rc_trace
category: std
date: 2024-03-17

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
  arch: https://mailarchive.ietf.org/arch/browse/netmod/
  github: TBD
  latest: TBD

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
  RFC8174:
  RFC8525:
  RFC8040:

  I-D.draft-rogaglia-netconf-trace-ctx-extension-03:

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

In {{I-D.draft-rogaglia-netconf-trace-ctx-extension-03}}, the NETCONF protocol extension is defined and we will re-use several of the YANG and XML objects defined in that document for RESTCONF. Please refer to that document for additional context and example applications.

## Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT","SHOULD","SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.

# RESTCONF Extensions

A RESTCONF server SHOULD support trace context traceparent header as defined in {{W3C-Trace-Context}}.

A RESTCONF server SHOULD support trace context tracestate header as defined in {{W3C-Trace-Context}}.

## Errors handling

The RESTCONF server SHOULD follow the "Processing Model for Working with Trace Context" as specified in {{W3C-Trace-Context}}.

If the server rejects the RPC because of the trace context headers values, the server MUST return an rpc-error with the following values:

      error-tag:      operation-failed
      error-type:     protocol
      error-severity: error

 Additionally, the error-info tag SHOULD contain a relevant details about the error.

 Finally, the sx:structure defined in {{I-D.draft-rogaglia-netconf-trace-ctx-extension-03}} SHOULD be present in any error message from the server.

 Example of a badly formated trace context extension using {{RFC8040}} example B.2.1:

      POST /restconf/data/example-jukebox:jukebox/library HTTP/1.1
      Host: example.com
      Content-Type: application/yang-data+json
      traceparent: SomeBadFormatHere
      tracestate: OrSomeBadFormatHere

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
               "ietf-netconf-otlp-context:meta-name" : "traceparent",
               "ietf-netconf-otlp-context:meta-value" :
               "SomeBadFormatHere",
               "ietf-netconf-otlp-context:error-type" :
               "ietf-netconf-otlp-context:bad-format"
             }
           }
         ]
       }
     }

## Trace Context header versionning

This extension refers to the {{W3C-Trace-Context}} trace context capability. The W3C traceparent and trace-state headers include the notion of versions. It would be desirable for a RESTCONF client to be able to discover the one or multiple versions of these headers supported by a server. We would like to achieve this goal avoiding the deffinition of new RESTCONF capabilities for each headers' version.

{{I-D.draft-rogaglia-netconf-trace-ctx-extension-03}} defines a pair YANG modules that SHOULD be included in the YANG library per {{RFC8525}} of the RESTCONF server supporting the RESTCONF Trace Context extension that will refer to the headers' supported versions. Future updates of this document could include additional YANG modules for new headers' versions.

# Security Considerations

TODO Security

# IANA Considerations

This document has no IANA actions.

# Acknowledgments

We would like to acknowledge

--- back

# Example RESTCONF calls

TBD

# Changes (to be deleted by RFC Editor)

## From version 00 to draft-ietf-netconf-restconf-trace-ctx-headers-00
- Adopted by NETCONF WG
- Moved repository to NETCONF WG
- Changed build system to use martinthomson's excellent framework
- Ran make fix-lint to remove white space at EOL etc.
- Added this change note. No other content changes.


# TO DO List (to be deleted by RFC Editor)

- Security Considerations
- Example RESTCONF Calls
- The W3C is working on a draft document to introduce the concept of "baggage" that we expect part of a future draft for NETCONF and RESTCONF
