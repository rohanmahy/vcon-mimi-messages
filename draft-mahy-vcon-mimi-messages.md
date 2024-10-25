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
optional. The document also adds a `thumbprint` party_object_type, which is
the JWK thumbprint of the public key of the party. If there are multiple
parties (clients) with the same `imUri` then the thumbprint is required,
otherwise it is optional.

## Extensions to the dialog object

The dialog object consists of an array of instant messages of type "text".
The start time is set to the hub received timestamp when available.
The duration is zero. The parties array consist of the party indexes of those who were active participants when the message was sent.
The originator is set to the parties index of the sender of the message.

- `messageId` is the base64url encoding of the MIMI content messageId. It is mandatory.
- `replaces` is the base64url encoding of the MIMI content messageId of the message this message replaces. It is optional if empty.
- `topicId` is the base64url encoding of the MIMI topicId. It is optional if empty.
- `expires` is the expiration date/time of the message expressed as a VCON
(text) date_type. It is optional if empty.
- `inReplyTo` is the base64url encoding of the MIMI content messageId of the message to which this message is replying (or reacting). It is optional if empty.
- `lastSeen` is an array of base64url-encoded message ids. It is mandatory.
- `mimiExtensions` is a object (map) containing MIMI extensions. It is optional.

VCON typically expresses content using the body, encoding, and mimetype
fields. In order to preserve this convention we use these fields directly
for a MIMI SinglePart structure, but use new MultiPart and ExternalPart
structure for those MIMI structures. For a SinglePart these two fields could
be present.

- `disposition` - optional if set to the default value ("render")
- `language` - optional if absent




## MultiPart

- `partSemantics` is one of "chooseOne", "singleUnit", or "processAll". It is mandatory.
- `parts` is an array of `Part`s. It is mandatory.

## Part

- `disposition` - optional if set to the default value ("render")
- `language` - optional if absent
- `partIndex` is an unsigned integer. It is mandatory
- `cardinality` is one of "nullpart", "single", "external", or "multi". mandatory.

if cardinality is "single" or "external", then body, encoding, and mimetype
fields are included directly in the Part object.


## ExternalPart

- mimetype (contentType in MIMI). optional but recommended.
- url is the URL of the content as a text string. mandatory
- expires is a date_type (text) date. optional.
- size is an integer number of octets. optional.
- description is a text string. optional.
- filename is a text string. optional.

ExternalPart has several fields for the decryption and integrity of the
referenced content. These can be omitted once the content has been
downloaded, decrypted, verified, and included in the VCON attachments array.


These encryption/validation related fields can be omitted once the content is available locally as an attachment. All of them are base64url encoded strings. Otherwise they are mandatory if present in the MIMI content.

- encAlg
- key
- nonce
- aad
- hashAlg
- contentHash

**TODO**: how to represent a downloaded and decrypted object in attachments?

## Changes to the room

Changes to the room metadata or participation should be accompanied by
a new dialog type:

- room metadata changes
- participant identity change
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

## MIMI examples as a VCON

The example vcon consists of the example messages from Section 5 of the MIMI content specification plus a single multipart message.

~~~ json
{
  "vcon": "0.0.1",
  "room": {
    "id": "mimi://example.com/r/engineering_team",
    "name": "Engineering Team",
  }
  "parties": [
    {
      "imUri": "mimi://example.com/u/alice-smith",
      "name": "Alice Smith",
      "role": "moderator",
      "thumbprint": "TODOFIXDZXCog_FfQp-xLemZkD5GKB9H7Z-Y41O4jEw"
    },
    {
      "imUri": "mimi://example.com/u/bob-jones",
      "name": "Bob Jones",
      "role": "member",
      "thumbprint": "TODOFIXYUFE7bV9pXAHZHi5bwSWzLJqjncevs9aLKDg"
    },
    {
      "imUri": "mimi://example.com/u/cathy-washington",
      "name": "Cathy Washington",
      "role": "member",
      "thumbprint": "TODOFIXzH3EeOGbI0-oiDGTXlgKmkMzQyQ2W_Y-TB1U"
    }
  ],
  "dialog": [
    {
      "type": "text",
      "start": "2022-02-08T22:13:45.019-00:00",
      "duration": 0,
      "parties": [
        0, 1, 2
      ],
      "originator": 0,
      "messageId": "yjetjodyYNLBto8YDnwLkEc89W09rOuEivbnxKGfHLQ",
      "lastSeen": [],
      "mimetype": "text/markdown;variant=GFM",
      "encoding": "none"
      "body":
        "Hi everyone, we just shipped release 2.0. __Good work__!",
    },

    {
      "type": "text",
      "start": "2022-02-08T22:13:57.492-00:00",
      "duration": 0,
      "parties": [
        0, 1, 2
      ],
      "originator": 1,
      "messageId": "Ygye96VUeBhTO6U3D0ZllYsSwSdKB3SiRJgxAb3cdAo",
      "inReplyTo": [
        "yjetjodyYNLBto8YDnwLkEc89W09rOuEivbnxKGfHLQ",
        1,
        "6MaXLsvHITc8xzIQp8BRz_7w_UNQL1n7a6DWysTW5T0"
      ],
      "lastSeen": [
        "yjetjodyYNLBto8YDnwLkEc89W09rOuEivbnxKGfHLQ"
      ],
      "mimetype": "text/markdown;variant=GFM",
      "encoding": "none"
      "body": "Right on! _Congratulations_ \'all!",
    },

    {
      "type": "text",
      "start": "2022-02-08T22:13:57.728-00:00",
      "duration": 0,
      "parties": [
        0, 1, 2
      ],
      "originator": 2,
      "messageId": "2YF0k6blW9ZbJ8sgXJgMCTJOlofazdKwVRvf6ZUORFA",
      "inReplyTo": [
        "yjetjodyYNLBto8YDnwLkEc89W09rOuEivbnxKGfHLQ",
        1,
        "6MaXLsvHITc8xzIQp8BRz_7w_UNQL1n7a6DWysTW5T0"
      ],
      "lastSeen": [
        "Ygye96VUeBhTO6U3D0ZllYsSwSdKB3SiRJgxAb3cdAo"
      ],
      "disposition": "reaction",
      "mimetype": "text/plain;charset=utf-8",
      "encoding": "none"
      "body": "❤",
    },

    {
      "type": "text",
      "start": "2022-02-08T22:14:03.008-00:00",
      "duration": 0,
      "parties": [
        0, 1, 2
      ],
      "originator": 2,
      "messageId": "rF4iS5BcJ-aFfisoFDCrnTE9pZZBGCxGCoXGJ9VXnck",
      "lastSeen": [
        "2YF0k6blW9ZbJ8sgXJgMCTJOlofazdKwVRvf6ZUORFA"
      ],
      "mimetype": "text/markdown;variant=GFM",
      "encoding": "none"
      "body":
        "Kudos to [@Alice Smith](mimi://example.com/alice-smith) for
making the release happen!",
    },

    {
      "type": "text",
      "start": "2022-02-08T22:14:08.621-00:00",
      "duration": 0,
      "parties": [
        0, 1, 2
      ],
      "originator": 1,
      "messageId": "-GqJXuoaJu1Xosu-N8nt6NWzA2RLmIRY0q9jf_kvWkM",
      "replaces": "Ygye96VUeBhTO6U3D0ZllYsSwSdKB3SiRJgxAb3cdAo",
      "inReplyTo": [
        "yjetjodyYNLBto8YDnwLkEc89W09rOuEivbnxKGfHLQ",
        1,
        "6MaXLsvHITc8xzIQp8BRz_7w_UNQL1n7a6DWysTW5T0"
      ],
      "messageId": "-GqJXuoaJu1Xosu-N8nt6NWzA2RLmIRY0q9jf_kvWkM",
      "lastSeen": [
        "2YF0k6blW9ZbJ8sgXJgMCTJOlofazdKwVRvf6ZUORFA",
        "rF4iS5BcJ-aFfisoFDCrnTE9pZZBGCxGCoXGJ9VXnck"
      ],
      "mimetype": "text/markdown;variant=GFM",
      "encoding": "none"
      "body": "Right on! _Congratulations_ y'all"
    },

    {
      "type": "text",
      "start": "2022-02-08T22:14:08.621-00:00",
      "duration": 0,
      "parties": [
        0, 1, 2
      ],
      "originator": 1,
      "messageId": "8cqo1B9IA6nnB63oPEnmWxIqr7uk7WvQTBgVVlD_Q2c",
      "replaces": "Ygye96VUeBhTO6U3D0ZllYsSwSdKB3SiRJgxAb3cdAo",
      "inReplyTo": [
        "yjetjodyYNLBto8YDnwLkEc89W09rOuEivbnxKGfHLQ",
        1,
        "6MaXLsvHITc8xzIQp8BRz_7w_UNQL1n7a6DWysTW5T0"
      ],
      "lastSeen": [
        "-GqJXuoaJu1Xosu-N8nt6NWzA2RLmIRY0q9jf_kvWkM"
      ]
    },

    {
      "type": "text",
      "start": "2022-02-08T22:14:10.389-00:00",
      "duration": 0,
      "parties": [
        0, 1, 2
      ],
      "originator": 2,
      "messageId": "bYZ2NHaryq0WS2pq5IE9fMc4zltUyE6WrKJ7bNB5tMc",
      "replaces": "Ygye96VUeBhTO6U3D0ZllYsSwSdKB3SiRJgxAb3cdAo",
      "inReplyTo": [
        "yjetjodyYNLBto8YDnwLkEc89W09rOuEivbnxKGfHLQ",
        1,
        "6MaXLsvHITc8xzIQp8BRz_7w_UNQL1n7a6DWysTW5T0"
      ],
      "lastSeen": [
        "8cqo1B9IA6nnB63oPEnmWxIqr7uk7WvQTBgVVlD_Q2c"
      ],
      "disposition": "reaction"
    },

    {
      "type": "text",
      "start": "2022-02-08T22:49:06.227-00:00",
      "duration": 0,
      "parties": [
        0, 1, 2
      ],
      "originator": 0,
      "messageId": "GkblzkXJxyar0lTgkfcMQ0wo8qpbcU0MqtTg-gM1FiY",
      "expiring": "2022-02-08T22:59:06.227-00:00"
      "lastSeen": [
        "bYZ2NHaryq0WS2pq5IE9fMc4zltUyE6WrKJ7bNB5tMc"
      ],
      "status": "expired"
    },

    {
      "type": "text",
      "start": "2022-02-08T22:53:41.134-00:00",
      "duration": 0,
      "parties": [
        0, 1, 2
      ],
      "originator": 1,
      "messageId": "Tdt8UCUl1ugSLRmuObvib_4bvu_Hz3NSwaYuy-TtET4",
      "lastSeen": [
        "GkblzkXJxyar0lTgkfcMQ0wo8qpbcU0MqtTg-gM1FiY"
      ],
      "disposition": "attachment",
      "language": "en",
      "ExternalPart": {
        "mimetype": "video/mp4",
        "url": "https://example.com/storage/8ksB4bSrrRE.mp4",
        "size": 708234961,
        "description": "2 hours of key signing video",
        "filename": "bigfile.mp4",

        "encAlg": 1,
        "key": "aZISOs306M7n_3csR-J1cw",
        "nonce": "5VM0QcZI3PNmnquV6CUgxg",
        "aad": "",
        "contentHash": "sha256:OczZYpW_L0B1DsMSLJqL2jTRJKoj7fTUJ2jhnX70-00"
      }
    },

    {
      "type": "text",
      "start": "2022-02-08T22:54:09.972-00:00",
      "duration": 0,
      "parties": [
        0, 1, 2
      ],
      "originator": 2,
      "messageId": "KgDTc29ZP4YyYmXtaYyxpZNrnigjvCjrCYuLTz19zWc",
      "lastSeen": [
        "Tdt8UCUl1ugSLRmuObvib_4bvu_Hz3NSwaYuy-TtET4"
      ],
      "disposition": "session",
      "ExternalPart": {
        "url": "https://example.com/join/12345",
        "description": "Join the Foo 118 conference"
      }
    },

    {
      "type": "text",
      "start": "2022-02-08T22:57:14.084-00:00",
      "duration": 0,
      "parties": [
        0, 1, 2
      ],
      "originator": 0,
      "messageId": "sD19bKRmu4mA9SHznfQOLQQzKnx1Q4mEtSNQhvzqCZw",
      "lastSeen": [
        "KgDTc29ZP4YyYmXtaYyxpZNrnigjvCjrCYuLTz19zWc"
      ],
      "disposition": "render",
      "partIndex": 0,
      "MultiPart": {
        "partSemantics": "chooseOne",
        "parts": [
          "Part": {
            "disposition": "render",
            "language": "en",
            "partIndex": 1,
            "mimetype": "text/markdown;variant=GFM",
            "encoding": "none"
            "body": "Hello!"
          },
          "Part": {
            "disposition": "render",
            "language": "fr",
            "partIndex": 2,
            "mimetype": "text/markdown;variant=GFM",
            "encoding": "none"
            "body": "Bonjour!"
          }
        ]
      }
    }
  ]
}
~~~

## MIMI VCON with an Attachment

~~~ json
{
  "vcon": "0.0.1",
  "room": {
    "id": "mimi://example.com/r/engineering_team",
    "name": "Engineering Team",
  }
  "parties": [
    {
      "imUri": "mimi://example.com/u/alice-smith",
      "name": "Alice Smith",
      "role": "moderator",
      "thumbprint": "TODOFIXDZXCog_FfQp-xLemZkD5GKB9H7Z-Y41O4jEw"
    },
    {
      "imUri": "mimi://example.com/u/bob-jones",
      "name": "Bob Jones",
      "role": "member",
      "thumbprint": "TODOFIXYUFE7bV9pXAHZHi5bwSWzLJqjncevs9aLKDg"
    },
    {
      "imUri": "mimi://example.com/u/cathy-washington",
      "name": "Cathy Washington",
      "role": "member",
      "thumbprint": "TODOFIXzH3EeOGbI0-oiDGTXlgKmkMzQyQ2W_Y-TB1U"
    }
  ],
  "dialog": [
    {
      "type": "text",
      "start": "2022-02-08T22:53:41.134-00:00",
      "duration": 0,
      "parties": [
        0, 1, 2
      ],
      "originator": 1,
      "messageId": "Tdt8UCUl1ugSLRmuObvib_4bvu_Hz3NSwaYuy-TtET4",
      "lastSeen": [
        "GkblzkXJxyar0lTgkfcMQ0wo8qpbcU0MqtTg-gM1FiY"
      ],
      "disposition": "attachment",
      "language": "en",
      "ExternalPart": {
        "mimetype": "video/mp4",
        "url": "https://example.com/storage/8ksB4bSrrRE.mp4",
        "size": 708234961,
        "description": "2 hours of key signing video",
        "filename": "bigfile.mp4",
        "contentHash": "sha256:OczZYpW_L0B1DsMSLJqL2jTRJKoj7fTUJ2jhnX70-00",
        "cached": true
      }
    },
  ],
  "attachments": [
    {
      "start": "2022-02-08T22:53:41.134-00:00",
      "party": 1,
      "contentHash": "sha256:OczZYpW_L0B1DsMSLJqL2jTRJKoj7fTUJ2jhnX70-00",
      "dialogObjectRef":
  "mid:Tdt8UCUl1ugSLRmuObvib_4bvu_Hz3NSwaYuy-TtET4:0@anonymous.invalid",
      "mimetype": "video/mp4",
      "filename": "bigfile.mp4",
      "encoding": "base64url",
      "body": "Ma0hHSr0f_iUk_RSShTgtY...nSQbZEip5danJYQqsvwWQ"
    }
  ]
}
~~~

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
