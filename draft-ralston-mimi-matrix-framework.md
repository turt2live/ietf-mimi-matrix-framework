---
title: "Matrix as a Messaging Framework"
abbrev: "Matrix as a Messaging Framework"
category: info

docname: draft-ralston-mimi-matrix-framework-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Applications and Real-Time"
workgroup: "More Instant Messaging Interoperability"
keyword:
 - matrix
 - mimi
 - framework
 - messaging
 - interoperability
venue:
  group: "More Instant Messaging Interoperability"
  type: "Working Group"
  mail: "mimi@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/mimi/"
  github: "turt2live/ietf-mimi-matrix-framework"
  latest: "https://turt2live.github.io/ietf-mimi-matrix-framework/draft-ralston-mimi-matrix-framework.html"

author:
 -
    fullname: Travis Ralston
    organization: The Matrix.org Foundation C.I.C.
    email: travisr@matrix.org

normative:

informative:
  MxSpec:
    target: https://spec.matrix.org/v1.5/
    title: "Matrix Specification | v1.5"
    date: 2022
    author:
      - org: The Matrix.org Foundation C.I.C.


--- abstract

This document describes how Matrix, an existing openly specified decentralized
protocol for secure interoperable communications, works to create a framework
for messaging.


--- middle

# Introduction

Matrix is an existing open standard suitable for Instant Messaging (IM), Voice over IP
(VoIP) signaling, Internet of Things (IoT) communication, and bridging other existing
communication platforms together. In this document we focus largely on the IM use case,
however the concepts can be applied to other forms of communication as well.

The existing Matrix specification {{MxSpec}} is quite large as a whole, it is easily
broken down into reusable specifications for simpler and more effective implementation.
Here, we specify the portion of Matrix which forms the framework of messaging, leaving
the remainder of the open Matrix specification out of scope.

This document assumes some prior knowledge of federated or decentralized systems, such
as the principles of email. This document additionally references concepts from
{{?I-D.rosenberg-mimi-taxonomy}} to build common understanding.

# Overall model

At a high level, Matrix consists of 4 primary concepts:

* Homeservers (also called "servers" for simplicity) contain user accounts and handle
  the algorithms needed to support Rooms.
* Users produce Events into Rooms through their Homeserver.
* Rooms are a defined set of algorithms which govern how all servers in that room behave
  and treat Events.
* Events are pieces of information that make up a room. They can be "state events" which
  track details such as membership, room name, and encryption algorithm or "timeline events"
  which are most commonly messages between users.

Homeservers replicate events created by their users to all other participating homeservers
in the room (any server with at least 1 joined user) and on-demand from those same participating
homeservers. The details regarding how this is done specifically, and how a server becomes
joined to a room, are discussed later in this document.

A 2 homeserver federation might look as follows:

~~~ aasvg
{::include art/network-arch.ascii-art}
~~~
{: #fig-network-arch title="Simple Network Architecture of Matrix"}

In this Figure, Alice, Bob, and Carol are on "hs1", with Dan and Erin being on "hs2". Despite
both having the root domain "example.org", they are considered two completely different homeservers.
Typically, a homeserver would use a domain which was closer to the root (ie: just "example.org"),
however for illustrative purposes and having two homeservers to work with, they have been "improperly"
named here.

If Alice creates a room and invites Bob to it, Alice and Bob can communicate without hs2 ever getting
involved. At any point in the conversation, hs2 can become involved by inviting Dan or Erin and them
accepting that invite. During the join process, hs1 replicates the current state of the room (membership,
room name, etc) to hs2 to validate and persist. After the initial replication, both homeservers replicate
any new content (events) from their side to the other, validating it on the receiving side to ensure that
content is allowed to be sent.

## Eventual Consistency

In federated environments it is extremely likely that a remote server will be offline or unreachable for
a variety of reasons, and a protocol generally needs to handle this network fault without causing undue
inconvenience to others involved. In Matrix, homeservers can go completely offline without affecting other
homeservers (and therefore users) in the room - only users on that offline homeserver would be affected.

During a network fault, homeservers can continue to send events to the room without the involvement of the
remaining homeservers. This applies to both sides of the fault: the "offline" server might have had an
issue where it could not send or receive from the federation side, but users are still able to send events
internally - the server can continue to queue these events until full connectivity is restored. When
network is restored between affected parties, they simply send any traffic the remote side missed and the
room's history is merged together. This is eventual consistency: eventually, all homeservers involved will
reach the same consistent state, even through network issues.

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
