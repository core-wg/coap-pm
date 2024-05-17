---
v: 3
docname: draft-ietf-core-coap-pm-latest
cat: std
stream: IETF

pi:
  strict: 'yes'
  comments: 'no'
  inline: 'no'
  editing: 'no'
  toc: 'yes'
  tocompact: 'yes'
  tocdepth: '3'
  symrefs: 'yes'
  sortrefs: 'yes'
  compact: 'yes'
  subcompact: 'no'

title: Constrained Application Protocol (CoAP) Performance Measurement Option
abbrev: COAP PM
area: Applications and Real-Time
wg: CORE Working Group
date: 2024

author:
- ins: G. Fioccola
  name: Giuseppe Fioccola
  org: Huawei
  street: Palazzo Verrocchio, Centro Direzionale Milano 2
  city: Segrate (Milan)
  code: '20054'
  region: ''
  country: Italy
  email: giuseppe.fioccola@huawei.com

- ins: T. Zhou
  name: Tianran Zhou
  org: Huawei
  street: 156 Beiqing Rd.
  city: Beijing
  code: '100095'
  region: ''
  country: China
  email: zhoutianran@huawei.com

- ins: M. Nilo
  name: Massimo Nilo
  org: Telecom Italia
  street: Via Reiss Romoli, 274
  city: Torino
  region: ''
  code: '10148'
  country: Italy
  email: massimo.nilo@telecomitalia.it

- ins: F. Milan
  name: Fabrizio Milan
  org: Telecom Italia
  street: Via Reiss Romoli, 274
  city: Torino
  region: ''
  code: '10148'
  country: Italy
  email: fabrizio.milan@telecomitalia.it

- ins: F. Bulgarella
  name: Fabio Bulgarella
  org: Telecom Italia
  street: Via Reiss Romoli, 274
  city: Torino
  region: ''
  code: '10148'
  country: Italy
  email: fabio.bulgarella@guest.telecomitalia.it

contributor:
- name: Mauro Cociglio
  org: Telecom Italia
  email: mauro.cociglio@outlook.com

normative:
  RFC7252:
  RFC8613:
  RFC9341:
informative:
  RFC6347:
  RFC9506:
  RFC9000:
  RFC9312:
  RFC7641:
  I-D.ietf-core-oscore-capable-proxies:

--- abstract


This document specifies a method for the Performance Measurement of the
Constrained Application Protocol (CoAP). A new CoAP option is defined
in order to enable network telemetry both end-to-end and hop-by-hop.
The endpoints cooperate by marking and, possibly, mirroring information
on the round-trip connection.

--- middle

# Introduction

In the CoAP protocol {{RFC7252}}, reliability is provided by marking a message
as Confirmable (CON), as to be retransmitted if not acknowledged by an ACK
message.
A message that does not require reliable transmission can be sent as a Non-confirmable
message (NON).

In case of CoAP reliable mode, Message IDs and ACKs could potentially be
used to measure Round-Trip Time (RTT) and losses.
But it can be resource-consuming for constrained nodes since they have to
look at Message IDs and take timestamps.
These operations are expensive in terms of resources. In case of CoAP unreliable
mode, there is no ACK and, consequently, it is not possible to measure RTT and losses.

Thus, there is no easy way to measure the performance metrics in a CoAP environment
including resource-constrained nodes.
And it is in any case limited to RTT and end-to-end losses.

A mechanism to measure both end-to-end and hop-by-hop performance in CoAP
can be useful to verify that the operational requirements are met, but it should be
a simple mechanism for network diagnostic to be developed on constrained nodes
requiring just a minimal amount of collaboration from the endpoints.

{{RFC9506}} describes the methodologies for Explicit Flow Measurement (EFM).
The EFM techniques employ few marking bits, inside the header of each packet,
for loss and delay measurement.
These are relevant for encrypted protocols, e.g. QUIC {{RFC9000}}, where there are only
few bits available in the non-encrypted header in order to allow passive
performance metrics from an on-path probe.
These methodologies could potentially be used and extended in CoAP.

{{RFC9506}} defines different combinations of bits.
Such flexibility is convenient when using a protocol with a limited number of
eligible bits to exploit, e.g., QUIC.
Different alternatives have been proposed, but all these methods together
imply complex algorithms that do not apply well to the CoAP environment.

This document aims to create an easy way to allow performance measurement
for CoAP, by defining a new option, called Performance Measurement (PM) CoAP Option.
The CoAP performance metrics (e.g. RTT and losses) allow to perform
both end-to-end and hop-by-hop measurements and can be useful for an operator
or an enterprise that is managing a constrained, low-power and lossy network.

This document ultimately is intended to be published as a standards-track RFC.
Its current stage of development, it is complete and stable enough to
be used as a basis for experiments, the evaluation and continuation of
which will lead to further evolution towards the intended
standards-track document.

## Requirements Language

{::boilerplate bcp14-tagged}


# Performance Measurement methods for CoAP {#PMmethods}

The approach proposed in this document relies on a new Performance Measurement
(PM) Option for CoAP {{RFC7252}}.
This new option is defined in {{PMO}} and carries PM bits.

The PM bits that are included in the Option are:

* sQuare bit (Q bit), based on {{RFC9341}} and further described in {{RFC9506}};

* Spin bit (S bit), described in {{RFC9312}} and included as optional bit in {{RFC9000}};

* Loss and Delay event information for further usage.


A requirement to enable PM methods in CoAP environment is that the methodologies
and the algorithm needs to be kept simple.
For this reason, the idea is to re-apply only the S bit and Q bit.

Thus, the advantages of using the CoAP PM Option are:

1. Simplification because it is not needed to read Message IDs, indeed there
   is a well-defined sQuare wave, and it is not necessary to store timestamps,
   since the duration of the Spin Bit period is equal to RTT.
2. Enabling easy on-path probe (proxy, gateway) metrics.


## sQuare bit and Spin bit {#QSbits}

The sQuare bit algorithm consists of creating square waves of a known length
(e.g. 64 packets). Each communicating endpoint can set
the Q bit and toggle its value each time a fixed number of messages have
been sent. The number of packets can be easily recognized
and packet loss can be measured.

The Spin bit algorithm consists of creating a square wave signal on the data
flow, using a bit, whose length is equal to RTT.
The Spin bit causes one bit to ‘spin’, generating one edge (a transition
from 0 to 1 or from 1 to 0) once per end-to-end RTT.
When sending a new request, a client sets the spin bit to the opposite value
it had in the immediately previously sent request to the same server.
Then, the server echoes the same value of the spin bit of the request
in the spin bit of the response.
Therefore the Spin bit is set by both sides to the same value for as long
as one round trip lasts and then it toggles the value.

An on-path probe can read the Q bit and S bit signals and perform the measurements.
All the possible measurements (end-to-end, hop-by-hop) that are enabled by
the Q bit and S bit are detailed in {{RFC9506}}.


## Combined sQuare bit {#Cbit}

The synergy between S bit and Q bit is also possible. As described above,
the length of the Q bit square waves is fixed (e.g. a predefined number of packets)
in this way each endpoint can detect a packet loss if it receives less packets than
the other endpoint has sent.
It is possible to potentiate the Q bit signal by incorporating RTT information
as well. This implies a little modification to the algorithm of the Q bit
that could also be used alone:

> A single packet in a period of the square wave can be selected and set to
> the opposite value of that period. After one RTT it comes back and another packet
> is selected and set again to the opposite value of that period.
> And the process can start again. By measuring the distance between these
> special packets, it is possible to measure the RTT in addition to packet loss.
> The periods with the special packets have one packet less than expected but this is
> easy to recognize and to take into account by both endpoints.

The Q bit and S bit signals use two single bit values and the new signal
is a Combined sQuare bit (C bit) signal.
This mechanism uses a single bit that serves two purposes: a loss indicator
and a delay indicator.
It is worth highlighting that it is similar to the Delay bit (D bit), described
in {{RFC9506}}.
Indeed, the D bit, as enhancement of the S bit, is set only once per RTT
and a single packet with the marked Delay bit bounces
between a client and a server. The C bit value can also be seen as an Exclusive
OR operation (XOR) between the two Q bit and D bit values:
C = Q XOR D.

Since C bit incorporates both Q bit and S bit information, the same considerations
for the two separate signals in {{RFC9506}} can also be extended in the case of C bit.
Therefore, all the possible measurements (end-to-end, hop-by-hop) that are
enabled by using only C bit can be found in {{RFC9506}} by merging Q bit and S bit
derived measurements.


# CoAP Performance Measurement Option {#PMO}

{{PMoption}} shows the property of the CoAP Performance Measurement (PM) Option.
The formatting of this table is reported in {{RFC7252}}.
The C, U, N, and R columns indicate the properties Critical, Unsafe, NoCacheKey,
and Repeatable as defined in {{RFC7252}}.

~~~~
   +--------+---+---+---+---+--------+--------+--------+---------+
   | Number | C | U | N | R | Name   | Format | Length | Default |
   +========+===+===+===+===+========+========+========+=========+
   | TBD    |   | x | - |   |   PM   | uint   | 1      | 0       |
   +--------+---+---+---+---+--------+--------+--------+---------+
             C=Critical, U=Unsafe, N=NoCacheKey, R=Repeatable
~~~~
{: #PMoption title='CoAP PM Option Properties'}


The CoAP PM Option is Elective and Proxy Unsafe. But as discussed in {{ncproxies}}, it MAY also
be Safe-to-Forward in some implementations with non-caching proxies.

As detailed in {{oscore}}, the option can be of class U, I and E in terms of OSCORE processing.

Note that it could be possible to make use of one bit in the option to identify
the mode. In this way two patterns can be defined.

## Structure of the PM Option

The value of the PM option is a 1 byte unsigned integer.
This integer value encodes the following fields:


~~~~
                        0
                        0 1 2 3 4 5 6 7
                       +-+-+-+-+-+-+-+-+
                       |M|   Pattern   |
                       +-+-+-+-+-+-+-+-+
~~~~
{: #PM title='Value of the CoAP Performance Measurement Option'}


Where:

* The Mode bit (M bit) can be set to 1 or 0 and it is used to identify whether
  the Option follows pattern 0 (M bit = 0) or pattern 1 (M bit = 1).

* Pattern bits can be of two kinds as reported below.


The PM Option can employ two patterns based on the value of the M bit:


~~~~
                        0
                        0 1 2 3 4 5 6 7
                       +-+-+-+-+-+-+-+-+
                       |0|Q|S|  Event  |
                       +-+-+-+-+-+-+-+-+
~~~~
{: #PM0 title='CoAP Performance Measurement Option pattern 0'}


~~~~
                        0
                        0 1 2 3 4 5 6 7
                       +-+-+-+-+-+-+-+-+
                       |1|C|   Event   |
                       +-+-+-+-+-+-+-+-+
~~~~
{: #PM1 title='CoAP Performance Measurement Option pattern 1'}


The CoAP Option could be defined with 2 PM bits (S and Q) or defined with
a single PM bit (C bit).

Where:

* Q bit is used in pattern 0. It is described in {{RFC9506}};

* S bit is used in pattern 0. It is described in {{RFC9312}} and also embedded in the QUIC Protocol {{RFC9000}};

* C bit is used in pattern 1. It is based on the enhancement of the Q bit signal
  with the S bit information. The two methods are described in {{RFC9506}} and coupled as detailed in {{Cbit}};

* Event bits MAY be used to encode additional Loss and Delay information based on well-defined encoding
  and they can also be used by on-path probes. If these Event bits are all zero, they MUST be ignored on receipt.
  Note that, if pattern 1 is used instead of pattern 0, there is an extra bit available for the event bits
  in order to carry additional information for the on-path probe.

It is worth noting that the only differences between the two patterns are
related to the accuracy of the measurements and to the number of event bits.
Further details can be found in {{RFC9506}}.

The Event bits can be divided into two parts, for instance: loss event bits and delay event bits.
Based on the average RTT, an endpoint can define different levels of thresholds
and set the delay event bits accordingly.
The same applies to loss event bits. In this way an on-path probe becomes
aware of the network conditions by simply reading these Event bits
and without applying any algorithm.

The on-path probe can read the event signaling bits and could be the Proxy
or the Gateway which interconnects disjointed CoAP networks.
It MAY communicate with Client and Server to set some parameters
such as timeout based on the network performance.

The CoAP PM Option described in this document can be used in both requests
and responses. If a CoAP endpoint does not implement the measurement methodologies,
it can simply exclude the option in the outgoing message.
In this way the other CoAP endpoints become aware that the measurement cannot
be executed in that case.

The fixed number of packets to create the Q bit (or C bit) signal is predefined
and its value is configured from the beginning for all the CoAP endpoints, as also mentioned in {{MaO}}.

It is worth mentioning that in some specific circumstances, e.g. CoAP clients
that "observe" resources {{RFC7641}} or empty-ACKs, the measurements can be done only for one direction.
For bidirectional measurements, it is required to have traffic in both directions.


# Application Scenarios

The main usage of the CoAP PM Option is to do end-to-end measurement between
the client and the server, but it can also allow on-path measurements.
The on-path measurement is the additional feature.
This information can be used to monitor the network in order to check the
operational performance and to employ further network optimization.

The intermediaries or on-path nodes could be:

* Probes that must be able to see deep into application.
* Proxies that, as specified in {{RFC7252}}, are CoAP endpoints tasked by CoAP clients
  to perform requests on their behalf.


## Non-proxying endpoints {#np}

The CoAP PM Option can be applied end-to-end between client and server and,
since it is Elective, it can be ignored by an endpoint
that does not understand it.


~~~~
     +--------+                               +--------+
     | Client |---------------+---------------| Server |
     +--------+               |               +--------+
                              |
                            probe
~~~~
{: id="Fig.1" title='Scenario with non-proxying endpoints'}

The enabled measurements are:

* end-to-end loss and delay measurements between Client and Server,

* on-path upstream and downstream loss and delay components (as explained in {{RFC9506}})
  if there is a probe (e.g. network functions).

* on-path intra-domain loss and delay portion if there are more than one probe
  as a result of the difference between the computed
  upstream or downstream components (as explained in {{RFC9506}}).

The on-path network probes can read Q bit and S bit (or C bit) and implement
the relevant algorithms to measure losses and RTT.
Otherwise they can simply read the Event bits and be informed about the performance
without implementing any algorithm.
The event signaling bits can be sent from the Server (that can do the performance
measurement calculation) to the Client, or viceversa.

If the CoAP PM Option is applied between client and server, a probe can measure
the total RTT by using the S bit, indeed it allows RTT measurement for all the intermediate points.
Additionally, with the Q bit and by applying {{RFC9341}}, it is also possible to do
hop-by-hop measurements for loss and delay and segment where possible between the probes,
according to the methodologies described in {{RFC9506}}.
Alternatively, it is possible to use the C bit to get the same information
for loss and delay as explained in {{RFC9506}}.


## Collaborating proxies {#cproxies}

The proxies can be collaborating and it means that they understand and are
configured to handle the CoAP PM Option.
The CoAP PM Option can be handled on the client, on the server and on each
Proxies.


~~~~
          client-proxy        proxy-proxy       proxy-server
           <------->     <----------------->     <------->
  +--------+       +-----+                 +-----+       +--------+
  | Client |---+---|Proxy|--------+--------|Proxy|---+---| Server |
  +--------+   |   +-----+        |        +-----+   |   +--------+
               |                  |                  |
             probe              probe              probe
~~~~
{: id="Fig.2" title='Scenario with collaborating proxies'}

In case of collaborating proxies, the enabled measurements are different
depending on where the CoAP PM Option is applied.

It can be possible to apply the CoAP PM Option between Client and Proxy,
between the Proxies and between Proxy and Server.
The sessions are separated end-to-end between Client and Server and the enabled
measurements can be done on the separated sessions:

* loss and delay measurements between Client and Proxy, between Proxies and
  between Proxy and Server,

* on-path upstream and downstream loss and delay components (as explained in {{RFC9506}}) on each Proxy,

* on-path upstream and downstream loss and delay components (as explained in {{RFC9506}}) on each Probe,

* end-to-end loss and delay measurements as a result of the addition of the
  loss and delay contributions of the separated sessions.

* on-path intra-domain loss and delay portion as a result of the difference
  between the computed upstream or downstream components
  (as explained in {{RFC9506}}).


So, if there are CoAP proxies, the measurement can be done between the Proxies
or between a Proxy and the Client or between a Proxy and the Server.
It can be done through Spin bit or by applying {{RFC9341}} on the sQuare Bit signal.
Therefore, it is also possible to do hop-by-hop measurements for loss and delay and
segment where possible according to the methodologies described in {{RFC9506}}.


## Non-collaborating proxies {#ncproxies}

The proxies can be non-collaborating and this means that they do not handle
the CoAP PM Option.
The CoAP PM Option can be applied end-to-end between client and server.

There are some issues that may occur in case of non-collaborating proxies.
In general, since CoAP proxies hide the identity of the client,
the data would appear mixed after the proxy in the presence of more than
one client doing the measurements.
Similarly, since CoAP proxies could also apply caching, it can happen to
receive mixed signals in the presence of cache entries.

In case of collaborating proxies, these issues are solved because the measurements
can be segmented and done between the Client and a Proxy
or between the Proxies or between a Proxy and the Server. In this case, it
could be possible for the proxy to still use the PM Option
for the bundle of clients for a specific server.

While, in case of non-collaborating proxies, it is RECOMMENDED to use the
Option only for a single client and a single server at once
in order to avoid that traffic from different clients would be mixed.
But, if the proxy has also cached data, the data can be reordered and mixed,
so that they cannot be used for measurement. For this reason, the PM Option
is defined as Proxy Unsafe and it is intended to be unsafe for forwarding
by a proxy that does not understand it. In conclusion, if there are non-collaborating
and caching proxies, the measurements would not be possible.


### Non-caching proxies {#ncncproxies}

An implementation MAY consider the PM Option as Safe-to-Forward if the proxies
are non-caching in general or in the only case the PM Option
is included in the message.


~~~~
                            client-server
           <--------------------------------------------->
  +--------+       +-----+                 +-----+       +--------+
  | Client |---+---|Proxy|--------+--------|Proxy|---+---| Server |
  +--------+   |   +-----+        |        +-----+   |   +--------+
               |                  |                  |
             probe              probe              probe
~~~~
{: id="Fig.3" title='Scenario with non-collaborating and non-caching proxies'}

In case of non-collaborating and non-caching proxies, proxies MAY be configured
to handle the PM Option as Safe-to-Forward, and
it means that not recognized option MUST be forwarded. Therefore, the enabled
measurements for a single client and a single server at once can be:

* end-to-end loss and delay measurements between Client and Server,

* on-path upstream and downstream loss and delay components (as explained in {{RFC9506}}) on each Probe,

* on-path intra-domain loss and delay portion as a result of the difference
  between the computed upstream or downstream components
  (as explained in {{RFC9506}}).


## DTLS {#dtls}

CoAP can be secured using Datagram TLS (DTLS) {{RFC6347}} over UDP
and it can prevent on-path measures in case of non-proxying endpoints.
When a client uses a collaborating proxy the sessions client-proxy, proxy-proxy,
proxy-server are secured using DTLS, but the separated sessions can still be measured.
An on-path probe cannot perform the measurements in any case.


## OSCORE {#oscore}

The CoAP PM Option can be used with OSCORE {{RFC8613}}.
Since an OSCORE message may contain both an Inner and an Outer instance of
a certain CoAP message field,
the CoAP PM Option can be an Inner option or an Outer option based on the
specific applications and required security and privacy.
Then network administrators can put their measurement probes in one or more
places to break down the different RTT and loss contributions where it is relevant
(e.g. at the ingress/egress of their respective network segments).

Inner options (Class E) are used to communicate directly with the other endpoint
and are encrypted and integrity protected. If the CoAP PM Option is sent as Inner Option,
it only enables end-to-end measurements in all the cases.
In case of collaborating proxies the separated sessions client-proxy, proxy-proxy, proxy-server
cannot be measured.

Outer options (Class U or I) are intended to be used to support proxy operations
and are unprotected or integrity protected only.
If the CoAP PM Option is sent as Outer Option, it allows both end-to-end
and on-path measurements by enabling hop-by-hop and segmented loss and delay measurements on the proxies.

If an OSCORE endpoint sends both outer and inner option, the inner is for
measuring the connection to the end-to-end peer,
and the outer can be used for measuring the connection to next proxy.

If the PM option is used as an Outer Option, it may also be integrity-protected,
to be reliably processed and this would require using also DTLS or an OSCORE association with a proxy
{{I-D.ietf-core-oscore-capable-proxies}}.


# Management and Configuration {#MaO}

The measurement points can perform RTT and packet loss calculation without
the need of any Network Management System (NMS) to collect information.
It may be possible that the measurement points inform the NMS if there are particular network conditions
(e.g. high packet loss or high RTT).
For some parameters (e.g. 64 packets sQuare Bit signal), it is assumed static
configuration on the client.
There are several alternatives for the implementation but this is out of
scope of this document.


# Congestion Control

As specified in Section 4.7 of {{RFC7252}}, clients (including proxies)
have to strictly limit the number of simultaneous outstanding interactions
that they maintain to a given server (including proxies) to NSTART.
The default value of NSTART is 1 but a value for NSTART greater than one
is also possible.
The CoAP PM Option implementation must not affect CoAP congestion control
mechanisms.


# Security Considerations

Security considerations related to CoAP proxying are discussed in {{RFC7252}}.

A CoAP endpoint can use the CoAP PM Option to affect the measures of a network
into which it is making requests by maliciously specifying a wrong option value.
Also, the PM bits may reveal performance information outside the administrative
domain.
To prevent that, a CoAP proxy that is located at the boundary of an administrative
domain MAY be instructed to strip the payload or part of it before forwarding the
message.

It is worth highlighting what happens if devices, transport network and server
are operated by different administrative domains. Security concerns need to be
taken into account.

CoAP can be used with DTLS {{RFC6347}} and it can prevent on-path measures
by on-path probes while it is still possible to do measurements on collaborating
proxies, as explained above.

CoAP can also be used with OSCORE {{RFC8613}} and the CoAP PM options can be
integrity protected end-to-end by OSCORE.
In this case, as explained above and differently from DTLS, the CoAP PM can
easily work with OSCORE.
OSCORE ensures end-to-end integrity protection and would tell the endpoints
if someone tampered with the option value, but it doesn't mean that the endpoints
are not lying to the probe. However it is possible to assume that for the typical
CoAP applications it is less likely that the endpoints are attackers while
it is more likely that an on-path probe is the attacker.


# IANA Considerations {#IANA}

IANA is requested to add the following entry to the "CoAP Option
Numbers" sub-registry available at
https://www.iana.org/assignments/core-parameters/core-parameters.xhtml#option-numbers:

~~~~
          Number          Name              Reference
          ---------------------------------------------
          TBD           PM Option          [This draft]

~~~~
{: #PMoptionNs title='CoAP PM Option Numbers'}


--- back

# Acknowledgements {#Acknowledgements}
{:unnumbered}

The authors would like to thank
{{{Christian Amsüss}}},
{{{Carsten Bormann}}},
{{{Marco Tiloca}}},
{{{Thomas Fossati}}}
for the precious comments and suggestions.
