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
a mandatory list of parties and dialogs, a new top-level room object,
and optionally attachment objects.

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
- `inReplyTo` is an array of three values: the base64url encoding of the MIMI content messageId of the message to which this message is replying (or reacting), the hash algorithm expressed as an integer (SHA-256 is 1), and the base64url hash of the decrypted mimi-content. It is optional if empty.
- `lastSeen` is an array of base64url-encoded message ids. It is mandatory.
- `mimiExtensions` is a object (map) containing MIMI extensions. It is optional.

VCON typically expresses content using the body, encoding, and mimetype
fields. In order to preserve this convention we use these fields directly
for a MIMI SinglePart structure, but use new MultiPart and ExternalPart
structure for those MIMI structures. For a SinglePart these two fields could
be present.

- `disposition` - optional if set to the default value ("render")
- `language` - optional if absent

If there is only a single part its "partIndex" is 0.


## MultiPart

- `partSemantics` is one of "chooseOne", "singleUnit", or "processAll". It is mandatory.
- `parts` is an array of `Part`s. It is mandatory.

## Part

- `disposition` - optional if set to the default value ("render")
- `language` - optional if absent
- `partIndex` is an unsigned integer. It is mandatory in a Part object.
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
- cached is a boolean. It is mandatory if it is true, which means that a
copy of the external content is available in a vcon attachment object
(see {#attachments}).
- contentHash is the base64url encoded string of the hash of the external
content, prefixed with the name of the hash algorithm and a colon (for
example "sha256:"). contentHash is optional unless the external content has
been included in a vcon attachment object.

ExternalPart has several fields for the decryption of the
referenced content. These can be omitted once the content has been
downloaded, decrypted, verified, and included in the VCON attachments array.
All of them are base64url encoded strings. Otherwise they are mandatory if
present in the MIMI content.

- encAlg
- key
- nonce
- aad

## Changes to the room

Changes to the room metadata or participation should be accompanied by
a new dialog type:

- room metadata changes
- participant identity change
- participants joining and leaving
- participants adding new clients
- moderation events?

**TODO** add additional fields

## party_event_type events

When there is a change to the parties represented in a room, the
`party_event_type.event` is added.

~~~
party_event_type.event /= "add" / "welcome" / "leave" / "remove" / "ban"
~~~

- add: user added herself directly
- welcome: user added by someone else
- leave: user leaves of their own accord
- remove: user is removed by another user
- ban: user is banned from the group

# Attachments

An attachment consists of the following fields:

- start: is the time when the attachment was downloaded. it is mandatory.
- party: is the party that downloaded the attachment. it is mandatory.
- contentHash: is the base64url encoded hash using the hash name prefixed with a colon before the hash (ex: "sha256:"). It is mandatory.
- dialogObjectRed is a string consisting of: "mid:" (representing the message ID URI), the messageId of the message in the dialog object, a colon, the partIndex of the Part (or "0" if the ExternalPart is at the top level), and the string "@anonymous.invalid"

mimetype, filename, encoding, and body are as defined in vcon and are all mandatory.


# Examples

## MIMI examples as a VCON

The example vcon consists of the example messages from Section 5 of the MIMI content specification plus a single multipart message.

~~~ json
{::include examples/mimi-examples.json}
~~~

## MIMI VCON with an Attachment

This example vcon consists of a single message in a dialog which references
an attachment.

~~~ json
{::include examples/attachment.json}
~~~

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# CDDL changes

**TODO**

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
