---
title: "Media over QUIC Relay Benchmark Methodology"
abbrev: moq-bench
docname: draft-evens-moq-bench-latest
date: {DATE}
category: std

ipr: trust200902
area:  "Web and Internet Transport"
submissionType: IETF
workgroup: "Media Over QUIC"
keyword:
 - media over quic
venue:
  group: "Media Over QUIC"
  type: "Working Group"
  mail: "moq@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/moq/"
  github: "timevens/moq-relay-peering"
  latest: "https://timevens.github.io/moq-relay-peering/draft-evens-moq-relay-peering.html"

stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs, docmapping]

author:
  -
    ins: T. Evens
    name: Tim Evens
    organization: Cisco
    email: tievens@cisco.com
  -
    ins: T. Rigaux
    name: Tomas Rigaux
    organization: Cisco
    email: trigaux@cisco.com

contributor:
#- name: First Last
#  org:  Cisco
#  email:  email@cisco.com

normative:
  MOQT: I-D.ietf-moq-transport

informative:
  QUIC: RFC9000

--- abstract

This document defines a methodology to benchmark {{MOQT}} relays by utilizing
configuration profiles to perform a series of tests to evaluate the performance
of relays. Completed test results provide benchmark information that can be used
for comparison and analysis of {{MOQT}} relays.

--- middle

# Introduction

 {{MOQT}} specifies the client–server interactions and messaging used by clients and relays to fan-out published data to one or more subscribers. Implementations employ several state machines to coordinate these interactions. Relays must serialize and deserialize data objects and their extension headers when forwarding to subscribers. Because {{MOQT}} runs over {{QUIC}}, relay performance is directly affected by receive/send processing, encryption/decryption, and loss‑recovery ACK handling.

 To evaluate relay performance under realistic conditions, this document defines a benchmarking methodology that mimics real‑world use cases.

# Terminology

{::boilerplate bcp14-tagged}

Commonly used terms in this document are described below.

Track Configurations:
: Track configurations define


Config Profile:
: A set of Track Configurations that are used to perform a series of tests to evaluate the performance
of relays.

Test Scenario:
: A test scenario is a set of config profiles that are used to perform a series of tests to evaluate the performance
of relays.

## Data Patterns

Data transmission patterns are relative to the data that is being transmitted at the time of transmission.

### Video Media

Video media is a common {{MOQT}} use case. A video media track typically begins with a large data object, followed by a series of smaller data objects within the same group. The initial object is often significantly larger than the subsequent ones. Video is transmitted at regular intervals (e.g., every 33 ms for 30 fps), producing a bursty fan‑out pattern at each interval. Each data object commonly carries at least three extension
headers.

Video has a direct dependency on previous objects within a group. If objects are lost in a group, it breaks
the ability to render the video correctly.  

Video is often larger than MTU and cannot be transmitted as
defined in {{MOQT}} using datagram. Unless video is very low quality, video uses {{QUIC}} stream forwarding instead of datagram forwarding.

### Audio Media

Audio media is a common {{MOQT}} use case. An audio media track typically maintains fixed intervals with similarly sized data objects. Like video, transmission occurs at each interval, producing a periodic fan‑out burst. Each data object usually carries fewer than three extension headers.

{{MOQT}} does not define how to handle larger than MTU sized data objects for {{QUIC}} datagarms.  Audio does not
have the same requirement as video where subsequent data objects are related to the previous object or group.

Audio is a prime candidate for datagram considering the size of the data objects are less than MTU and
there is no dependency on previous object within a group.

Audio primarily uses {{QUIC}} datagrams for forwarding.

### Fetch

A fetch in {{MOQT}} is similar to a retrieval
of a file where the data will be sent all at once and not paced on an interval. In {{MOQT}} the data
will be sent using a single {{QUIC}} stream. The data will be sent as fast as the receiver {{QUIC}} connection
can receive it.

### Interactive Data

Chat, metrics, and logging are use cases for interactive data. This data is sent based on interaction, such as a human typing a message or metrics event, or logging event.  A unique
characteristic of this data is that it is sent when needed and isn't always constant. Metrics might be an exception where metrics are sent at a constant interval, but metrics may also be sent based on a triggered event.  

Interactive data is often sent using {{QUIC}} streams but can also be sent via datagram considering the size of the data is often less than MTU size.  

### Group and Subgroup Churn

In {{MOQT}}, when a group or subgroup changes a {{QUIC}} stream is created for the group/subgroup.  This results in churn for the {{MOQT}} relay implementation to handle multiple groups/subgroups in-flight.  The data transmission use case drives how frequently groups/subgroups change, which can impact the performance of relays. Changing of group/subgroup is only impactful
with {{MOQT}} stream tracks. 


## Scenarios

A combination of data patterns are often used to define a scenario. The scenario becomes a Config Profile.

There are video, audio, and fetch scenarios. Often, these scenarios combine a mix of the above data patterns.

### Single Publisher to Multiple Subscribers - Audio

### Single Publisher to Multiple Subscribers - Audio and Video

### Groups of Publishers and Subscribers (aka Meeting) 

### Fetch Concurrent With Other Scenarios


#### TODO: Remove

| Format | Textual Value                         |
|:-------|:--------------------------------------|
 NidF1    | `<uint16>.<uint16>:<uint16>.<uint16>`
 NidF2    | `<uint16>.<uint16>:<uint32>`
 NidF3    | `<uint32>:<uint16>.<uint16>`
 NidF4    | `<uint32>:<uint32>`
{: title="Textual NodeId Formats" }

### Node Structure

The node structure conveys relevant information needed by the selection algorithm.

Id:
: NodeId of the node as an unsigned 64bit number

Contact:
: String value that defines the contact information for this node. This **SHOULD** be the FQDN that resolves
A/AAAA/CNAME uniquely for the node. Peering will use this to establish
a connection to this node. This can be an IPv4 or IPv6 address.

Type:
: Node Type for the node.

BestViaRelays:
: An array of `NodeId`s of other nodes that are considered best to reach the node. This is based on the node itself
running reachability probing. The array has a fixed maximum number of via relays.

Longitude:
: Longitude of where the node is located, as a double/float value

Latitude:
: Latitude of where the node is located, as a double/float value


~~~
{
  full_hash (uint64),
  received_by_node_id (uint64),
  origin_node_id (uint64),
  accepted_by_node_id (uint64),
  namespace_tuple_hash (uint64[]),
  name_hash (uint64),
}
~~~
{: #sib-fields title="SIB Fields" }


# Security Considerations {#security}
TODO: Expand this section

# IANA Considerations
TODO: Expand this section
