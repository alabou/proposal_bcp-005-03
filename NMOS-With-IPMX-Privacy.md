# NMOS With IPMX Privacy Encryption
{:.no_toc}  
Copyright 2023, Matrox Graphics Inc.

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

---

{:toc}  

## Introduction

The Privacy Encryption Protocol (PEP) is defined by the [VSF][] technical recommendation [TR-10-13][]. It specifies a method to generate keying material for the encryption, decryption and authentication of media content over multicast and unicast networks. It is designed to support multiple types of transport protocol adaptations. The default adaptation defined in [TR-10-13][] technical recommendation specifies privacy encryption of media streams having an RTP payload format. The [VSF][] technical recommendation [TR-10-14][] provides the adaptation for the USB-IP protocol. This document defines additional adaptations for the SRT and UDP transport protocols.

This document briefly describes the PEP parameters. Detailed information is provided in the [TR-10-13][] technical recommendations.

Although the Privacy Encryption Protocol (PEP) is specified for an IPMX streaming environment, it may be used in non-IPMX streaming environments with devices supporting the transport protocol-specific PEP adaptation, along with the configuration of PEP Pre-Shared Keys.

## Use of Normative Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119][RFC-2119].

## Definitions
| Term | Definition |
|:--------------|:---------|
| PSK           | A Pre-Shared Key that serves as the root secret for the derivation of encryption and authentication keys in the privacy encryption process. |
| PEP           | Privacy Encryption Protocol, defined in the [TR-10-13][] technical recommendation for IPMX. |
| ECDH          | Elliptic Curve Diffie-Hellman |
| Administrator | An individual or entity with administrative authority for managing devices within a network, responsible for enforcing security policies, maintaining configuration compliance, and ensuring operational standards within a defined administrative domain. |

## Compliance

An implementation MUST be compliant with the strict requirements of [TR-10-13][] that are introduced by a `shall` clause. Some of these requirements are repeated in this specification to emphasize their importance, without impacting their original normative scope. This specification MAY document additional `protocol`, `mode` and `ecdh_curve` in addition to those specified in [TR-10-13][], [TR-10-14] or other VSF/IPMX technical recommendations. The inclusion of  additional values for these parameters MUST NOT be interpreted as a violation of any `shall` clause within the referenced technical recommendations.

An implementation MUST be compliant with the non-strict requirements of [TR-10-13][] that are introduced by `should` and `may` clauses when these requirements are explicily elevated to strict requirements in this specification through a `MUST` clause.

An implementation MUST be compliant with new requirements introduced by this specification that are not part of [TR-10-13][].

## Enabling/Disabling Privacy Encryption

As indicated in [TR-10-13][], the enabling and disabling of privacy encryption in devices supporting the PEP technology is under the control of the device manufacturer.

An NMOS API MUST NOT allow changing the enabling or disabling of privacy encryption. 

Refer to "SDP transport file parameters / NMOS transport parameters" of [TR-10-13][] for more details about enabling/disabling privacy encryption.

The enabling and disabling of privacy encryption is intentionally kept under the control of the device manufacturer, providing flexibility in implementation methods to accommodate diverse deployment environments and security requirements. 

> Note: The security postulate for a Sender is that privacy-encrypted content always remain protected and never transmitted in clear by the same device. For a Receiver, the postulate is that content can be trusted only if received with privacy encryption, requiring that privacy-encrypted content is never composited, mixed, or multiplexed with content received in clear. 

> Note: Privacy encryption is not a content protection mechanism and providing access to a low quality stream violates the privacy objective.

## PSK provisioning

As indicated in [TR-10-13][], the provisioning of PSK(s) in devices supporting the PEP technology is under the control of the device manufacturer. 

An NMOS API MUST NOT allow the provisioning of PSK in devices. 

Refer to the section "Key distribution" of [TR-10-13][] for more details about the keys distribution/provisioning process.

The PSK provisioning is intentionally kept under the control of the device manufacturer, providing flexibility in implementation methods to accommodate diverse deployment environments and security requirements. 

### Identification

A PSK has a value and a size (128, 256, or 512 bits). Each PSK is identified by a `key_id`, which MUST be unique among all devices within a network under a given administrative authority. This ensures that each PSK is uniquely identifiable across devices in the network and prevents ambiguity or potential security risks arising from duplicate key associations. Only one `key_id` SHOULD be associated with a given PSK value. For high-security deployments, only one `key_id` MUST be associated with a given PSK value. If a device spans multiple administrative authorities, the various authorities MUST use an identical definition (value, size, `key_id`) of the shared PSK(s). The key provisioning process MUST ensure that, for a given `key_id`, all devices within the network under a given administrative authority receive the same PSK value and size.

> Note: The [TR-10-13][] technical recommendation imposes precise requirements about the identification of the PSK size to the vendor-specific PSK provisioning API.

### Association

A device MAY be provisioned with multiple PSK. For a Sender device each Sender using privacy encryption MUST be associated with a provisioned PSK through its `key_id`. For a Receiver device each Receiver using privacy encryption MUST be associated with a set of allowed provisioned PSK through their `key_id`. A Receiver MUST populate the constraints associated with the IS-05 extended `ext_privacy_key_id` transport parameter with all the `key_id` values allowed by the Receiver. A Receiver using privacy encryption becomes associated with one such provisioned PSK through its `key_id` at activation time. A Receiver MUST fail the activation if the provided `key_id` is not provisioned in the Receiver device or is not included within the Receiver's `ext_privacy_key_id` transport parameter constraints.

## Parameters

The [TR-10-13][] technical recommendation defines the following parameters that are accessible as IS-05 extended transport parameters and constraints, as well as `privacy` attribute parameters in an SDP transport file.

Transport Parameter Name | Type | SDP Name | Sender | Receiver
 --- | --- | --- | --- | --- 
ext_privacy_protocol | string | protocol | r/w | r/w
ext_privacy_mode | string | mode | r/w | r/w
ext_privacy_iv | string | iv | read-only | r/w
ext_privacy_key_generator | string | key_generator | read-only | r/w
ext_privacy_key_version | string | key_version | read-only | r/w
ext_privacy_key_id | string | key_id | read-only | r/w
ext_privacy_ecdh_sender_public_key | string | - | read-only | r/w
ext_privacy_ecdh_receiver_public_key | string | - | r/w | read-only
ext_privacy_ecdh_curve | string | - | r/w | r/w

### IS-05 Transport Parameters

A Sender/Receiver implementing [TR-10-13][] MUST provide the following IS-05 extended transport parameters in the `active`, `staged` and `constraints` endpoints: `ext_privacy_protocol`, `ext_privacy_mode`, `ext_privacy_iv`, `ext_privacy_key_generator`, `ext_privacy_key_version`, and `ext_privacy_key_id`.

A Sender/Receiver implementing [TR-10-13][] and supporting the ECDH mode MUST also provide the following IS-05 extended transport parameters in the `active`, `staged`, and `constraints` endpoints: `ext_privacy_ecdh_sender_public_key`, `ext_privacy_ecdh_receiver_public_key` and `ext_privacy_ecdh_curve`.

The `ext_privacy` transport parameters MAY be used with any transport supporting privacy encryption and having a protocol adaptation specified in either one of [TR-10-13][], [TR-10-14][], other VSF/IPMX technical recommendations, or this specification.

### IS-05 Transport Parameters Constraints

Each `ext_privacy` transport parameter MUST have an associated constraint that indicates either that the parameter is unconstrained, allowing any valid value, or that it is constrained to a specific set of allowable values. A parameter identified as `read-only` in the parameter definitions table MUST always be constrained to a single value. A Sender/Receiver MUST fail an activation if any IS-05 `ext_privacy` transport parameter violates its defined constraints.

The `constraints` endpoint of the parameters `ext_privacy_protocol`, and `ext_privacy_mode` of Senders and Receivers MUST declare all the supported protocols and modes. These parameters MUST NOT be unconstrained. The constraints of these parameters MUST NOT change when `master_enable` attribute of a Sender/Receiver `active` endpoint is `true`.

The `constraints` endpoint of the parameter `ext_privacy_ecdh_curve` of Senders and Receivers MUST declare all the supported curves. This parameter MUST NOT be unconstrained. The constraints of this parameter MUST NOT change when `master_enable` attribute of a Sender/Receiver `active` endpoint is `true`.

> Note: The constraints on the parameters `ext_privacy_protocol`, `ext_privacy_mode`, and `ext_privacy_ecdh_curve` are expected to remain the same unless the device's Privacy Encryption Protocol support is reconfigured by a User through a vendor-specific mechanism.

### Protocol
The `protocol` parameter MUST be one of: "RTP", "RTP_KV", "USB", "USB_KV", or "NULL"

If privacy encryption is disabled or not supported by a Sender/Receiver and the `ext_privacy` transport parameters are present, the "NULL" protocol MUST be used for the active and staged `ext_privacy_protocol` extended transport parameters to indicate that privacy encryption is not available / disabled. The associated constraints MUST allow only the "NULL" protocol when an active or staged `ext_privacy_protocol` value is "NULL".

The `protocol` "RTP" MUST be supported by all devices implementing [TR-10-13][] for the `urn:x-nmos:transport:rtp`, `urn:x-nmos:transport:rtp.mcast`, and `urn:x-nmos:transport:rtp.ucast` transports.

The `protocol` "RTP_KV" MAY be supported by devices supporting the "RTP" `protocol`.

The `protocol` "USB_KV" MUST be supported by all devices implementing [TR-10-13][] and [TR-10-14][] for the `urn:x-matrox:transport:usb` transport.

The `protocol` "USB" MAY be supported by devices supporting the "USB_KV" `protocol`.

### Mode

If privacy encryption is disabled or not supported by a Sender/Receiver and the `ext_privacy` transport parameters are present, the "NULL" mode MUST be used for the active and staged `ext_privacy_mode` extended transport parameters to indicate that privacy encryption is not available or is disabled. The associated constraints MUST allow only the "NULL" mode when an active or staged `ext_privacy_mode` parameter is "NULL".

#### For protocols "RTP" and "RTP_KV"
The `mode` parameter MUST be one of: "AES-128-CTR", "AES-256-CTR", "AES-128-CTR_CMAC-64", "AES-256-CTR_CMAC-64", "AES-128-CTR_CMAC-64-AAD", "AES-256-CTR_CMAC-64-AAD", "ECDH_AES-128-CTR", "ECDH_AES-256-CTR", "ECDH_AES-128-CTR_CMAC-64", "ECDH_AES-256-CTR_CMAC-64", "ECDH_AES-128-CTR_CMAC-64-AAD", or "ECDH_AES-256-CTR_CMAC-64-AAD".

The `mode` "AES-128-CTR" MUST be supported by all devices implementing the "RTP" or "RTP_KV" protocols.

A Sender configured by an Administrator to use a 256 or 512 bit PSK MUST support only modes based on AES-256. A Sender configured to use a 128 bit PSK MAY support either or both AES-128 and AES-256 based modes.

> Note: if the IS-05 `ext_privacy_mode` transport parameter constraints of a Sender only allow modes based on AES-128, it indicates that only PSK of 128 bit are used.

A Receiver configured by an Administrator to allow the use of a 256 or 512 bit PSK MUST support modes based on AES-256. A Receiver configured to allow the  use of a 128 bit PSK MUST support modes based on AES-128. A Receiver MAY simultaneously support modes based on AES-128 and AES-256. A Receiver MUST fail activation if a `key_id` associated with a 256 or 512 bit PSK is used along with a `mode` that is not based on AES-256.

> Note: if the IS-05 `ext_privacy_mode` transport parameter constraints of a Receiver only allow modes based on AES-128, it indicates that only PSK of 128 bit are allowed.

The PEP "urn:ietf:params:rtp-hdrext:PEP-Full-IV-Counter" and "urn:ietf:params:rtp-hdrext:PEP-Short-IV-Counter" RTP Extension Headers MUST be declared in the SDP transport file. The declaration MUST be performed as per RFC 8285 using the "sendonly" direction.


#### For protocol "USB" and "USB_KV"
The `mode` parameter MUST be one of: "AES-128-CTR_CMAC-64-AAD", "AES-256-CTR_CMAC-64-AAD", "ECDH_AES-128-CTR_CMAC-64-AAD", or "ECDH_AES-256-CTR_CMAC-64-AAD".

The `mode` "AES-128-CTR_CMAC-64-AAD" MUST be supported by all devices implementing the "USB" or "USB_KV" protocols.

### Elliptic Curve Diffie-Hellman (ECDH)

The ECDH mode allows Perfect Forward Secrecy.

The `ecdh_curve` parameter MUST be one of: "secp256r1", "secp521r1", "25519", "448", or "NULL".

If the ECDH mode is not supported by a Sender/Receiver and the `ext_privacy_ecdh` transport parameters are present, the "NULL" curve MUST be used for the active and staged `ext_privacy_ecdh_curve` extended transport parameters to indicate that the ECDH mode is not available. The associated constraints MUST allow only the "NULL" curve when an active or staged `ext_privacy_ecdh_curve` parameter is "NULL".

The `ecdh_curve` “secp256r1" MUST be supported by all devices implementing the ECDH mode.

The ECDH functionality is available through the IS-05 extended transport parameters only. There are no ECDH parameters in the `privacy` attribute of an SDP transport file. The ECDH modes of operation are optional and none of these modes are required to be supported by an implementation conforming with [TR-10-13][], [TR-10-14][], or other VSF/IPMX technical recommendations.

> Note: A Sender/Receiver generates a new public key whenever it explicitly or implicitly becomes inactive.

## IS-04, IS-05, IS-11 Senders

A Sender compliant with this specification MUST provide a `privacy` Sender attribute to indicate that privacy encryption and the PEP protocol are used by the Sender. This attribute MUST be `true` if a `privacy` attribute is present in the Sender's SDP transport file and MUST be `false` if no `privacy` attributes are present. If an SDP transport file is not currently available, because the Sender is inactive, this attribute indicate wether or not such SDP transport file would contain a `privacy` attribute if the Sender was active at this time. If the Sender's transport protocol does not use an SDP transport file, this attribute indicate wether or not privacy encryption and the PEP protocol are used by the Sender.

> Note: A Sender not providing the `privacy` attribute is either not supporting privacy encryption and the PEP protocol or declare itself as not being compatible with the "Privacy" section of this document.

A Sender MAY provide a `urn:x-matrox:cap:transport:privacy` capability to indicate that privacy encryption and the PEP protocol are supported. A Sender MAY support either `true` or 
`false` values. 

> Note: A Sender is not allowed by [TR-10-13][] to support both values. The NMOS API is not allowed to change the enabling/disabling of privacy encryption.

A Sender implementing privacy encryption and the PEP protocol MUST provide IS-05 `ext_privacy` extended transport parameters and constraints that specify the extent of support for the features defined in [TR-10-13][]. A Controller MAY use a Sender's `urn:x-matrox:cap:transport:privacy` capability and the IS-05 `ext_privacy` transport parameters constraints to verify Receivers compliance with a Sender and if necessary constrain the Sender to make it compliant with the Receivers. It is not allowed to constrain a Sender for the `urn:x-matrox:cap:transport:privacy` capability as privacy encryption is a protection mechanism under the control of the Sender only. However, a Controller MAY select the value of IS-05 `ext_privacy` parameters within the limits of the associated constraints.

> Note: A Sender is configured by an administrator to produce either privacy encrypted streams or non-encrypted streams. The Sender  `urn:x-matrox:cap:transport:privacy` capability indicates the current configuration.



### SDP Transport File

The `privacy` attribute of [TR-10-13][] is not yet registered with IANA. If it would, the definition would indicate "Usage Level: session, media" indicating that a session-level `privacy` attribute represents the default value for a media-level `privacy` attribute that is not specified. The SDP transport file may provide the `privacy` information either at the session-level and media-level.

The PEP specification uses the expression "a privacy session attribute or a number of privacy media attributes" to clearly indicate the "Usage Level: session, media" usage.

### Consistency

If the `urn:x-matrox:cap:transport:privacy` capability only allows the value `true` then the Sender's associated SDP transport file, if any, MUST have a `privacy` attribute and the IS-05 `ext_privacy_protocol` and `ext_privacy_mode` transport parameters MUST have a value that is not `NULL`.

If the `urn:x-matrox:cap:transport:privacy` capability only allows the value `false` then the Sender's associated SDP transport file, if any, MUST NOT have a `privacy` attribute and the IS-05 `ext_privacy_protocol` and `ext_privacy_mode` transport parameters, if present, MUST have a `NULL` value.

The `urn:x-matrox:cap:transport:privacy` capability MUST NOT allow both `true` and `false` values.

## IS-04, IS-05, IS-11 Receivers

A Receiver supporting privacy encryption MUST follow the requirements defined in the "Privacy" section of the "NMOS With IPMX" specification regarding IS-04 Receiver Capabilities, and IS-05 transport parameter constraints.


A Receiver SHOULD provide a `urn:x-matrox:cap:transport:privacy` capability to indicate its support for Senders that use privacy encryption and the PEP protocol. A capability value of `true` indicates support for privacy encryption and the PEP protocol, while a value of `false` indicates that they are not supported. A Receiver implementing privacy encryption and the PEP protocol MUST provide IS-05 `ext_privacy` extended transport parameters and constraints that specify the extent of support for the features defined in [TR-10-13][].

A Receiver MAY support either `true` or `false` values.

> Note: A Receiver is not allowed by [TR-10-13][] to support both values. The NMOS API is not allowed to change the enabling/disabling of privacy encryption.

## Controller

A Receiver supporting privacy encryption MUST follow the requirements defined in the "Privacy" section of the "NMOS With IPMX" specification regarding IS-04 Sender/Receiver Capabilities, IS-05 transport parameter constraints, and IS-11 supported constraints.

A Controller MUST verify the compliance of Receivers with an active Sender using privacy encryption and the PEP protocol by referring to the Sender's SDP transport file `privacy` attribute or checking the Sender's associated `privacy` attribute, and verifying the IS-05 active `ext_privacy` extended transport parameters. The presence of a `privacy` attribute in an SDP transport file or the value `true` for the associated `privacy` Sender attribute indicates that the stream is privacy-protected. The presence of the IS-05 active `ext_privacy_protocol` and `ext_privacy_mode` transport parameters with a value that is not `NULL` indicates that the stream is privacy-protected. Only Receivers supporting privacy encryption and the PEP protocol MAY consume such streams.

A Controller has the responsibility of assessing the privacy encryption compatibility of Receivers with a Sender. This process is performed both at the IS-04 and IS-05 levels. If the Sender and the Receivers implement the `urn:x-matrox:cap:transport:privacy` capability, a Controller MAY perform an initial compatibility verification using this capability. Then, if the Sender and Receivers are compatible at the IS-04 level, or if the `urn:x-matrox:cap:transport:privacy` capability is not implemented by all the parties, a Controller MUST perform a final compatibility verification using the IS-05 `ext_privacy` transport parameters and associated constraints. A Controller MAY constrain the Sender with privacy encryption parameters compatible with the Receivers.

A Controller MUST ensure that the `protocol` and `mode` parameters are identical among the subscribing/connecting Receivers and the Sender. When an ECDH `mode` is used, the Controller MUST also ensure that the `ecdh_curve` parameter is identical between the subscribing/connecting Receiver and the peer Sender, and MUST exchange the ECDH `public_key` parameters between the peers.

A Controller MUST forward the `key_generator`, `key_version`, and `key_id` of a Sender to the subscribing/connecting Receivers.

If a mismatch is detected in `protocol`, `mode`, or `ecdh_curve` parameters, or if the ECDH `public_key` parameters cannot be exchanged, the Controller MUST prevent activation and SHOULD notify the User or an Administrator.

> Note: IS-11 operates at the IS-04 capabilities/constraints level and cannot be used to constrain privacy encryption, which must be managed using IS-05.

### IS-05 Sender Activation

The effective values of the read-only IS-05 `ext_privacy` transport parameters `iv`, `key_generator`, `key_version`, and `key_id` and the associated `privacy` attribute parameters of the SDP transport file of a Sender are not fixed until the activation of the Sender and `master_enable` becomes `true` at the `active` endpoint. A Controller MUST NOT assume final values for the IS-05 `ext_privacy` transport parameters of a Sender prior to activation. A Controller MUST NOT assume final values for the SDP transport file `privacy` attribute parameters of a Sender prior to activation.

The values of the parameters of the `privacy` attribute of the SDP transport file of an active Sender MUST match the values of the active `ext_privacy` transport parameters of that active Sender.

It is important to consider this requirement of [IS-05][] [Re-Activating Senders & Receivers](https://specs.amwa.tv/is-05/releases/v1.1.2/docs/Behaviour.html#re-activating-senders--receivers) that states:

> "If an explicit activation is performed against a Sender or Receiver, the API MUST request a re-application of settings to the underlying Sender or Receiver implementation whether the settings have changed or not. For example, in the case of multicast Receivers, it is suggested that this involves an explicit IGMP leave and join. For a Sender, this might involve stopping and re-starting the stream."

The [TR-10-13][] expression "becomes inactive" in the context of the ECDH private/public keys pair MUST be interpreted as an activation with `master_enable` set to `false` resulting in `master_enable` remaining or becoming `false` at the `active` endpoint of a Sender.

The [TR-10-13][] expression "becomes inactive" in other contexts MUST be interpreted as either internally becoming momentarily inactive during an activation with `master_enable` set to `true` resulting in `master_enable` remaining `true` at the `active` endpoint of a Sender (re-activation), or becoming inactive during an activation with `master_enable` set to `false` resulting in `master_enable` remaining or becoming `false` at the `active` endpoint of a Sender (de-activation).

In an activation (`master_enable` becomes true) or re-activation (`master_enable` remains true) a Sender MAY change all the privacy encryption parameters but the Sender's ECDH private/public keys pair MUST remain the same.

At activation (`master_enable` becomes true) and re-activation (`master_enable` remains true) a Sender MUST update the `ext_privacy` transport parameters at the `staged`, `active` and `constraints` endpoints and the `privacy` attribute parameters of the SDP transport file at the `transportfile` endpoint prior to completing the activation.

#### With ECDH

The ECDH mode is possible only in peer-to-peer mode where one Receiver connects or subscribes to one Sender.

The de-activation of a Sender with `master_enable` set to `false` MUST regenerate the value of the active and staged `ext_privacy_ecdh_sender_public_key` transport parameter if the ECDH mode is supported.

At de-activation (`master_enable` becomes or remains false) a Sender MUST update the `ext_privacy_ecdh_sender_public_key`, at the `staged`, `active` and `constraints` endpoints prior to completing the activation. To change the value of the `ext_privacy_ecdh_curve` transport parameter of a Sender, a Controller MUST set `master_enable` to `false` during an activation in order to regenerate a new value for the `ext_privacy_ecdh_sender_public_key` transport parameter.

A Controller MUST read the value of the `ext_privacy_ecdh_sender_public_key` transport parameter after the activation of the Sender with `master_enable` set to `true`.

A Controller MUST provide the value of the peer Receiver's `ext_privacy_ecdh_receiver_public_key` transport parameter to the Sender at activation with `master_enable` set to `true`.

With ECDH a Controller MUST exchange the Sender and Receiver public keys to activate an ECDH session. The ECDH functionality is available for peer-to-peer connections only. A Sender becomes associated with a peer Receiver at activation when `master_enable` becomes true.

### IS-05 Receiver activation

For transports supporting an SDP transport file, if the ECDH mode is not used, the process of activating a Receiver is the same with and without privacy encryption. A Controller SHOULD get the SDP transport file of a Sender and provide it to the Receivers at activation. The privacy encryption parameters of the Sender are automatically taken from the SDP transport file.

It is important to consider this requirement of [IS-05][] [Re-Activating Senders & Receivers](https://specs.amwa.tv/is-05/releases/v1.1.2/docs/Behaviour.html#re-activating-senders--receivers) that states:

> "If an explicit activation is performed against a Sender or Receiver, the API MUST request a re-application of settings to the underlying Sender or Receiver implementation whether the settings have changed or not. For example, in the case of multicast Receivers, it is suggested that this involves an explicit IGMP leave and join. For a Sender, this might involve stopping and re-starting the stream."

The [TR-10-13][] expression "becomes inactive" in the context of the ECDH private/public keys pair MUST be interpreted as an activation with `master_enable` set to `false` resulting in `master_enable` remaining or becoming `false` at the `active` endpoint of a Receiver.

The [TR-10-13][] expression "become inactive" in other contexts MUST be interpreted as either internally becoming momentarily inactive during an activation with `master_enable` set to `true` resulting in `master_enable` remaining `true` at the `active` endpoint of a Receiver (re-activation), or becoming inactive during activation with `master_enable` set to `false` resulting in `master_enable` remaining or becoming `false` at the `active` endpoint of a Receiver (de-activation).

In an activation (`master_enable` becomes true) or re-activation (`master_enable` remains true) a Receiver MAY change all the privacy encryption parameters but the Receiver's ECDH privat/public keys pair MUST remain the same.

At activation (`master_enable` becomes true) and re-activation (`master_enable` remains true) a Receiver MUST update the `ext_privacy` transport parameters, with the exception of `ext_privacy_ecdh_sender_public_key`, at the `staged`, `active`, and `constraints` endpoints prior to completing the activation.

#### With ECDH

The ECDH mode is possible only in peer-to-peer mode where one Receiver connect/subscribe to one Sender.

The de-activation of a Receiver with `master_enable` set to `false` MUST regenerate the value of the active and staged `ext_privacy_ecdh_receiver_public_key` transport parameter if the ECDH mode is supported. To change the value of the `ext_privacy_ecdh_curve` transport parameter of a Receiver, a Controller MUST set `master_enable` to `false` during an activation in order to regenerate a new value for the `ext_privacy_ecdh_receiver_public_key` transport parameter.

At de-activation (`master_enable` becomes or remains false) a Receiver MUST update the `ext_privacy_ecdh_receiver_public_key`, at the `staged`, `active`, and `constraints` endpoints prior to completing the activation.

Once a Controller reads the `ext_privacy_ecdh_receiver_public_key` transport parameter of a Receiver to provide its value to a Sender it MUST NOT perform any other activation of the Receiver with `master_enable` set to `false`, as otherwise the value of `ext_privacy_ecdh_receiver_public_key` would change.

A Controller MUST provide the value of the peer Sender's `ext_privacy_ecdh_sender_public_key` transport parameter to the Receiver at activation with `master_enable` set to `true`.

With ECDH a Controller MUST exchange the Sender and Receiver public keys to activate an ECDH session. The ECDH functionality is available for peer-to-peer connections only. A Sender becomes associated with a peer Receiver at activation when `master_enable` becomes`true`.

## RTP Transport Adaptation

This `protocol` is used for `urn:x-nmos:transport:rtp`, `urn:x-nmos:transport:rtp.mcast`, and `urn:x-nmos:transport:rtp.ucast`.

See the [TR-10-13][] technical recommendation for further details.

## USB-IP Transport Adaptation

This `protocol` is used for `urn:x-matrox:transport:usb`.

See the [TR-10-14][] technical recommendation for the details.

[RFC-2119]: https://tools.ietf.org/html/rfc2119 "Key words for use in RFCs"
[RFC-2250]: https://tools.ietf.org/html/rfc2250 "RTP Payload Format for MPEG1/MPEG2 Video"
[RFC-3551]: https://tools.ietf.org/html/rfc3551 "RTP Profile for Audio and Video Conferences with Minimal Control"
[IS-04]: https://specs.amwa.tv/is-04/ "AMWA IS-04 NMOS Discovery and Registration Specification"
[IS-05]: https://specs.amwa.tv/is-05/ "AMWA IS-05 NMOS Device Connection Management Specification"
[NMOS Parameter Registers]: https://specs.amwa.tv/nmos-parameter-registers/ "Common parameter values for AMWA NMOS Specifications"
[VSF]: https://vsf.tv/ "Video Services Forum"
[SMPTE]: https://www.smpte.org/ "Society of Media Professionals, Technologists and Engineers"
[BCP-004-01]: https://specs.amwa.tv/bcp-004-01/ "AMWA BCP-004-01 NMOS Receiver Capabilities"
[BCP-004-02]: https://specs.amwa.tv/bcp-004-02/ "AMWA BCP-004-02 NMOS Sender Capabilities"
[TR-10-13]: https://vsf.tv/download/technical_recommendations/VSF_TR-10-13_2024-01-19.pdf "Internet Protocol Media Experience (IPMX): Privacy Encryption Protocol (PEP)"
[TR-10-14]: https://vsf.tv/download/technical_recommendations/VSF_TR-10-14_2024-09-24.pdf "Internet	Protocol Media Experience (IPMX): IPMX USB"
