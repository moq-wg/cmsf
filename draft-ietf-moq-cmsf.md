---
title: "CMSF- a CMAF compliant implementation of MOQT Streaming Format"
category: info

docname: draft-ietf-moq-cmsf-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Web and Internet Transport"
workgroup: "Media Over QUIC"
keyword:
 - moq
 - moqt
 - MSF
 - CMAF
venue:
  group: "Media Over QUIC"
  type: "Working Group"
  mail: "moq@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/moq/"
  github: "moq-wg/cmsf"
  latest: "https://moq-wg.github.io/cmsf/draft-wilaw-moq-cmsf.html"

author:
  -
    ins: W. Law
    name: "Will Law"
    organization: Akamai
    email: wilaw@akamai.com

normative:
  MoQTransport: I-D.draft-ietf-moq-transport-18
  MSF:  I-D.draft-ietf-moq-msf-01
  LOC: I-D.draft-ietf-moq-loc-02
  CMAF:
    author:
      - name: International Organization for Standardization
        org: ISO
    title: "Information technology — Multimedia application format (MPEG-A) — Part 19:
            Common media application format (CMAF) for segmented media"
    date: 2021-10
  BASE64: RFC4648
  JSON: RFC8259
  HTTP: RFC9110
  CENC:
    author:
      - name: International Organization for Standardization
        org: ISO
    title: "Information technology — MPEG systems technologies — Part 7:
            Common encryption in ISO base media file format files"
    date: 2024-08
    seriesinfo:
      ISO/IEC: 23001-7:2024

informative:
  DASH:
    author:
      - name: International Organization for Standardization
        org: ISO
    title: "Information technology — Dynamic adaptive streaming over HTTP (DASH) —
            Part 1: Media presentation description and segment formats."
    date: 2022-08
    edition: 5
    seriesinfo:
      ISO/IEC: 23009-1:2022
  DASHIF-ECCP:
    author:
      - name: DASH Industry Forum
        org: DASH-IF
    title: "DASH-IF Implementation Guidelines: Encryption and Content Protection (ECCP)"
    target: https://dashif.org/docs/DASH-IF-ECCP-v1.0.0.pdf
    date: 2023
  EME:
    author:
      - name: W3C
        org: W3C
    title: "Encrypted Media Extensions"
    target: https://www.w3.org/TR/encrypted-media/
    date: false
  MEDIA-CAPABILITIES:
    author:
      - name: W3C
        org: W3C
    title: "Media Capabilities"
    target: https://www.w3.org/TR/media-capabilities/
    date: false
  HDCP:
    author:
      - name: Digital Content Protection LLC
        org: Digital Content Protection LLC
    title: "High-bandwidth Digital Content Protection (HDCP) Specifications"
    target: https://www.digital-cp.com/hdcp-specifications
    date: false
  WIDEVINE:
    author:
      - name: Google
        org: Google
    title: "Widevine DRM"
    target: https://developers.google.com/widevine/drm/overview
    date: false
  PLAYREADY:
    author:
      - name: Microsoft
        org: Microsoft
    title: "PlayReady Documentation"
    target: https://learn.microsoft.com/en-us/playready/
    date: false
  FAIRPLAY:
    author:
      - name: Apple
        org: Apple
    title: "FairPlay Streaming"
    target: https://developer.apple.com/streaming/fps/
    date: false

...

--- abstract

This document updates MOQT Streaming Format by defining optional syntax and semantics
for carrying CMAF-packaged media.


--- middle

# Introduction

CMAF compliant MOQT Streaming Format (CMSF) is a media format designed to deliver CMAF [CMAF]
and LOC [LOC] compliant media content over MOQ Transport (MOQT) [MoQTransport]. CMSF extends
MSF and retains all the scope, capabilities and features of MSF including the catalog
format, timeline, ABR switching and LOC support. CMSF is targeted at real-time and
interactive levels of live latency, as well as VOD content.

This document describes version 1 of the CMSF streaming format.

# MSF Extension
All of the specifications, requirements, and terminology defined in [MSF] apply to
implementations of this extension unless explicitly noted otherwise in this document.

# CMAF Packaging

## Initialization headers {#initheaders}
A CMAF header is a sequence of CMAF constrained ISO BMFF boxes that do not reference any
media samples, but are associated with a CMAF track and are necessary for initializing
the decoding of the subsequent CMAF fragments.

The header for a given MOQT Track MUST be added to the catalog by generating an initDataList
entry with a "type" of "inline", a "data" value of the header encoded using [BASE64] and an
"id" which is unique across the initDataList items. That "id" value is then inserted as the
value of an "initRef" property in all tracks which use that header for initialization.

## Switching sets and tracks
This specification defines a direct mapping between CMAF Tracks ( [CMAF] Section 3.2.1) and
MOQT tracks ([MoQTransport] Section 2.3).

CMAF switching sets are a set of one or more CMAF tracks (3.2.1), where each track is an
alternative encoding of the same source content and are constrained to enable seamless
track switching (3.3.9).

Each CMAF track in a switching set MUST be transmitted as a separate MOQT Track. The
catalog entry for each of these tracks in the switching set MUST carry a Alternate group
(altGroup) key with a common value.

The MOQT Group numbers within these switching set tracks MUST be media time-aligned.
Mandating the track being media time-aligned requires that the presentation time of the
first media sample contained within the first MOQT Object of each MOQT Group is identical.

## Object Packaging

The payload of each Object is subject to the following requirements:

* MUST contain at least one Movie Fragment Box (moof) followed by a Media Data
  Box (mdat). This is equivalent to requiring that each Object hold at least one CMAF
  Chunk. The Media Fragment Box (moof) MUST contain a Movie Fragment Header Box
  (mfhd) and Track Box (trak) with a Track ID (track_ID) matching a Track Box in the
  initialization fragment.
* MAY contain multiple successive CMAF Chunks.
* MUST contain a single track.

## Group Packaging

Each MOQT Group

* MUST begin with an Object containing a stream access point (SAP) type 1 or 2.
* MUST contain one or more contiguous independently coded sequences of media samples.
* The Group boundary MUST align with a CMAF Fragment boundary. CMAF Fragments and CMAF
  Chunks MUST NOT span Groups.

## Catalog description

### CMAF packaging type
This specification extends the allowed packaging values defined in [MSF]
to include one new entry, as defined in Table 1 below:

| Name              |   Value         |      Reference        |
|:==================|:================|:======================|
| CMAF              | cmaf            | This RFC              |

Every Track entry in a CMSF catalog carrying CMAF-packaged media data MUST declare a
"packaging" type value of "cmaf".

### Max SAP starting types
This specification adds two track-level catalog fields, as defined in Table 2 below:

| Field                       |  Name                  |           Definition      |
|:============================|:=======================|:==========================|
| Max Group SAP starting type | maxGrpSapStartingType  | {{maxgrpsapstartingtype}} |
| Max Object SAP starting type| maxObjSapStartingType  | {{maxobjsapstartingtype}} |

#### Max Group SAP starting type {#maxgrpsapstartingtype}
Location: T    Required: Optional   JSON Type: Number

A number indicating the maximum SAP type the MOQT Groups in the track start with.

#### Max Object SAP starting type {#maxobjsapstartingtype}
Location: T    Required: Optional   JSON Type: Number

A number indicating the maximum SAP type the MOQT Objects in the track start with.

## Event Timelines

### SAP Type timeline {#saptypetimeline}
CMSF defines a special instance of an Event Timeline track, termed the SAP Type timeline
track. Its purpose is to convey information about the distribution of Stream Access Point
types and their associated Earlist Presentation Times.

In the catalog, the SAP-type timeline track MUST include a 'packaging' value of 'eventtimeline"
and MUST include an 'eventType' value of 'org.ietf.moq.cmsf.sap'.

In the SAP Type timeline JSON payload:

* The index reference MUST be 'l' for Location
* The data field is a JSON Array containing two integers. The first integer defines SAP type
  with an allowed value of 0,1,2 or 3. The value 0 indicates that the Object does not start
  with an ISOBMFF stream access point. The value equal to 1, 2, or 3 indicates that the Object
  begins with a stream access point of SAP type 1, 2, or 3, respectively. When the Object is
  the first Object in the Group, the value MUST be equal to 1 or 2. The second integer defines
  the earliest media presentation timestamp, rounded to the nearest millisecond, of all media
  samples in the Object defined by the Location of that record.

### SAP-type timeline track example
This shows an example of 30-fps HEVC-encoded content, in which each 4s Group beings with
SAP-type 2 (i.e., the first picture in the Group is an IDR picture, while there may be one or more
pictures in the Group following the IDR picture in decoding order but preceding it in output
order). After 2 seconds in each Group, there is a SAP-type 3, i.e., a CRA picture, which
is associated with one or more Random Access Skipped
Leading (RASL) pictures. A small buffer of frames (10 frames at 30 fps) is skipped/discarded
(RASL pictures) when the streaming session starts from the SAP-type 3 location. In this example,
the EPT is the presentation time of the first picture after the RASL pictures in decoding order;
all pictures after the RASL pictures can be fully correctly decoded and are thus presentable
when the streaming session starts from the SAP-type 3 location. Note that if the streaming session
starts from the start of the Group, then these RASL pictures can be fully correctly decoded and are
thus presentable.

~~~json
[
    {
        "l": [0,0],
        "data": [2,0]
    },
    {
        "l": [0,60],
        "data": [3,2100]
    },
    {
        "l": [1,0],
        "data": [2,4000]
    },
    {
        "l": [1,60],
        "data": [3,6100]
    }
]
~~~

# URL usage

CMSF inherits MSF semantics for the construction of URLs. In
particular, the fragment type of "msf" MUST be used when referencing CMSF
catalogs. An example CMSF URL is shown below:

moqt://example.com/server/config?a=1#msf:customer-123--catalog&b=2

# Content Protection {#contentprotection}

CMSF supports content protection using Common Encryption [CENC] for
CMAF-packaged media. This enables interoperability with existing DRM ecosystems
by reusing the signaling model established by DASH [DASH] and the DASH-IF
Encryption and Content Protection (ECCP) guidelines [DASHIF-ECCP].

Content protection in CMSF differs from the encryption scheme defined in
[MSF] Section 4.3. While MSF defines an end-to-end encryption mechanism
using MoQ Secure Objects for LOC-packaged content, CMSF uses ISO Common
Encryption [CENC] applied at the CMAF media layer. In CMSF, the media
samples within CMAF chunks are encrypted as specified by [CENC], and the
DRM signaling is carried in the catalog rather than in per-object headers.

A key advantage of CENC-based content protection is that decryption can be
delegated to a Content Decryption Module (CDM) operating at a deeper system
level, potentially in hardware or a trusted execution environment. This
enables robust content protection through commercial DRM systems such as
Widevine, PlayReady, and FairPlay Streaming, where the decryption keys and
decrypted media are handled within a secure pipeline that is not accessible
to the application layer. This hardware-level protection is a requirement
for high-value content distribution and is not achievable with
application-layer encryption schemes alone.

## Content Protection catalog fields

Content protection information is signaled in the catalog at two levels:
a root-level `contentProtections` array containing DRM system descriptions,
and track-level references to those descriptions.

### Content Protections {#contentprotections}
Location: R    Required: Optional   JSON Type: Array

A JSON array of Content Protection objects at the root level of the catalog.
Each object describes a single DRM system configuration. Content protection
information MUST NOT be duplicated at the track level; all tracks reference
the root-level entries.

Each Content Protection object contains the following fields:

#### Reference ID {#refid}
Location: CP    Required: Required   JSON Type: String

A unique identifier for this content protection entry. Track entries
reference this value via the `contentProtectionRefIDs` field.

#### Default KIDs {#defaultkid}
Location: CP    Required: Required   JSON Type: Array of Strings

An array of default Key IDs (KIDs) expressed as UUID strings in the format
"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx". These correspond to the
default_KID values defined in [CENC] Section 5. When multiple KIDs are
present, each identifies a key used to encrypt a different set of samples
(e.g., separate keys for different track types).

#### Scheme {#scheme}
Location: CP    Required: Required   JSON Type: String

The Common Encryption protection scheme as defined in [CENC]. Allowed
values are:

| Name  | Value | Description                                         |
|:======|:======|:====================================================|
| CENC  | cenc  | AES-CTR full sample and sub-sample encryption       |
| CBCS  | cbcs  | AES-CBC pattern-based encryption with subsample     |

The RECOMMENDED scheme for CMSF is "cbcs" as it provides better
compatibility with hardware decoders and aligns with DASH-IF ECCP
recommendations [DASHIF-ECCP].

#### DRM System {#drmsystem}
Location: CP    Required: Required   JSON Type: Object

An object describing the DRM system associated with this content protection
entry. It contains the following fields:

##### System ID {#systemid}
Location: DS    Required: Required   JSON Type: String

The DRM System ID expressed as a UUID string. Well-known system IDs include:

| DRM System   | System ID                            |
|:=============|:=====================================|
| Widevine     | edef8ba9-79d6-4ace-a3c8-27dcd51d21ed |
| PlayReady    | 9a04f079-9840-4286-ab92-e65be0885f95 |
| FairPlay     | 94ce86fb-07ff-4f43-adb8-93d2fa968ca2 |
| ClearKey     | 1077efec-c0b2-4d02-ace3-3c1e52e2fb4b |

The ClearKey system ID follows the W3C EME specification and corresponds
to the DASH-IF ECCP "Explicit Clear Key Content Protection" mechanism
[DASHIF-ECCP].

##### License URL {#laurl}
Location: DS    Required: Optional   JSON Type: Object

An object containing the URL of the license acquisition service. The object
has the following fields:

* `url` (String, Required): The URL of the license server.
* `type` (String, Optional): The license protocol type (e.g., "EME-1.0").

##### Certificate URL {#certurl}
Location: DS    Required: Optional   JSON Type: Object

An object containing the URL of the DRM certificate service. This field is
REQUIRED for DRM systems that require a server certificate (e.g., FairPlay
Streaming). The object has the following fields:

* `url` (String, Required): The URL of the certificate server.
* `type` (String, Optional): The MIME type of the certificate resource
  (e.g., "application/pkcs7-mime", "application/x-x509-ca-cert"). This
  corresponds to the certType attribute defined in [DASHIF-ECCP].

##### Authorization URL {#authzurl}
Location: DS    Required: Optional   JSON Type: Object

An object containing the URL of an authorization service. The object
has the following fields:

* `url` (String, Required): The URL of the authorization server.
* `type` (String, Optional): The authorization protocol type.

##### PSSH {#pssh}
Location: DS    Required: Optional   JSON Type: String

A Base64-encoded [BASE64] Protection System Specific Header (PSSH) box
as defined in [CENC] Section 8.1. The PSSH box contains DRM
system-specific initialization data needed by the client to acquire
a license. This field SHOULD be present for DRM systems that require
PSSH data (e.g., Widevine, PlayReady).

##### Robustness {#robustness}
Location: DS    Required: Optional   JSON Type: String

A string indicating the minimum robustness level required by the DRM system.
The interpretation of this value is DRM system-specific.

##### Output Protection {#outputprotection}
Location: DS    Required: Optional   JSON Type: Object

An object with an "hdcp" String giving the minimum HDCP version required to
output content decrypted with the keys of this entry. This field is
informative to the subscriber. Enforcement is performed by the DRM system
through the license.

### Content Protection Reference IDs {#contentprotectionrefids}
Location: T    Required: Optional   JSON Type: Array of Strings

An array of Reference ID strings (see {{refid}}) identifying which content
protection entries from the root-level `contentProtections` array apply to
this track. When this field is present, the track content is encrypted
using Common Encryption [CENC] and the subscriber MUST acquire appropriate
licenses before decryption.

When this field is absent, the track content is not protected by Common
Encryption.

## Initialization data for protected tracks

For protected CMAF tracks, the initialization data (carried in the catalog
`initDataList` array (as defined in {{initheaders}}) MUST include the Protection
Scheme Information Box ('sinf') containing the Scheme Type Box ('schm')
and Scheme Information Box ('schi') with the Track Encryption Box ('tenc')
as specified in [CENC] Section 6. This enables the subscriber to determine
the encryption parameters (default_isProtected, default_Per_Sample_IV_Size,
default_KID) from the initialization segment.

## ClearKey content protection

For testing and development scenarios, CMSF supports ClearKey content
protection using the W3C EME ClearKey mechanism. ClearKey uses the
Common System ID "1077efec-c0b2-4d02-ace3-3c1e52e2fb4b" and follows
the DASH-IF ECCP Explicit Clear Key Content Protection (ECCP) model
[DASHIF-ECCP].

When using ClearKey:

* The `systemID` MUST be set to "1077efec-c0b2-4d02-ace3-3c1e52e2fb4b".
* The `laURL` field SHOULD contain the URL of a ClearKey license server
  that implements the EME ClearKey protocol.
* The optional `pssh` field SHOULD contain a Base64-encoded PSSH box with
  version 1, the Common System ID, and the KID(s) in the KID list.

# Capability Preflight {#preflightworkflow}

Some content is only delivered to subscribers that meet requirements such as a
decoding capability, display characteristics, or a minimum content protection
robustness. Capability preflight allows a publisher to learn, and where
possible verify, these capabilities before it issues authorization for the
tracks that depend on them, so that a subscriber is only authorized for tracks
it is able, and permitted, to consume.

Without preflight, the first check that inspects the subscriber itself happens
when the subscriber tries to use media it has already been authorized for and
received, for example when it decodes the media or, for protected content,
when it acquires a license. Preflight moves that check ahead of track access,
so that:

- relays only deliver tracks to subscribers that have met the publisher's
  requirements;
- track authorization and any later per-subscriber checks are derived from
  the same decision;
- subscribers are not authorized for tracks that they will be unable to use,
  which would otherwise fail at playback time.

{{fig-preflight-order}} compares the two orderings.

~~~ ascii-art
Without preflight:

  catalog --> token --> SUBSCRIBE --> media --> use (decode, license)
                                                 ^
                                     first check of the subscriber

With preflight:

  catalog --> preflight --> scoped token --> SUBSCRIBE --> media --> use
                  ^                              ^
         subscriber check            relay enforces token scope
~~~
{: #fig-preflight-order title="Order of subscriber checks with and without preflight"}

The preflight mechanism is independent of the capabilities it checks. What is
checked is defined by probe types ({{preflightprobetypes}}). This document
defines a content protection probe type and probe types for decoding, display
and clock capabilities. Other specifications can define further probe types.

Preflight results are an input to an authorization decision. Access is enforced
by the authorization scheme ([MSF] Section "Authorization") and, for protected
content, by the content protection system. Preflight replaces neither.

## Preflight catalog fields {#preflightcatalogfields}

This section defines catalog fields in addition to those defined in [MSF].

### Preflight {#preflight}
Location: R    Required: Optional   JSON Type: Object

An object signaling that the publisher offers a capability preflight through
which subscribers obtain authorization for tracks marked with the Preflight
required field ({{preflightrequired}}). The object has the following fields:

* requestTrack (String, Required): the Track Name of an event timeline track
  ([MSF] Section "Event Timeline track"), declared in the tracks array, with an
  eventType of "org.ietf.moq.cmsf.preflight.request".
* submitUri (String, Conditional): an "https" URI to which the subscriber
  submits its preflight response.
* submitTrack (String, Conditional): the Track Name of a publish track ([MSF]
  Section "Publish tracks") with an eventType of
  "org.ietf.moq.cmsf.preflight.response" on which the subscriber publishes its
  preflight response.
* timeout (Number, Optional): the maximum time, in milliseconds, that a
  subscriber SHOULD wait for a preflight decision after submitting its response.
* fallbackCatalog (String, Optional): the Track Name of a catalog track whose
  tracks the subscriber MAY use if it does not support preflight, or if the
  preflight fails or times out.

Exactly one of submitUri or submitTrack MUST be present. The values of
requestTrack, submitUri, submitTrack and fallbackCatalog MAY contain variables
([MSF] Section "Variable Substitution"), allowing a single cached catalog to
direct each subscriber to a per-session preflight exchange.

### Preflight required {#preflightrequired}
Location: T    Required: Optional   JSON Type: Boolean

When TRUE, authorization for this track is obtained through the preflight
exchange ({{preflightexchange}}). A subscriber MUST NOT send SUBSCRIBE or FETCH
for such a track until it has received a preflight decision that lists the
track as authorized. This field MUST NOT be included if it is FALSE.

This field MAY be set on a track with a packaging value of "catalog", in which
case it applies to that catalog track and to every track the referenced catalog
declares.

A catalog that declares a track with this field set to TRUE MUST contain the
Preflight field ({{preflight}}), or be referenced by a catalog that does.

## Preflight exchange {#preflightexchange}

The preflight exchange proceeds as follows:

1. The subscriber obtains a catalog containing the Preflight field
   ({{preflight}}).
2. The subscriber subscribes to the request track and receives a preflight
   request ({{preflightrequest}}).
3. The subscriber executes the requested probes and submits a preflight
   response ({{preflightresponse}}) to the submitUri, or publishes it on the
   submitTrack.
4. The publisher, or a service acting on its behalf, evaluates the response and
   returns a preflight decision ({{preflightdecision}}).
5. The subscriber subscribes to the authorized tracks, presenting the
   authorization provided in the decision as described in [MSF] Section
   "Presenting Authorization".

{{fig-preflight-exchange}} shows the exchange followed by track access.

~~~ ascii-art
Subscriber                     Relay                 Preflight
                                                      Service
     |                           |                       |
     | SUBSCRIBE catalog         |                       |
     |-------------------------->|                       |
     | catalog with preflight    |                       |
     |<--------------------------|                       |
     | SUBSCRIBE request track   |                       |
     |-------------------------->|                       |
     | preflight request: nonce, probes                  |
     |<--------------------------------------------------|
[run probes]                     |                       |
     | preflight response: results, evidence             |
     |-------------------------------------------------->|
     |                           |          [verify, decide]
     | decision: authorizedTracks, token                 |
     |<--------------------------------------------------|
     | SUBSCRIBE authorized track, token                 |
     |-------------------------->|                       |
     |                     [scope check]                 |
     | media objects             |                       |
     |<--------------------------|                       |
~~~
{: #fig-preflight-exchange title="Preflight exchange followed by track access"}

A subscriber that does not support preflight, or whose preflight fails or times
out, MUST NOT subscribe to tracks for which Preflight required
({{preflightrequired}}) is TRUE. Such a subscriber MAY use the tracks declared
in the fallbackCatalog, if one is present.

## Preflight request {#preflightrequest}

A preflight request is an event timeline record ([MSF] Section "Event Timeline
data format") published on the request track. The eventType of the request
track MUST be "org.ietf.moq.cmsf.preflight.request". The record SHOULD use a
Location index reference. The data Object contains the following fields:

* nonce (String, Required): a value that is unique to this request and
  unpredictable to the subscriber. The value MUST conform to the restrictions
  on variable values defined in [MSF] Section "Variable Syntax", so that probes
  can embed it in the evidence they generate.
* expires (Number, Required): the wallclock time, in milliseconds since the
  Unix epoch, after which the publisher will not accept a response to this
  request.
* probes (Object, Required): an Object whose keys are probe type identifiers
  ({{preflightprobetypes}}) and whose values hold the parameters for that probe
  type, as defined by the specification of the probe type.

A record on the request track whose data Object contains a "decision" field is
a preflight decision ({{preflightdecision}}), not a request.

A subscriber MUST NOT fail the exchange because a request contains a probe type
it does not understand. It MUST report such probe types as unsupported in its
response.

## Preflight response {#preflightresponse}

A preflight response is a JSON {{JSON}} Object with the following fields:

* nonce (String, Required): the nonce from the request being answered.
* results (Object, Required): an Object containing one key for each probe type
  present in the request. The value is the result Object defined by the
  specification of that probe type or, if the subscriber did not execute the
  probe, an Object containing the single field "unsupported" with the value
  TRUE.

When the Preflight field contains a submitUri, the subscriber sends the
response as the body of an HTTP {{HTTP}} POST request to that URI, with a media
type of "application/json". The method used to authenticate this request is
deployment-specific.

When the Preflight field contains a submitTrack, the subscriber publishes the
response as the data Object of an event timeline record on that publish track.

## Preflight decision {#preflightdecision}

A preflight decision is a JSON {{JSON}} Object with the following fields:

* nonce (String, Required): the nonce from the request being answered.
* expires (Number, Required): the wallclock time, in milliseconds since the
  Unix epoch, after which the decision and any authorization it carries are no
  longer valid.
* catalog (Object, Optional): an Object holding the name, and optionally the
  namespace, of the catalog track that declares the authorized tracks. If
  absent, the authorized tracks are declared in the catalog containing the
  Preflight field. If the named catalog track requires preflight, it MUST also
  be listed in authorizedTracks.
* authorizedTracks (Array, Required): an Array of Objects, each holding the
  name and, optionally, the namespace of a track the subscriber is authorized
  to subscribe to.
* deniedTracks (Array, Optional): an Array of Objects, each holding the name,
  optionally the namespace, and optionally a "reason" String, of a track the
  subscriber is not authorized to subscribe to. The reason is informative and
  is not intended for display to end users. This document defines the reason
  values "decode-not-supported", "display-not-supported",
  "hdr-not-supported", "robustness-not-met", "output-protection-not-met" and
  "policy". Other values MAY be used, and a subscriber MUST treat an unknown
  reason value as "policy".
* authorization (Object, Optional): an Object whose keys are authorization
  scheme identifiers ([MSF] Section "Authorization Info") and whose values are
  the tokens to present for the authorized tracks. For tracks listed in
  authorizedTracks, these tokens take precedence over any authInfo value
  declared for the same scheme in the catalog.
* recheck (Object, Optional): an Object holding a submitUri or a submitTrack to
  which the subscriber SHOULD submit a new preflight response if a probe result
  changes during the session.

When the response was submitted to a submitUri, the decision is returned as the
body of the HTTP response with a status code of 200. A publisher that denies
access to every track that requires preflight MAY instead return status code
403.

When the response was published on a submitTrack, the decision is published on
the request track as an event timeline record whose data Object contains a
single field, "decision", holding the decision Object. A decision that carries
authorization tokens MUST only be published on a track whose namespace is
unique to the subscriber session.

The subscriber MUST treat any track that requires preflight and is not listed
in authorizedTracks as denied. It MUST NOT send SUBSCRIBE or FETCH for denied
tracks. A publisher SHOULD scope the tokens in a decision to exactly the tracks
listed in authorizedTracks. A relay that rejects a request for a track outside
that scope does so with an insufficient-scope failure, which the subscriber
handles without retrying the same token.

## Token acquisition via preflight {#preflighttoken}

[MSF] leaves token acquisition out of scope. Preflight provides one optional,
in-band mechanism for it. A publisher MAY require subscribers to complete a
preflight before authorization is issued for certain tracks, identified by the
Preflight required field ({{preflightrequired}}). The authorization field of
the preflight decision ({{preflightdecision}}) then provides the token, which
the subscriber presents as described in [MSF] Section "Presenting
Authorization". Other token acquisition mechanisms remain available for tracks
that do not require preflight.

## Preflight probe types {#preflightprobetypes}

A probe type identifies a capability check, the parameters the publisher sends
in a request, and the result the subscriber returns. Probe types are listed in
the "CMSF Preflight Probe Types" registry ({{iana-preflight-probe-types}}).
Probe types that are not registered SHOULD use Reverse Domain Name Notation.

A specification defining a probe type defines its identifier, its request
value, its result value, and how, if at all, the publisher can verify the
result. Probe types whose results gate access to tracks SHOULD return evidence
that the publisher can verify independently of the subscriber, for example a
challenge signed by a trusted component of the subscriber platform that embeds
the request nonce.

This document defines the following probe types.

### Content protection probe {#cppreflight}

Identifier: "org.ietf.moq.cmsf.contentprotection"

This probe type lets a publisher learn, and verify, the content protection
capabilities of a subscriber before authorizing protected tracks. The license
server continues to enforce its own policy at license acquisition, so
protected tracks are checked twice: by the relay, through the token scope
derived from the preflight decision, and by the license server.

#### Request value {#cppreflight-request}

The request value is an Object with the following fields:

* contentProtections (Array, Required): an Array of Content Protection objects
  ({{contentprotections}}) describing canary keys. The drmSystem object of each
  entry MAY contain a robustnessLadder field (Array of Strings) in place of the
  robustness field, listing the robustness values to test in descending order
  of preference. An empty robustnessLadder indicates that the DRM system does
  not expose robustness values to the subscriber. In web user agents,
  robustness values are those used with the Encrypted Media Extensions
  {{EME}}.
* hdcp (Array of Strings, Optional): the HDCP {{HDCP}} versions for which the
  subscriber is to report output status, for example "1.4" and "2.2".
* encryptionSchemes (Array of Strings, Optional): the Common Encryption schemes
  to test, from "cenc" and "cbcs".
* canaryPlayback (Object, Optional): the name, optional namespace, and initRef
  of a short protected track that the subscriber decrypts and decodes to confirm
  that playback succeeds under the highest robustness granted.
* sessionTypes (Array of Strings, Optional): license session types to test,
  for example "temporary" and "persistent-license".

The canary initialization data of each entry MUST embed the preflight request
nonce in a field that is covered by the integrity protection of the DRM
system's license request. Examples are the content_id of a Widevine
{{WIDEVINE}} PSSH, the CUSTOMATTRIBUTES element of a PlayReady {{PLAYREADY}}
Header Object, and the content identifier of a FairPlay Streaming
{{FAIRPLAY}} key request. Canary keys MUST NOT be used to encrypt content.

In web user agents, the subscriber obtains each challenge by passing the
canary initialization data to generateRequest() on an EME {{EME}} media key
session created at the robustness value being tested, and returns the
resulting license request message without sending it to a license server.

{{fig-cp-nonce-binding}} shows how the nonce binds the evidence to the request.

~~~ ascii-art
       Subscriber                                   Preflight
     (browser + CDM)                                 Service
            |                                           |
            | request: nonce N, canary init data with N |
            |<------------------------------------------|
[generateRequest(canary)]                               |
[CDM signs license request]                             |
     [containing N]                                     |
            | response: challenge(N), granted levels    |
            |------------------------------------------>|
            |                                [verify CDM signature]
            |                               [N matches this request]
            |                                  [level, revocation]
            |                               [reject replay/mismatch]
~~~
{: #fig-cp-nonce-binding title="Binding content protection evidence to a preflight request"}

#### Result value {#cppreflight-result}

The result value is an Object with one key per refID from the request. Each
value is an Object with the following fields:

* supported (Boolean, Required).
* granted (Array of Strings, Optional): the robustness values from the
  robustnessLadder that the subscriber was able to obtain.
* challenges (Object, Optional): one key per granted robustness value tested,
  holding the Base64 {{BASE64}} encoded license request generated for the
  canary key. Publishers determine the actual security level of the subscriber
  from these values, not from the granted field.
* encryptionSchemes (Object, Optional): a Boolean per scheme tested.
* hdcp (Object, Optional): one key per HDCP version tested, holding a String
  status such as "usable" or "output-restricted". In web user agents these
  are the key status values defined by EME {{EME}}.
* canaryPlayback (Object, Optional): framesRendered and droppedFrames (Number)
  and keyStatus (String).
* sessionTypes (Object, Optional): a Boolean per session type tested.

The minimum HDCP version required for each entry is signaled in the catalog
with the outputProtection field ({{outputprotection}}).

#### Robustness values (informative) {#cppreflight-robustness}

The following table lists commonly used robustness values for Widevine
{{WIDEVINE}}, PlayReady {{PLAYREADY}} and FairPlay Streaming {{FAIRPLAY}}.
The authoritative definitions are those of each DRM system.

| DRM system | Robustness value(s)                     | Common description |
|:===========|:========================================|:===================|
| Widevine   | HW_SECURE_ALL, HW_SECURE_DECODE         | L1                 |
| Widevine   | HW_SECURE_CRYPTO                        | L2                 |
| Widevine   | SW_SECURE_DECODE, SW_SECURE_CRYPTO      | L3                 |
| PlayReady  | 3000                                    | SL3000             |
| PlayReady  | 2000                                    | SL2000             |
| FairPlay   | none exposed; level reported in request | Main, Baseline     |

### Decode probe {#decodeprobe}

Identifier: "org.ietf.moq.cmsf.decode"

The request value is an Array of Objects, each with the fields label (String,
Required), codec (String, Required, as defined in [MSF] Section "Codec"),
width, height, framerate and bitrate (Number, Optional), and hdr (Object,
Optional) holding transferFunction, colorGamut and metadataType Strings.

The result is an Object with one key per label. Each value is an Object with
the fields supported (Boolean, Required), and smooth and powerEfficient
(Boolean, Optional). In web user agents these values correspond to those
returned by the Media Capabilities API {{MEDIA-CAPABILITIES}}.

### Display probe {#displayprobe}

Identifier: "org.ietf.moq.cmsf.display"

The request value is an empty Object. The result is an Object with the fields
width and height (Number, Optional, in device pixels), dynamicRange (String,
Optional, one of "standard" or "high") and external (Boolean, Optional,
indicating that output is directed to an external or extended display).

### Clock probe {#clockprobe}

Identifier: "org.ietf.moq.cmsf.clock"

The request value is an empty Object. The result is an Object with the single
field clientTime (Number, Required), the subscriber wallclock time in
milliseconds since the Unix epoch at which the response was generated. The
publisher can use this to detect clock skew that would affect token expiry.

## Preflight example {#preflightexample}

This example shows a root catalog that offers a preflight. The catalog contains
no media tracks of its own. It references a tier catalog that requires
preflight, and a fallback catalog that does not. The session identifier `sid`
is resolved from the URI fragment ([MSF] Section "Variable Substitution").

~~~json
{
  "version": "draft-01",
  "generatedAt": 1746104606044,
  "preflight": {
    "requestTrack": "preflight-%sid%",
    "submitUri": "https://api.example.com/v1/preflight/%sid%",
    "timeout": 4000,
    "fallbackCatalog": "catalog-sd"
  },
  "tracks": [
    {
      "name": "preflight-%sid%",
      "namespace": "example.com/preflight/%sid%",
      "packaging": "eventtimeline",
      "eventType": "org.ietf.moq.cmsf.preflight.request",
      "mimeType": "application/json",
      "isLive": true
    },
    {
      "name": "catalog-premium",
      "packaging": "catalog",
      "preflightRequired": true
    },
    {
      "name": "catalog-sd",
      "packaging": "catalog"
    }
  ]
}
~~~

Preflight request record. The content protection probe asks for a Widevine
canary key whose PSSH carries the nonce in its content_id field:

~~~json
[
  {
    "L": [0, 0],
    "data": {
      "nonce": "n-7Qe2xK",
      "expires": 1746104636044,
      "probes": {
        "org.ietf.moq.cmsf.decode": [
          { "label": "uhd", "codec": "hvc1.2.4.L153.90",
            "width": 3840, "height": 2160, "framerate": 30,
            "bitrate": 16000000 }
        ],
        "org.ietf.moq.cmsf.display": {},
        "org.ietf.moq.cmsf.clock": {},
        "org.ietf.moq.cmsf.contentprotection": {
          "contentProtections": [
            {
              "refID": "wv-canary",
              "defaultKID": ["5f1e7c2a-93b4-4d61-8e0f-2a7c9b3d4e11"],
              "scheme": "cbcs",
              "drmSystem": {
                "systemID": "edef8ba9-79d6-4ace-a3c8-27dcd51d21ed",
                "robustnessLadder": [
                  "HW_SECURE_ALL", "SW_SECURE_CRYPTO"
                ],
                "pssh": "AAAAVnBzc2gAAAAA7e+LqXnWSs6jyCfc1R0h7Q..."
              }
            }
          ],
          "hdcp": ["2.2"],
          "encryptionSchemes": ["cbcs"]
        }
      }
    }
  }
]
~~~

Preflight response, sent by HTTP POST to the submitUri:

~~~json
{
  "nonce": "n-7Qe2xK",
  "results": {
    "org.ietf.moq.cmsf.decode": {
      "uhd": { "supported": true, "smooth": true, "powerEfficient": true }
    },
    "org.ietf.moq.cmsf.display": {
      "width": 3840, "height": 2160, "dynamicRange": "standard",
      "external": false
    },
    "org.ietf.moq.cmsf.clock": { "clientTime": 1746104607180 },
    "org.ietf.moq.cmsf.contentprotection": {
      "wv-canary": {
        "supported": true,
        "granted": ["HW_SECURE_ALL", "SW_SECURE_CRYPTO"],
        "challenges": {
          "HW_SECURE_ALL": "CAESvAsKiAsIARLtCQquAggCEhBhM...",
          "SW_SECURE_CRYPTO": "CAES9QoKwQoIARL0CAqwAggCEhCW..."
        },
        "encryptionSchemes": { "cbcs": true },
        "hdcp": { "2.2": "usable" }
      }
    }
  }
}
~~~

The publisher verifies the signature of the HW_SECURE_ALL challenge, confirms
that the content_id it carries contains the request nonce, and determines the
security level from the challenge itself. It then authorizes the 2160p tracks,
and denies the HDR rendition because the display probe reported a standard
dynamic range.

Preflight decision, returned in the HTTP response:

~~~json
{
  "nonce": "n-7Qe2xK",
  "expires": 1746104906044,
  "catalog": { "name": "catalog-premium" },
  "authorizedTracks": [
    { "name": "catalog-premium" },
    { "name": "video-2160" },
    { "name": "video-1080" },
    { "name": "audio" }
  ],
  "deniedTracks": [
    { "name": "video-2160-hdr", "reason": "hdr-not-supported" }
  ],
  "authorization": { "cat": "eyJhbGciOiJFZERTQSJ9..." }
}
~~~

# Catalog Examples

The following section provides non-normative JSON examples of various catalogs
compliant with this draft.


## Simulcast video tracks - 3 alternate video qualities along with audio

This example shows catalog for a media producer capable of sending 3
time-aligned video tracks for high definition, low definition and medium
definition video qualities, along with an audio track.

~~~json
{
  "version": "draft-01",
  "generatedAt": 1746104606044,
  "tracks":[
    {
      "name": "hd",
      "renderGroup": 1,
      "packaging": "cmaf",
      "isLive": true,
      "targetLatency": 2000,
      "initRef": "init-hd",
      "role": "video",
      "codec":"avc1.640028",
      "width":1920,
      "height":1080,
      "bitrate":5000000,
      "framerate":30,
      "altGroup":1
    },
    {
      "name": "md",
      "renderGroup": 1,
      "packaging": "cmaf",
      "isLive": true,
      "targetLatency": 2000,
      "initRef": "init-md",
      "role": "video",
      "codec":"avc1.64001e",
      "width":720,
      "height":640,
      "bitrate":3000000,
      "framerate":30,
      "altGroup":1
    },
    {
      "name": "sd",
      "renderGroup": 1,
      "packaging": "cmaf",
      "isLive": true,
      "targetLatency": 2000,
      "initRef": "init-sd",
      "role": "video",
      "codec":"avc1.64000d",
      "width":192,
      "height":144,
      "bitrate":500000,
      "framerate":30,
      "altGroup":1
    },
    {
      "name": "audio",
      "renderGroup": 1,
      "packaging": "cmaf",
      "isLive": true,
      "targetLatency": 2000,
      "initRef": "init-audio",
      "role": "audio",
      "codec":"mp4a.40.5",
      "samplerate":48000,
      "channelConfig":"2",
      "bitrate":67071
    }
   ],
  "initDataList": [
    {
      "id": "init-hd",
      "type": "inline",
      "data": "AAAAHGZ0eXBjbWYyAAAAAGNtZjJpc282bXA0MQ..."
    },
    {
      "id": "init-md",
      "type": "inline",
      "data": "AAAAHGZ0eXBjbWYyAAAAAGNtZjJpc282bXA0MQ..."
    },
    {
      "id": "init-sd",
      "type": "inline",
      "data": "AAAAHGZ0eXBjbWYyAAAAAGNtZjJpc282bXA0MQ..."
    },
    {
      "id": "init-audio",
      "type": "inline",
      "data": "AAAAHGZ0eXBjbWYyAAAAAGNtZjJpc282bXA0MQ..."
    }
  ]
}
~~~


## DRM-protected video with audio

This example shows a catalog for a single DRM-protected video track and an
unprotected audio track. The video track is encrypted using CBCS and
references three content protection entries: Widevine, PlayReady, and
FairPlay.

~~~json
{
  "version": "draft-01",
  "generatedAt": 1746104606044,
  "contentProtections": [
    {
      "refID": "1",
      "defaultKID": [
        "01234567-89ab-cdef-0123-456789abcdef"
      ],
      "scheme": "cbcs",
      "drmSystem": {
        "systemID": "edef8ba9-79d6-4ace-a3c8-27dcd51d21ed",
        "laURL": {
          "url": "https://widevine-license.example.com/proxy"
        },
        "pssh": "AAAAP3Bzc2gAAAAA7e+LqXnWSs6jy..."
      }
    },
    {
      "refID": "2",
      "defaultKID": [
        "01234567-89ab-cdef-0123-456789abcdef"
      ],
      "scheme": "cbcs",
      "drmSystem": {
        "systemID": "9a04f079-9840-4286-ab92-e65be0885f95",
        "laURL": {
          "url": "https://playready-license.example.com/auth"
        },
        "pssh": "AAACvnBzc2gAAAAAmgTweZhAQoar..."
      }
    },
    {
      "refID": "3",
      "defaultKID": [
        "01234567-89ab-cdef-0123-456789abcdef"
      ],
      "scheme": "cbcs",
      "drmSystem": {
        "systemID": "94ce86fb-07ff-4f43-adb8-93d2fa968ca2",
        "laURL": {
          "url": "https://fps-license.example.com/api/licenses"
        },
        "certURL": {
          "url": "https://fps-license.example.com/cert"
        }
      }
    }
  ],
  "tracks": [
    {
      "name": "video_protected",
      "packaging": "cmaf",
      "isLive": true,
      "buffers": {"target":1500},
      "role": "video",
      "renderGroup": 1,
      "altGroup": 1,
      "initRef": "1",
      "codec": "avc3.4D401F",
      "framerate": 25,
      "bitrate": 581905,
      "width": 1280,
      "height": 720,
      "contentProtectionRefIDs": ["1", "2", "3"]
    },
    {
      "name": "audio",
      "packaging": "cmaf",
      "isLive": true,
      "buffers": {"target":1500},
      "role": "audio",
      "renderGroup": 1,
      "initRef": "2",
      "codec": "mp4a.40.5",
      "samplerate": 48000,
      "channelConfig": "2",
      "bitrate": 67071
    }
  ],
  "initDataList": [
    {
      "id": "1",
      "type": "inline",
      "data": "AAAAHGZ0eXBjbWYyAAAAAGNtZjJpc282bXA0MQAA..."
    },
    {
      "id": "2",
      "type": "inline",
      "data": "AAAAHGZ0eXBjbWYyAAAAAGNtZjJpc282bXA0MQAA..."
    }
  ]
}
~~~

## ClearKey-protected video

This example shows a catalog using ClearKey content protection following
the ECCP model, suitable for testing and development.

~~~json
{
  "version": "draft-01",
  "generatedAt": 1746104606044,
  "contentProtections": [
    {
      "refID": "1",
      "defaultKID": [
        "01234567-89ab-cdef-0123-456789abcdef"
      ],
      "scheme": "cenc",
      "drmSystem": {
        "systemID": "1077efec-c0b2-4d02-ace3-3c1e52e2fb4b",
        "laURL": {
          "url": "https://clearkey-server.example.com/clearkey",
          "type": "EME-1.0"
        },
        "pssh": "AAAANHBzc2gBAAAAEHfv7MCyTQKs4..."
      }
    }
  ],
  "tracks": [
    {
      "name": "video",
      "packaging": "cmaf",
      "isLive": true,
      "buffers": {"target":1500},
      "role": "video",
      "renderGroup": 1,
      "initRef": "init-video",
      "codec": "avc1.640028",
      "framerate": 30,
      "bitrate": 5000000,
      "width": 1920,
      "height": 1080,
      "contentProtectionRefIDs": ["1"]
    }
  ],
  "initDataList": [
    {
      "id": "init-video",
      "type": "inline",
      "data": "AAAAHGZ0eXBjbWYyAAAAAGNtZjJpc282bXA0MQAA..."
    }
  ]
}
~~~


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security

## Capability preflight {#security-preflight}

Preflight results are generated by the subscriber and can be falsified. A
publisher MUST NOT base an authorization decision solely on results that it
cannot verify. Probe types that gate access to tracks SHOULD return evidence
the publisher can verify independently, such as a challenge signed by a
trusted component of the subscriber platform that embeds the request nonce.

A publisher MUST reject a preflight response whose nonce is unknown or whose
request has expired, and MUST NOT accept more than one response per nonce.

Preflight decisions can carry authorization tokens. A decision MUST NOT be
published on a track that is accessible to other subscribers. Decisions
returned over HTTP SHOULD be marked as not cacheable. Tokens issued through
preflight SHOULD be short-lived, scoped to the tracks listed in
authorizedTracks, and bound to the subscriber session where the authorization
scheme supports it.

Preflight does not prevent a subscriber that is authorized for a track from
redistributing its content.

## Content protection preflight {#security-cppreflight}

The robustness values granted to a subscriber are reported by the subscriber
and can be falsified. Publishers SHOULD determine the security level from the
license requests returned in the challenges field, and license servers SHOULD
independently enforce the same requirement when issuing content keys.

Content that is delivered to subscribers at different security levels SHOULD
be encrypted with different keys, so that a key obtained at a lower level does
not decrypt content intended for a higher one. Audio SHOULD be encrypted with a
key separate from video, since some DRM systems do not apply hardware
robustness to audio.

## Preflight privacy {#security-preflight-privacy}

Preflight results describe the subscriber device and so increase the
fingerprinting surface available to the publisher. A publisher SHOULD request
only the probe types it needs for an authorization decision, and SHOULD NOT
retain results longer than needed for that decision and any associated audit.
Probe type specifications SHOULD avoid results that carry stable device
identifiers. Where verifiable evidence necessarily identifies a device, as
some content protection challenges do, probe type specifications SHOULD
require any privacy protections that the evidence source offers.

For the content protection probe ({{cppreflight}}), a license request can
carry a device or client identifier. For example, a Widevine {{WIDEVINE}}
license request includes the client identification of the device unless it is
encrypted to the license service using a service certificate. Publishers
SHOULD use such privacy features when they are available, SHOULD limit access
to challenges to the service that verifies them, and SHOULD NOT use
identifiers obtained from challenges for purposes other than the
authorization decision.


# IANA Considerations

## Preflight event timeline types {#iana-preflight-event-types}

This document registers the following entries in the "MSF Event Timeline
Types" registry established by [MSF]:

| Event Type                           | Description        | Change Controller | Specification             |
|:=====================================|:===================|:==================|:==========================|
| org.ietf.moq.cmsf.preflight.request  | Preflight request  | IETF              | RFC XXXX                  |
| org.ietf.moq.cmsf.preflight.response | Preflight response | IETF              | RFC XXXX                  |

## CMSF Preflight Probe Types registry {#iana-preflight-probe-types}

This document establishes the "CMSF Preflight Probe Types" registry, in the
same registry group as the "MSF Event Timeline Types" registry established by
[MSF]. The registration policy is Specification Required {{!RFC8126}}. Each
entry contains an Identifier, a Description, and a Reference. The initial
contents are:

| Identifier                          | Description                    | Reference |
|:====================================|:===============================|:==========|
| org.ietf.moq.cmsf.contentprotection | Content protection capability  | RFC XXXX  |
| org.ietf.moq.cmsf.decode            | Codec decoding capability      | RFC XXXX  |
| org.ietf.moq.cmsf.display           | Display characteristics        | RFC XXXX  |
| org.ietf.moq.cmsf.clock             | Subscriber wallclock time      | RFC XXXX  |

### Guidance for Designated Experts {#iana-preflight-probe-experts}

Designated experts should evaluate registration requests against the
following criteria, per {{!RFC8126}}, Section 4.6:

* The specification defines the request value, the result value, and the
  form of the result when the probe is not executed, as required by
  {{preflightprobetypes}}.
* The specification states whether, and how, the publisher can verify the
  result independently of the subscriber. A probe type whose result cannot be
  verified should say so explicitly.
* The specification considers the fingerprinting impact of its result, as
  described in {{security-preflight-privacy}}, and avoids stable device
  identifiers where it can.
* The identifier does not duplicate the purpose of an existing entry.
* Identifiers registered by specifications outside the IETF stream should use
  Reverse Domain Name Notation under a domain controlled by the registrant.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
