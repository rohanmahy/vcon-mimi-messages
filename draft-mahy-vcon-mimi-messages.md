---
title: "VCON for MIMI Messages"
abbrev: "VCON for MIMI"
category: info

docname: draft-mahy-vcon-mimi-messages-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Applications and Real-Time"
workgroup: "Virtualized Conversations"
keyword:
 - instant messaging
 - message history
venue:
  group: "Virtualized Conversations"
  type: "Working Group"
  mail: "vcon@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/vcon/"
  github: "rohanmahy/vcon-mimi-messages"
  latest: "https://rohanmahy.github.io/vcon-mimi-messages/draft-mahy-vcon-mimi-messages.html"

author:
 -
    fullname: Rohan Mahy
    organization: Rohan Mahy Consulting Services
    email: rohan.ietf@gmail.com

normative:

informative:


--- abstract

This document describes extensions to the Virtualized Conversation (VCON)
syntax for instant messaging systems using the More Instant Messaging
Interoperability (MIMI) content format.


--- middle

# Introduction

VCON {{!I-D.ietf-vcon-vcon-container}} is a format for recording and
transmitting conversations. MIMI content {{!I-D.ietf-mimi-content}}
is a format for conveying Instant Messages in an interoperable way
when end-to-end encrypted. This document describes a way to translate
a set of MIMI messages in a conversation or part of a conversation into
a VCON.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Syntax

A MIMI conversation (or portion thereof) is represented in VCON with
a mandatory list of parties and dialogs, a new top-level room object.

## Room information

This document adds a top-level `room` object. It contains metadata
known about the room at the start of the VCON period.

- `id` is the MIMI room ID URL. It is mandatory.
- `name` is the textual name of the room. It is optional.

## Parties

The `parties` array MUST contain all the
participants who were in the conversation at any point in time during the
period represented by the VCON.

This document adds a new party_object_type: `imUri`. It is mandatory.
The `name` field is optional. The `role` indicates the MIMI role and is
optional.

## Extensions to the dialog object

The dialog object consists of an array of instant messages of type "text".
The start time is set to the hub received timestamp when available.
The duration is zero. The parties array consist of the party indexes of those who were active participants when the message was sent.
The originator is set to the parties index of the sender of the message.

- `messageId` is the base64url encoding of the MIMI content messageId. It is mandatory.
- `replaces` is the base64url encoding of the MIMI content messageId of the message this message replaces. It is optional if empty.
- `expires` is the expiration date/time of the message expressed as a VCON
(text) date_type. It is optional if empty.
- `inReplyTo` is the base64url encoding of the MIMI content messageId of the message to which this message is replying (or reacting). It is optional if empty.
- `lastSeen` is an array of base64url-encoded message ids. It is mandatory.
- `mimiExtensions` is a object (map) containing MIMI extensions. It is optional.

VCON typically expresses content using the body, encoding, and mimetype
fields. In order to preserve this convention we use these fields directly for a MIMI SinglePart structure, but use new MultiPart and ExternalPart
structure for those MIMI structures.

ExternalPart has several fields for the decryption and integrity of the referenced content. These can be omitted once the content has been downloaded, decrypted, verified, and included in the VCON attachments array.

- `disposition` - optional if set to the default value ("render")
- `language` - optional if absent
- `partIndex` is an integer. It is mandatory


## MultiPart

- `partSemantics` is one of "chooseOne", "singleUnit", or "processAll". It is mandatory.
- `parts` is an array of `Part`s. It is mandatory.

## ExternalPart

- mimetype (contentType in MIMI). optional but recommended.
- url is the URL of the content as a text string. mandatory
- expires is a date_type (text) date. optional.
- size is an integer number of octets. optional.
- description is a text string. optional.
- filename is a text string. optional.

These encryption/validation related fields can be omitted once the content is available locally as an attachment. All of them are base64url encoded strings. Otherwise they are mandatory if present in the MIMI content.

- encAlg
- key
- nonce
- aad
- hashAlg
- contentHash

**TODO**: how to represent a downloaded and decrypted object in attachments?

## Part

- `disposition` - optional if set to the default value ("render")
- `language` - optional if absent
- `partIndex` is an integer. It is mandatory
- `cardinality` is one of "nullpart", "single", "external", or "multi". mandatory.

if cardinality is "single", then body, encoding, and mimetype
fields are included directly in the Part object.

## Changes to the room

Changes to the room metadata or participation should be accompanied by
a new dialog type:

- room metadata changes
- participants joining and leaving
- participants adding new clients
- moderation events?

**TODO**

## party_event_type events

~~~
party_event_type.event /= "add" / "welcome" / "leave" / "remove" / "ban"
~~~

- add: user added herself directly
- welcome: user added by someone else
- leave: user leaves of their own accord
- remove: user is removed by another user
- ban: user is banned from the group


# Examples

Will fill in after the draft deadline.


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
