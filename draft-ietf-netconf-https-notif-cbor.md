---
title: "CBOR Encoding for HTTPS-based YANG Notifications Transport"
abbrev: "https-notif-cbor"
category: std

docname: draft-ietf-netconf-https-notif-cbor-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: ""
workgroup: "Network Configuration"
keyword:
 - cbor
 - https
 - yang

venue:
  group: "Network Configuration"
  type: ""
  mail: "netconf@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/netconf/"
  github: "MeherRushi/draft-ietf-netconf-https-notif-cbor"
  latest: "https://MeherRushi.github.io/draft-ietf-netconf-https-notif-cbor/draft-ietf-netconf-https-notif-cbor.html"

author:
  - name: Bharadwaja Meherrushi Chittapragada
    email: meherrushi2@gmail.com
    ins: B. M. Chittapragada
    org: National Institute of Technology Karnataka

  - name: Siddharth Bhat
    email: siddharth.bhat10@gmail.com
    ins: S. Bhat
    org: National Institute of Technology Karnataka

  - name: Vartika T Rao
    email: vartikatrao@gmail.com
    ins: V. T. Rao
    org: National Institute of Technology Karnataka

  - name: Hayyan Arshad
    email: hayyanhamnah@gmail.com
    ins: H. Arshad
    org: National Institute of Technology Karnataka

  - name: Mohit P. Tahiliani
    email: tahiliani@nitk.edu.in
    ins: M. P. Tahiliani
    org: National Institute of Technology Karnataka

normative:
 I-D.draft-ietf-netconf-https-notif:
 RFC8949:
 RFC9254:

informative:
 RFC3553:


--- abstract

This document extends {{!I-D.draft-ietf-netconf-https-notif}} by introducing CBOR encoding for YANG notifications over HTTPS transport in addition to the existing JSON and XML encoding schemes.


--- middle

# Introduction

~~~~ quote
CBOR offers an efficient and compact representation of YANG.
~~~~

This document introduces a CBOR encoding scheme for event notifications over HTTPS by using the framework proposed in {{!I-D.draft-ietf-netconf-https-notif}} which supports transfer of YANG notifications over HTTPS using JSON and XML encoding schemes.


In {{!I-D.draft-ietf-netconf-https-notif}}, the capabilities HTTP-target resource allows a publisher to retrieve supported encoding formats via GET requests, while the relay-notification resource enables the publisher to send YANG notifications via POST requests. These requests and responses use different content types based on the selected encoding scheme. This document defines support for CBOR encoding, in addition to the JSON and XML encodings defined in {{!I-D.draft-ietf-netconf-https-notif}}.

Examples of the GET and POST request and reply encoded in CBOR are also provided.


# Terminology

This document uses the following terms defined in Sections 2, 3, and 4 of {{!I-D.draft-ietf-netconf-https-notif}}:

   - Capabilities Resource

   - Relay-Notification

   - Event Notification

The following term(s) are defined in Subscription to YANG Notifications {{!RFC8639}}:

   - Publisher

   - Receiver

   - Subscribed Notifications

The following term(s) are defined in Encoding of Data Modeled with YANG in the Concise Binary Object Representation (CBOR) {{!RFC9254}}:

   - Diagnostic Notation

   - YANG Schema Item iDentifier (or "YANG SID" or simply "SID"): 63-bit unsigned integer used to identify different YANG items.



# CBOR Encoding of the notification(s)

YANG notifications can be encoded in CBOR using Names or SIDs in keys. Notifications encoded using names are similar to JSON encoding as defined in Section 3.4 and 4.3 of {{!I-D.draft-ietf-netconf-https-notif}}. Notification encoded using YANG-SIDs replaces the names of the keys of the CBOR encoded message with a 63 bit unsigned integer.  In this case, the term 'SID' is defined in Section 3.2 of {{!RFC9254}}, and the keys of the encoded data use SID value as mentioned in 4.3.2 of this document.

The "application/yang-data+cbor" media type used throughout this document is defined in Section 9 of {{!RFC9254}}. That registration also defines the "id" parameter: "application/yang-data+cbor; id=name" for the name-based keys described in this document, and "application/yang-data+cbor; id=sid" for the SID-based keys.

## Capabilities Request

The publisher sends a request to the receiver to learn its capabilities. In the below example, the “Accept” states that the publisher wants to receive the capabilities response in CBOR but if not supported then in XML or JSON in that order.

~~~ http-request
GET /some/path/capabilities HTTP/1.1
   Host: example.com
   Accept: application/yang-data+cbor; id=name, application/yang-data+xml;q=0.9, application/yang-data+json;q=0.5
~~~

## Capabilities Response

 If the receiver is able to reply using “application/yang-data+cbor” and assuming it is only capable of receiving CBOR encoded messages the response would look like this

### CBOR using names as keys (receiver accepts CBOR only)

~~~ http-message
   HTTP/1.1 200 OK
   Date: Tue, 4 March 2025 20:33:30 GMT
   Server: example-server
   Cache-Control: no-cache
   Content-Type: application/yang-data+cbor; id=name
~~~

Diagnostic Notation:

~~~
   {
   "ietf-https-notif-transport:receiver-capabilities": {
     "receiver-capability": [
       "urn:ietf:params:yang-notif:https-capability:encoding:cbor"
        ]
      }
   }
~~~

CBOR Encoding:

~~~
A1                                      # map(1)
   78 30                                # text(48)
      696574662D68747470732D6E6F7469662D7472616E73706F72743A72656365697665722D6361706162696C6974696573 # "ietf-https-notif-transport:receiver-capabilities"
   A1                                   # map(1)
      73                                # text(19)
         72656365697665722D6361706162696C697479 # "receiver-capability"
      81                                # array(1)
         78 39                          # text(57)
            75726E3A696574663A706172616D733A79616E672D6E6F7469663A68747470732D6361706162696C6974793A656E636F64696E673A63626F72 # "urn:ietf:params:yang-notif:https-capability:encoding:cbor"
~~~

If the receiver is able to reply using “application/yang-data+cbor” and assuming it is not capable of receiving cbor, but can receive both json and xml notifications:

### CBOR using names as keys (receiver accepts JSON and XML)

~~~ http-message
   HTTP/1.1 200 OK
   Date: Tue, 4 March 2025 20:33:30 GMT
   Server: example-server
   Cache-Control: no-cache
   Content-Type: application/yang-data+cbor; id=name
~~~

Diagnostic Notation:

~~~
   {
   "ietf-https-notif-transport:receiver-capabilities": {
     "receiver-capability": [
       "urn:ietf:params:yang-notif:https-capability:encoding:json",
       "urn:ietf:params:yang-notif:https-capability:encoding:xml"
        ]
      }
   }
~~~

CBOR Encoding:

~~~
A1                                      # map(1)
   78 30                                # text(48)
      696574662D68747470732D6E6F7469662D7472616E73706F72743A72656365697665722D6361706162696C6974696573 # "ietf-https-notif-transport:receiver-capabilities"
   A1                                   # map(1)
      73                                # text(19)
         72656365697665722D6361706162696C697479 # "receiver-capability"
      82                                # array(2)
         78 39                          # text(57)
            75726E3A696574663A706172616D733A79616E672D6E6F7469663A68747470732D6361706162696C6974793A656E636F64696E673A6A736F6E # "urn:ietf:params:yang-notif:https-capability:encoding:json"
         78 38                          # text(56)
            75726E3A696574663A706172616D733A79616E672D6E6F7469663A68747470732D6361706162696C6974793A656E636F64696E673A786D6C # "urn:ietf:params:yang-notif:https-capability:encoding:xml"
~~~

If the receiver cannot encode the capabilities response in "application/yang-data+cbor" but is itself only able to receive CBOR-encoded notifications, it replies using JSON (or XML) while still advertising the CBOR capability:

~~~ http-message
   HTTP/1.1 200 OK
   Date: Tue, 4 March 2025 20:33:30 GMT
   Server: example-server
   Cache-Control: no-cache
   Content-Type: application/yang-data+json

   {
   "ietf-https-notif-transport:receiver-capabilities": {
     "receiver-capability": [
       "urn:ietf:params:yang-notif:https-capability:encoding:cbor"
        ]
      }
   }
~~~

##  Relay Notification request

The publisher sends an HTTP POST request to the "relay-notification" resource on the receiver with the "Content-Type" header set to "application/yang-data+cbor" in case the receiver is CBOR capable and a body containing the notification encoded in CBOR. The "id" parameter of the media type indicates whether the keys are encoded as names ("id=name") or as SIDs ("id=sid").

### CBOR encoding using names as keys

~~~ http-request
POST /some/path/relay-notification HTTP/1.1
   Host: example.com
   Content-Type: application/yang-data+cbor; id=name
~~~

Diagnostic Notation:

~~~ http-response
   {
     "ietf-https-notif:notification": {
       "eventTime": "2013-12-21T00:01:00Z",
       "example-mod:event" : {
         "event-class" : "fault",
         "reporting-entity" : { "card" : "Ethernet0" },
         "severity" : "major"
       }
     }
   }
~~~

CBOR Encoding:

~~~
A1                                      # map(1)
   78 1D                                # text(29)
      696574662D68747470732D6E6F7469663A6E6F74696669636174696F6E # "ietf-https-notif:notification"
   A2                                   # map(2)
      69                                # text(9)
         6576656E7454696D65             # "eventTime"
      74                                # text(20)
         323031332D31322D32315430303A30313A30305A # "2013-12-21T00:01:00Z"
      71                                # text(17)
         6578616D706C652D6D6F643A6576656E74 # "example-mod:event"
      A3                                # map(3)
         68                             # text(8)
            7365766572697479            # "severity"
         65                             # text(5)
            6D616A6F72                  # "major"
         6B                             # text(11)
            6576656E742D636C617373      # "event-class"
         65                             # text(5)
            6661756C74                  # "fault"
         70                             # text(16)
            7265706F7274696E672D656E74697479 # "reporting-entity"
         A1                             # map(1)
            64                          # text(4)
               63617264                 # "card"
            69                          # text(9)
               45746865726E657430       # "Ethernet0"
~~~

### CBOR encoding using SIDs as keys

~~~ http-request
POST /some/path/relay-notification HTTP/1.1
   Host: example.com
   Content-Type: application/yang-data+cbor; id=sid
~~~

Diagnostic Notation:

~~~
   {
     2601: {                        / ietf-https-notif:notification (SID 2601) /
       1: "2013-12-21T00:01:00Z",   / eventTime (SID 2602) /
       57400: {                     / example-mod:event (SID 60001) /
         1: "fault",                / event-class (SID 60002) /
         2: {                       / reporting-entity (SID 60003) /
           1: "Ethernet0"           / card (SID 60004) /
         },
         4: "major"                 / severity (SID 60005) /
       }
     }
   }
~~~

The above assumes that the YANG modules for the notification envelope and for the event have corresponding .sid files with the following entries:

~~~
"item": [
      {
        "namespace": "module",
        "identifier": "ietf-https-notif",
        "sid": "2600"
      },
      {
        "namespace": "data",
        "identifier": "/ietf-https-notif:notification",
        "sid": "2601"
      },
      {
        "namespace": "data",
        "identifier": "/ietf-https-notif:notification/eventTime",
        "sid": "2602"
      },
      {
        "namespace": "module",
        "identifier": "example-mod",
        "sid": "60000"
      },
      {
        "namespace": "data",
        "identifier": "/example-mod:event",
        "sid": "60001"
      },
      {
        "namespace": "data",
        "identifier": "/example-mod:event/event-class",
        "sid": "60002"
      },
      {
        "namespace": "data",
        "identifier": "/example-mod:event/reporting-entity",
        "sid": "60003"
      },
      {
        "namespace": "data",
        "identifier": "/example-mod:event/reporting-entity/card",
        "sid": "60004"
      },
      {
        "namespace": "data",
        "identifier": "/example-mod:event/severity",
        "sid": "60005"
      }
    ]
~~~

Keys are encoded as SID deltas relative to the reference SID of the enclosing map, as defined in Section 3.2 of {{!RFC9254}}. Because "example-mod:event" is defined in a different module than its parent, its delta is large (60001 - 2601 = 57400); alternatively, it MAY be encoded as an absolute SID using CBOR tag 47.

CBOR Encoding:

~~~
A1                                      # map(1)
   19 0A29                              # unsigned(2601)
   A2                                   # map(2)
      01                                # unsigned(1)
      74                                # text(20)
         323031332D31322D32315430303A30313A30305A # "2013-12-21T00:01:00Z"
      19 E038                           # unsigned(57400)
      A3                                # map(3)
         01                             # unsigned(1)
         65                             # text(5)
            6661756C74                  # "fault"
         02                             # unsigned(2)
         A1                             # map(1)
            01                          # unsigned(1)
            69                          # text(9)
               45746865726E657430       # "Ethernet0"
         04                             # unsigned(4)
         65                             # text(5)
            6D616A6F72                  # "major"
~~~

## Relay Notification Response

The response on success SHOULD be from the 2XX class of codes. In case of corrupted or malformed event, the response SHOULD be an appropriate HTTP error response.

## Legacy RFC 5277 Notifications

{{!I-D.draft-ietf-netconf-https-notif}} defines optional support for sending legacy notifications, as defined in NETCONF Event Notifications {{!RFC5277}}, using the "application/xml" media type. This legacy mode exists for interoperability with publishers that are not aware of YANG media types; a receiver advertises support for it using the "urn:ietf:params:yang-notif:https-capability:rfc5277-notif" capability.

The CBOR encoding defined in this document applies to YANG-modeled notifications, including the eventTime envelope, and is advertised using the "urn:ietf:params:yang-notif:https-capability:encoding:cbor" capability registered by this document. Because {{!I-D.draft-ietf-netconf-https-notif}} defines the legacy RFC 5277 mode as XML-only, this document does not define a CBOR variant of it. A publisher that is able to produce CBOR is YANG-aware and therefore uses the CBOR encoding described here rather than the legacy mode.

## Implementation Status

This section records the status of known implementations of the specification defined by this document at the time of posting. The information is provided to assist the IETF in evaluating the maturity and implementability of the specification. This section will be removed prior to publication as an RFC.

## Implementation: HTTPS Notification CBOR Draft Implementation
- *Organization*:
  National Institute of Technology Karnataka

- *Implementation Name / Web Page*:
  HTTPS Notification CBOR Draft Implementation
  [https://github.com/MeherRushi/https-notif-draft-impl](https://github.com/MeherRushi/https-notif-draft-impl)

- *Description*:
  This implementation provides a Python-based prototype of the mechanism defined in this document for transporting YANG notifications over HTTPS using JSON, XML and CBOR encoding. It supports name-based CBOR encoding and includes basic publisher and receiver roles to demonstrate end-to-end message exchange.

- *Maturity Level*:  Prototype

- *Coverage*:
  - Capabilities discovery via HTTP GET to /capabilities
  - Event publication via HTTP POST to /relay-notification
  - Support for name-based CBOR encoding as described in this document


- *Version Compatibility*:
  The implementation is based on draft-ietf-netconf-https-notif-16 and draft-ietf-netconf-https-notif-cbor-00.

- *Licensing*:
  Freely distributable under an MIT-style license.

- *Implementation Experience*:
  - Developed and demonstrated at IETF 121 and 122 Hackathon.
  - Worked toward enabling CBOR encoding in the libyang library as part of the hackathon effort ([slides](https://datatracker.ietf.org/meeting/123/materials/slides-123-hackathon-sessd-adding-cbor-support-in-libyang-00)).
  - Evaluated CBOR efficiency compared to JSON and XML in constrained environments.
  - Built tooling to simulate and measure notification transfer behavior over varying network conditions.
  - Diagnostic encoding examples used for validation of CBOR structures.

- *Contact Information*:
  - Bharadwaja Meherrushi Chittapragada (meher.211cs216@nitk.edu.in)
  - Vartika T Rao (vartikatrao.211it077@nitk.edu.in)
  - Siddharth Bhat (sidbhat.211ee151@nitk.edu.in)
  - Hayyan Arshad (hayyanarshad.211cs222@nitk.edu.in)

- *Last Updated*:
  July 24, 2025


# Security Considerations

Addition of the CBOR encoding introduces no specific security exposures or risks other than the ones mentioned in {{!RFC9254}} and {{!I-D.draft-ietf-netconf-https-notif}} (An HTTPS-based Transport for YANG Notifications).

# IANA Considerations

This document requests that IANA include an additional entry in the “Capabilities for HTTPS Notification Receivers” registry, defined in {{!I-D.draft-ietf-netconf-https-notif}}. The following entry is added:

~~~
Record:
   URN:         urn:ietf:params:yang-notif:https-capability:encoding:cbor
   Reference:   RFC XXXX:An HTTPS-based Transport for YANG Notifications
   Description: Identifies support for CBOR-encoded notifications.
~~~

--- back

# Acknowledgments
{:numbered="false"}

The authors acknowledge the support of Kent Watsen and Mahesh Jethanandani, the authors of {{!I-D.draft-ietf-netconf-https-notif}} for their guidance and support provided to draft this document.
