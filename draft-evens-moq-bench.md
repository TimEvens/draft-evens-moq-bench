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
  github: "timevens/draft-evens-moq-bench"
  latest: "https://timevens.github.io/draft-evens-moq-bench/draft-evens-moq-bench.html"

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

# Data Patterns

Data transmission patterns are relative to the data that is being transmitted at the time of transmission.

## Video Media

Video media is a common {{MOQT}} use case. A video media track typically begins with a large data object, followed by a series of smaller data objects within the same group. The initial object is often significantly larger than the subsequent ones. Video is transmitted at regular intervals (e.g., every 33 ms for 30 fps), producing a bursty fan‑out pattern at each interval. Each data object commonly carries at least three extension
headers.

Video has a direct dependency on previous objects within a group. If objects are lost in a group, it breaks
the ability to render the video correctly.

Video is often larger than MTU and cannot be transmitted as
defined in {{MOQT}} using datagram. Unless video is very low quality, video uses {{QUIC}} stream forwarding instead of datagram forwarding.

## Audio Media

Audio media is a common {{MOQT}} use case. An audio media track typically maintains fixed intervals with similarly sized data objects. Like video, transmission occurs at each interval, producing a periodic fan‑out burst. Each data object usually carries fewer than three extension headers.

{{MOQT}} does not define how to handle larger than MTU sized data objects for {{QUIC}} datagarms.  Audio does not
have the same requirement as video where subsequent data objects are related to the previous object or group.

Audio is a prime candidate for datagram considering the size of the data objects are less than MTU and
there is no dependency on previous object within a group.

Audio primarily uses {{QUIC}} datagrams for forwarding.

## Fetch

A fetch in {{MOQT}} is similar to a retrieval
of a file where the data will be sent all at once and not paced on an interval. In {{MOQT}} the data
will be sent using a single {{QUIC}} stream. The data will be sent as fast as the receiver {{QUIC}} connection
can receive it.

## Interactive Data

Chat, metrics, and logging are use cases for interactive data. This data is sent based on interaction, such as a human typing a message or metrics event, or logging event.  A unique
characteristic of this data is that it is sent when needed and isn't always constant. Metrics might be an exception where metrics are sent at a constant interval, but metrics may also be sent based on a triggered event.

Interactive data is often sent using {{QUIC}} streams but can also be sent via datagram considering the size of the data is often less than MTU size.

## Group and Subgroup Impact to Relays

In {{MOQT}}, when a group or subgroup changes a {{QUIC}} stream is created for the group/subgroup.  This results in additional state for the {{MOQT}} relay implementation to handle multiple groups/subgroups in-flight.  The data transmission use case drives how frequently groups/subgroups change, which can impact the performance of relays. Changing of group/subgroup is only impactful with {{MOQT}} stream tracks.


# Scenarios

A combination of data patterns are used to define a scenario. The scenario becomes a Config Profile. Various data patterns are combined to define a scenario. The scenario (Config Profile)
is used to benchmark a relay.

Below describes common scenarios that the benchmark and configuration profiles need to support.
Other scenarios can be defined as needed using the configuration profiles.

## Single Publisher to Multiple Subscribers - Audio

~~~ aasvg
                   ┌──────────────┐
                   │   Publisher  │
                   └──────────────┘
                           │
                           │
                           ▼
                      ┌─────────┐
        ┌─────────────│  Relay  │──────────────┐
        │             └─────────┘              │
        │                  │                   │
        ▼                  ▼                   ▼
┌──────────────┐    ┌──────────────┐   ┌──────────────┐
│ Subscriber 1 │    │ Subscriber 2 │   │ Subscriber 3 │
└──────────────┘    └──────────────┘   └──────────────┘
~~~
{: artwork-align="center" artwork-name="Publisher to Many Subscribers"}

The above diagram shows a single publisher sending audio to multiple subscribers.

A single audio track is published using datagram. The audio track is sent at a constant interval that is then sent to all subscribers.  The benchmark is to see how many subscribers can be supported by the relay using datagram forwarding.


## Single Publisher to Multiple Subscribers - Audio and Video

~~~ aasvg
                   ┌──────────────┐
                   │   Publisher  │
                   └──────────────┘
                           │
                           │
                           ▼
                      ┌─────────┐
        ┌─────────────│  Relay  │──────────────┐
        │             └─────────┘              │
        │                  │                   │
        ▼                  ▼                   ▼
┌──────────────┐    ┌──────────────┐   ┌──────────────┐
│ Subscriber 1 │    │ Subscriber 2 │   │ Subscriber 3 │
└──────────────┘    └──────────────┘   └──────────────┘
~~~
{: artwork-align="center" artwork-name="Publisher to Many Subscribers"}

The above diagram shows a single publisher sending audio and video to multiple subscribers.

Two tracks are published, an audio datagram track and a reliable stream track for video. The benchmark is to see how many subscribers can be supported by the relay using both datagram and
stream forwarding tracks.

## Groups of Publishers and Subscribers (aka Meeting)

~~~ aasvg

   ┌─────────────────────────────────────────────────────┐
   │ namespace = meeting, 1234                           │
   │                                                     │
   │ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐  │
   │ │ Publisher 1  │ │ Publisher 2  │ │ Publisher 3  │  │
   │ │ Subscriber 1 │ │ Subscriber 2 │ │ Subscriber 3 │  │
   │ └───────▲──────┘ └──────▲───────┘ └──────▲───────┘  │
   │         │               │                │          │
   └─────────┼───────────────┼────────────────┼──────────┘
             │               │                │
             │               │                │
             │               │                │
             │        ┌──────┴───────┐        │
             ├────────┤    Relay     ├────────┤
             │        └──────┬───────┘        │
             │               │                │
             │               │                │
             │               │                │
   ┌─────────┼───────────────┼────────────────┼──────────┐
   │         │               │                │          │
   │ ┌───────▼──────┐ ┌──────▼───────┐ ┌──────▼───────┐  │
   │ │ Publisher 1  │ │ Publisher 2  │ │ Publisher 3  │  │
   │ │ Subscriber 1 │ │ Subscriber 2 │ │ Subscriber 3 │  │
   │ └──────────────┘ └──────────────┘ └──────────────┘  │
   │                                                     │
   │ namespace = meeting, 5678                           │
   └─────────────────────────────────────────────────────┘
~~~
{: artwork-align="center" artwork-name="Groups of Publishers and Subscribers"}

The above diagram shows a set of publishers and subscribers. Each client in this scenario publishes two tracks, audio and video, and subscribe to the other subscribers audio and video tracks.  In this scenario, the set of publishers and subscribers mimic a video conference meeting.  The benchmark is to see how many sets (aka meetings) can be supported by the relay using datagram and stream forwarding tracks.

# Methodology

## Config Profile

A configuration profile consists of per track configuration settings. The following track settings are defined:

| Field | Notes |
|:-------|:--------------------------------------|
| namespace | Namespace tuple of the track |
| name | Name of the track |
| reliable | True to use QUIC streams and false to use datagrams |
| mode | Is a value of 1 to indicate publisher, 2 to indicate subscriber, or 3 to indicate both. When both, the track namespace or name will often be interpolated based on the test script |
| priority | Priority of the track |
| ttl | Time to live for objects published |
| interval | Interval in milliseconds to send data objects |
| Objects Per Group | Number of objects within a group |
| First Object Size | First object of group size in bytes |
| Remaining Object Size | Remaining objects of group size in bytes |
| Duration | Duration in milliseconds to send data objects |
| Start Delay | Start delay in milliseconds |
{: title="Textual NodeId Formats" }

Interpolation can be used to template the configuration profile.

Upon start, the track will start off with `"First Object Size"` of data bytes.  Subsequent objects will use the `Remaining Object Size` in bytes. The remaining objects sent will be
`Objects Per Group` number minus the first object. For example, if `Objects Per Group` is 10 and `First Object Size` is 100 bytes and `Remaining Object Size` is 50 bytes, then the first group will be 100 bytes and the remaining 9 groups will be 50 bytes each.

Groups value will increment based on sending `Objects Per Group` number of objects. TTL is used
to define how long published objects should remain in the transmit queue.

Objects will be published at the `Interval` value.

Objects and groups will continue for as long as the `Duration` value.

## Synchronize Start of Benchmark

Synchronization of the benchmark is required to ensure that all clients start at the same time as the publisher so that objects published can be compared to objects received.

To synchronize the start of the benchmark, the publisher will publish a {{start-message}} repeatedly for the duration of `Start Delay` value in milliseconds. The {{start-message}} will be published every every 1/10th interval of the `Start Delay` value in milliseconds. If 1/10th is less than 100ms, the {{start-message}} will be published every 100ms.

Clients will subscribe to the published track and will wait for the {{start-message}} to be received. If the start message is not received before data messages or completion message,
the track benchmark is considered failed.

Clients will ignore the repeated {{start-message}} after receiving the first one. The test begins on the first {{data-message}} message received. {{start-message}} MUST not be sent after sending the first {{data-message}} message.

## Completion of benchmark

The per track benchmark is completed when the publisher sends a {{completion-message}}. The {{completion-message}} will be sent after the last {{data-message}} message is sent.

## Messages

Testing utilizes various messages to start testing, send data, and report metrics on completion.

The following messages are defined:

### START

~~~
START Message {
  type (8) = 0x01,
  objects_per_group (uint32),
  first_object_size (uint32),
  remaining_object_size (uint32),
  interval (uint32),
}
~~~
{: #start-message title="Start Test Message" }

### DATA

~~~
DATA Message {
  type (8) = 0x02,
  group_number (uint64),
  object_number (uint64),
  milliseconds_since_first_object (uint32_t),
  data_length (uint32),
  data (bytes),
}
~~~
{: #data-message title="Data Message" }

milliseconds_since_first_object:
: Number of milliseconds since the first object was sent. The first object starts at zero and is incremented by milliseconds from the publisher for each message sent. The publisher adds the time when it
sends the message. If there are publisher delays, then this time would reflect the delay variance from expected interval.

### COMPLETION

~~~
COMPLETION Message {
  type (8) = 0x03,
  objects_sent (uint64),
  groups_sent (uint64),
  total_duration (uint32),
}
~~~
{: #completion-message title="Completion Message" }


total_duration:
: Total duration is the total duration in milliseconds from first object to last object sent


## Benchmark Metrics

### While Track is in Progress

The following metrics are computed ongoing as the track is in progress:

| Metric                  | Notes                                                                                                                                                      |
| :---------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Received Objects        | Count of received objects                                                                                                                                  |
| Received Groups         | Count of received groups                                                                                                                                   |
| Receive Delta           | Delta time in milliseconds from last message to current message received                                                                                   |
| Average Delta           | Average delta time in milliseconds from last message to current message received                                                                           |
| Max Delta               | Maximum delta time in milliseconds seen                                                                                                                    |
| Publisher Variance      | Received milliseconds_since_first_object should be zero based on interval.  This metric tracks the variance of publisher sending delay                     |
| Receive Variance        | Received milliseconds_since_first_object should be zero based on interval.  This metric tracks the variance of received objects based on expected interval |
| Average Bits Per Second | Average bits per second received                                                                                                                           |



### On completion of track

The following result metrics are computed at the end of a track that has completed. Each track has the following metrics:

| Metric                     | Notes                                                                                               |
| :------------------------- | :-------------------------------------------------------------------------------------------------- |
| Objects Sent               | Number of objects reported sent by the publisher                                                    |
| Groups sent                | Number of groups reported sent by the publisher                                                     |
| Objects Received           | Number of objects received                                                                          |
| Groups Received            | Number of groups received                                                                           |
| Lost Objects               | Computed based on objects received to objects sent                                                  |
| Total Duration             | Total duration in milliseconds from the publisher                                                   |
| Actual Duration            | Duration in milliseconds from received first object to last object received                         |
| Average Receive Variance   | The average receive variance of received objects based on expected interval                         |
| Average Publisher Variance | The average publisher variance of objects published                                                 |
| Average Bits Per Second    | The average bits per second received                                                                |
| Expected Bits Per Second   | Computed bits per second rate based on {{start-message}} first, remaining sizes and interval values |


## Running Benchmark

Using a configuration profile, multiple instances of clients are ran to connect to the relay.  They will publish and subscribe based on the configuration profile.  Interpolation is used
to dynamically modify the configuration profile namespace and name based on the test script. In the case of mode 3 (both), the namespace and name will be interpolated based on the test script
to avoid multi-publisher and mirroring where the publisher subscribes to itself.

# Security Considerations {#security}
TODO: Expand this section

# IANA Considerations
TODO: Expand this section
