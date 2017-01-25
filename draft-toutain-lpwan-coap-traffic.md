---
stand_alone: true
ipr: trust200902
docname: draft-toutain-lpwan-coap-traffic-latest
cat: info
pi:
  symrefs: 'yes'
  sortrefs: 'yes'
  strict: 'yes'
  compact: 'yes'
title: CoAP Traffic
abbrev: CoAP Traffic
date: 2016-04
author:
- ins: A. Minaburo
  name: Ana Minaburo
  org: Acklio
  street: 2bis rue de la Chataigneraie
  city: 35510 Cesson-Sevigne Cedex
  country: France
  email: ana@ackl.io
- ins: L. Toutain
  name: Laurent Toutain
  org: Institut Mines Telecom Atlantique
  street:
  - 2 rue de la Chataigneraie
  - CS 17607
  city: 35576 Cesson-Sevigne Cedex
  country: France
  email: Laurent.Toutain@imt-atlantique.fr
normative:
  rfc7967:
  I-D.vanderstok-core-comi:

--- abstract


This document describes different CoAP scenarios for the SCHC compression.
It goes
from the simplest exchange without acknowledgments to a basic study of the
CoMI traffic.

--- middle

# Introduction

This document describes different CoAP scenarios for the SCHC compression.
It goes
from the simplest exchange without acknowledgments to a basic study of the
CoMI traffic.
These scenarios currently do not cover cases where encryption is used (COSE,
OSCOAP,...).


# scenario 1 - Unidirectional traffic

## CoAP POST without Acknowledgement from thing {#FromThingPOSTnoAck}

The thing sends a CoAP POST/PUT request without acknowledgment using
CoAP NON message and no response option {{RFC7967}}. This is a common traffic
in a LPWAN network to minimize the downlink.

~~~~
 thing                          LPWAN SCHC                   CoAP Server
|                                |                                |
|                                |   NON POST MID=0x00AB          |
|                                |   Token = 0x11                 |
|                                |   Path = /elm1/elm2            |
|                                |   Content-format = val         |
|                                |   NoResponse = 0|2|8|16        |
|     rule-id value              |   Value                        |
|------------------------------->|                                |
|                                | ------------------------------>|



~~~~
{: #Fig-Thing-Simple-POST title='POST with no ACK'}
Compression Objectives:

* do not send version, type, token length, code because they are defined in
  the rule.

* compress Message ID and Token.

* do not sent options because they are defined in the rule.

* send value
\*\*\* NOTE: The Mid may not be sent, since no acknowledgement is expected.
Nevertheless
several copies of the same message will not be detected by the receiver which
will view
them as several requests. This can be solved at L2 if the technology sends
the frame with
unique value.


## Scenario 2 - CoAP POST without Acknowledgement to thing

Same as {{FromThingPOSTnoAck}} but request comes from network to thing.
Selected values for Mid, token need to be control to allow a better compression
rate. A
CoAP proxy is need to normalize these values.

~~~~
thing                          LPWAN SCHC                  CoAP Server
|                                |                                |
|                                |   NON POST MID=0xCDAB          |
|                                |   Token = 0x11223344           |
|                                |   Path = /elm1/elm2            |
|                                |   Content-format = val         |
|                                |   NoResponse = 0|2|8|16        |
|                                |   Value                        |
|    rule-id Value               |<-------------------------------|
|<-------------------------------|                                |
|                                |                                |



~~~~
{: #Fig-Network-simple-POST title='POST with no ACK'}
Compression Objectives:

* do not send version, type, token length, code because they are defined in
  the rule.

* reduce value size and compress Message ID and Token.

* do not sent options because they are defined in the rule

* send value




# bi-directional traffic

## Scenario 3 - CoAP ack

Same as {{FromThingPOSTnoAck}} but the network acknowledge the CoAP message.

~~~~
thing                          LPWAN SCHC                   CoAP Server
|                                |                                |
|                                |   CON POST MID=0x00AB          |
|                                |   Token = 0x11                 |
|                                |   Path = /elm1/elm2            |
|                                |   Content-format = val         |
|            TDB                 |   NoResponse = 0|2|8|16        |
|------------------------------->|   Value                        |
|                                |------------------------------->|
|                                |                                |
|                                |   ACK 0.00 MID=0x00AB          |
|            TBD                 |<-------------------------------|
|<-------------------------------|



~~~~
{: #Fig-Thing-Simple-POST-CoAPAcked title='POST with no ACK'}
Objectives:

* Thing

  * do not send version, token length.

  * compress type, Message ID, Token and code.

  * do not sent options.

  * send value


* Network

  * compress type and code.




## Scenario 4 - REST ack {#FromThingPOSTwithAck}

Same as {{FromThingPOSTnoAck}} but the thing wait for an acknowledgement
at REST level. Response code can be 2.04, 2.01, 2.02, 4.0Y, 5.0Y.

~~~~
thing                          LPWAN SCHC                   CoAP Server
|                                |                                |
|                                |   NON POST MID=0x00AB          |
|                                |   Token = 0x11                 |
|                                |   Path = /elm1/elm2            |
|------------------------------->|   Value                        |
|            TBD                 |------------------------------->|
|                                |                                |
|                                |   NON X.YY MID=0x1234          |
|                                |   Token = 0x11                 |
|            TBD                 |<-------------------------------|
|<-------------------------------|                                |



~~~~
{: #Fig-Thing-Simple-POST-Acked title='POST with no ACK'}
Objectives:

* Thing

  * do not send version, token length.

  * compress type, Message ID and Token.

  * compress code.

  * do not sent options

  * send value


* Network

  * do not send version, token length.

  * compress type, Message ID and token.

  * reduce MID size in response and compress it.




## Scenario 5 - GET

Same as {{FromThingPOSTwithAck}} but GET request and value in response..

~~~~
thing                          LPWAN SCHC                     CoAP Server
|                                |                                |
|                                |   NON GET MID=0x00AB           |
|                                |   Token = 0x11                 |
|          TBD                   |   Accept = val                 |
|------------------------------->|   Path = /elm1/elm2            |
|                                |------------------------------->|
|                                |                                |
|                                |   NON 2.05 MID=0x1234          |
|                                |   Token = 0x11                 |
|                                |   Content-format = val         |
|                                |   Value                        |
|          TDB                   |<-------------------------------|
|<-------------------------------|                                |



~~~~
{: #Fig-Thing-Simple-GET title='POST with no ACK'}
Objectives:

* Thing

  * do not send version, type, token length.

  * compress Message ID and Token.

  * compress code.

  * do not sent options


* Network

  * reduce MID size in response and compress it.

  * compress code.

  * send token.

  * send value





# CoMI

{{I-D.vanderstok-core-comi}} defines the different exchanges using CoMI.
The path /c gives access to the CoMI module.

## GET

With the GET method, /c is followed by the SID integer value coded in base64.
A query
parameter allows to specify a particular instance by sending the YANG keys.
If no value
is given then the full structure is returned.
For instance (taken from the draft):

* GET /c/a3

* GET /c/Bf4?k="eth0"

* GET /c


Objective:

* allow to compress partially the path and send only the SID value on the radio
  link.



## FETCH

The FETCH method uses a CBOR structure sent in the payload instead of the
URI.

~~~~
FETCH /c/ Content-Format (application/YANG-fetch+cbor)
       <CBOR array of instance identifiers>
~~~~
{: #FETCHStructure title='FETCH Structure'}


The CBOR structure is an array containing either CBOR Integer for SID or
array with a SID
and a list of filtering parameters.

~~~~
FETCH /c Content-Format (application/YANG-fetch+cbor)
   [ 1719,              # ID 1719
     [-186, "eth0"]   # ID 1533 with name = "eth0"
   ]
~~~~
{: #Simple-FETCH title='FETCH with CBOR parameters'}


For the compression point of view, Fetch is a new code value 0.05. But to
send
dynamically SIDs and keys, the CBOR structure must be understood by SCHC.


## PUT/POST

PUT allows to remplace or create a data resource instance, POST always add
a data
resource instance. Draft imposes to use a confirmable CoAP message (which
may not
needed since there is REST confirmation). POST URI contains
only the SID value and PUT can add a query parameter with keys.


## iPATCH

As for FETCH, the URI is static and the parameters are sent in a CBOR array
containing the SID and then values.

~~~~
   iPATCH /c Content-Format(application/YANG-patch+cbor)
   [
     [1533, "eth0"] ,                # interface (ID = 1533)
       {
         +4 : "eth0",                # name (ID 1537)
         +1 : "Ethernet adaptor",    # description (ID 1534)
         +5 : 1179,                  # type (ID 1538),
                                     # identity ethernetCsmacd
         +2 : true                   # enabled (ID 1535)
       }
     +203 , 60          # timezone-utc-offset (delta = 1736 - 1533)
   ]

   2.04 Changed
~~~~
{: #Simple-iPATCH title='iPATCH with CBOR parameters'}



## DELETE

If GET and PUT/POST can be replaced by FETCH and iPATCH to have a more compact
representation of SID and keys, DELETE imposes to put these values in the
URI.


## Examples

 {{Fig-generic-rule-tree}} gives the YANG module tree for SCHC. Two keys are
defined: one for the rule-id and the other for the position (field-id).

~~~~
module: ietf-lpwan-compression
    +--rw compression-context
       +--rw context-rules* [rule-id]
          +--rw rule-id        uint8
          +--rw rule-fields* [position]
             +--rw name?                                       string
             +--rw position                                    uint8
             +--rw target-value?                               lpwan-types
             +--rw matching-operator?                          matching-operator-type
             +--rw matching-operator-value?                    lpwan-types
             +--rw compression-decompression-function?         compression-decompression-function-type
             +--rw compression-decompression-function-value?   lpwan-types


~~~~
{: #Fig-generic-rule-tree title='Generic module tree'}
 {{Fig-generic-rule-sid}} gives some SID value that may be applied
to this YANG module. Note that Matching Operators and Compression/Decompression
Functions are identified by a SID value.

~~~~
SID        Assigned to
---------  --------------------------------------------------
1000       Module ietf-lpwan-compression
1001       identity /compression-decompression-function
1002       identity /compression-decompression-function/cdf-compute-ipv6-length
1003       identity /compression-decompression-function/cdf-compute-udp-checksum
1004       identity /compression-decompression-function/cdf-compute-udp-length
1005       identity /compression-decompression-function/cdf-esiid-did
1006       identity /compression-decompression-function/cdf-laiid-did
1007       identity /compression-decompression-function/cdf-lsb
1008       identity /compression-decompression-function/cdf-not-sent
1009       identity /compression-decompression-function/cdf-value-sent
1010       identity /matching-operator
1011       identity /matching-operator/mo-equal
1012       identity /matching-operator/mo-ignore
1013       identity /matching-operator/mo-msb
1014       node /compression-context
1015       node /compression-context/context-rules
1016       node /compression-context/context-rules/rule-fields
1017       node /compression-context/context-rules/rule-fields/compression-decompression-function
1018       node /compression-context/context-rules/rule-fields/compression-decompression-function-value
1019       node /compression-context/context-rules/rule-fields/matching-operator
1020       node /compression-context/context-rules/rule-fields/matching-operator-value
1021       node /compression-context/context-rules/rule-fields/name
1022       node /compression-context/context-rules/rule-fields/position
1023       node /compression-context/context-rules/rule-fields/target-value
1024       node /compression-context/context-rules/rule-id

File ietf-lpwan-compression@2016-11-01.sid created
Number of SIDs available : 200
Number of SIDs assigned : 25

~~~~
{: #Fig-generic-rule-sid title='Example of SID allocation'}
Some simple scenarii where SCHC rules can be modified:

* a thing selects a dynamic port number for a flow and informs the other end
  of this value.

* a thing informs the other end of the destination IPv6 address.

* Network informs the thing of the IPv6 prefix
 {{Fig-iPATCH-example}} gives an example of a simple request where:

* field-SID contains the SID value identifying the Target Value (1023 in {{Fig-generic-rule-sid}}  ),

* rule-id is the rule to be modified

* field-pos is the value indicating that it is respectively a ESport, or a
  prefix


~~~~
iPATCH /c Content-Format(application/YANG-patch+cbor)
[
  [field-SID, rule-id, field-pos], value
]

~~~~
{: #Fig-iPATCH-example title='YANG definition of the IPv6 UDP compression'}


To set a port number for Rule 10 a CoMI iPTACH is sent. The rule 1 is
used for that CoMI request and contains the Path, the content-format values
and also the
field-SID for Target Value (1023).

~~~~
  +----------------+------------------------+----------------+-----------------+
  | Field          |   Function             | Target Value   | Sent compressed |
  +----------------+------------------------+----------------+-----------------+
  |CoAP version    | not-sent               | 1              |                 |
  |CoAP Type       | not-sent               | CON*           | t               |
  |CoAP TKL        | compute-token-length   |                |                 |
  |CoAP Code       | map-code               | mapping table  |  CC             |
  |CoAP MID        | remapping              | 7 bits         |    MM           |
  |CoAP Token      | remapping              | 8 bits         |       TTT       |
  |CoAP Path       | not-sent               | /c             |                 |
  |CoAP content-F  | not-sent               | YANG-patch+cbor|                 |
  +----------------+------------------------+----------------+-----------------+
  * draft mandate a Confirmable message
~~~~
{: #Fig-CoAP-header-2 title='CoAP Context to compress header with token'}
Where t defines the request type (0: CON, 1: ACK), CC defines the code
(00: iPTACH, 01: 2.04, 10: ????, 11: ????). Message id is
remapped in 2 bits which allows 4 messages  simultaneously in the air and
Token to
3 bits which allows 8 simultaneous pending requests.

The objective will be to sent on the radio link a message containing:

* rule-id for CoMI message : 0x01

* CoAP header : 0xXX (1 byte)

* CBOR Rule-id : 0x0A (10 in the example) (1 byte)

* CBOR position to identify ESport : 0xPPPPPPPP (4 bytes)

* CBOR port number value : 0xPPPP (2 bytes)
Therefore 9 bytes are sent on the LPWAN radio link.



--- back
