# NMOS With IPMX Privacy Encryption
{:.no_toc}  
Copyright 2023, Matrox Graphics Inc.

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

---

{:toc}  

## Introduction

The Privacy Encryption Protocol (PEP) is defined by the [VSF][] technical recommendation [TR-10-13][]. It specifies a method for generating cryptographic keys used in the encryption, decryption, and authentication of media content transmitted over multicast and unicast networks. PEP is designed to support multiple transport protocol adaptations. The default adaptation, defined in the [TR-10-13][] technical recommendation, addresses privacy encryption of media streams using the RTP streaming protocol. Additionally, the [VSF][] technical recommendation [TR-10-14][] defines an adaptation for the USB-IP streaming protocol.

This document focuses on the application of PEP within an NMOS environment. Detailed information about PEP itself is provided in the [TR-10-13][] technical recommendation.

While the Privacy Encryption Protocol (PEP) is primarily specified for IPMX streaming environments, it may also be utilized in non-IPMX contexts by devices implementing the [TR-10-14][] technical recommendation alongside compatible transport protocol adaptations.

## Use of Normative Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119][RFC-2119].

## Definitions
| Term | Definition |
|:--------------|:---------|
| PSK           | A Pre-Shared Key that serves as the root secret for the derivation of encryption and authentication keys in the privacy encryption process. |
| PEP           | Privacy Encryption Protocol, defined in the [TR-10-13][] technical recommendation for IPMX. |
| ECDH          | Elliptic Curve Diffie-Hellman |

## Compliance

An implementation MUST comply with the strict requirements of [TR-10-13][] introduced by a `shall` clause. Some of these requirements are reiterated in this specification to emphasize their importance, without altering their original normative scope. This specification MAY document additional values for `protocol`, `mode`, and `ecdh_curve` beyond those specified in [TR-10-13][], [TR-10-14], or other VSF/IPMX technical recommendations. The inclusion of additional values for these parameters MUST NOT be interpreted as violating any `shall` clause within the referenced technical recommendations.

An implementation MUST comply with the non-strict requirements of [TR-10-13][] introduced by `should` and `may` clauses if these requirements are explicitly elevated to strict requirements in this specification through a `MUST` clause.

An implementation MUST comply with new requirements introduced by this specification that are not part of [TR-10-13][].

## General Provisions

Nodes capable of transmitting privacy encrypted streams using the Privacy Encryption Protocol MUST have Source, Flow and Sender resources in the IS-04 Node API.

Nodes capable of receiving privacy encrypted streams using the Privacy Encryption Protocol MUST have Receiver resources in the IS-04 Node API.

Nodes compliant with this specification MUST implement [IS-04][] v1.3 or higher and [IS-05][] v1.1 or higher.

## PSK Provisioning

As indicated in [TR-10-13][], the provisioning of PSK(s) in devices supporting the PEP technology remains under the control of the device manufacturer.

An NMOS API MUST NOT allow the provisioning of PSKs in devices.

Refer to the section "Key Distribution" of [TR-10-13][] for more details about the key distribution and provisioning processes.

Provisioning of PSKs is intentionally under the control of the device manufacturer, providing flexibility in implementation methods to accommodate diverse deployment environments and security requirements.

### Identification

A PSK has a value and a size (128, 256, or 512 bits). Each PSK is identified by a `key_id`, which MUST be unique among all devices within a network under a given administrative authority. This uniqueness ensures clear identification of each PSK across networked devices, preventing ambiguity and reducing security risks associated with duplicate key associations. Only one `key_id` SHOULD be associated with a given PSK value. For high-security deployments, exactly one `key_id` MUST be associated with a given PSK value. If a device spans multiple administrative authorities, all involved authorities MUST use an identical definition (including value, size, and `key_id`) for the shared PSK(s). The key provisioning process MUST ensure that, for a given `key_id`, all devices within the network under a given administrative authority receive the same PSK value and size.

> Note: The [TR-10-13][] technical recommendation includes explicit requirements regarding the identification of the PSK size in the vendor-specific PSK provisioning API.

### Association

A device MAY be provisioned with multiple PSKs. For a Sender device, each Sender using privacy encryption MUST be associated with a provisioned PSK via its `key_id`. For a Receiver device, each Receiver using privacy encryption MUST be associated with a set of allowed provisioned PSKs through their `key_id` values. A Receiver MUST populate the constraints associated with the IS-05 extended `ext_privacy_key_id` transport parameter with all `key_id` values allowed by the Receiver. A Receiver using privacy encryption becomes associated with one of the provisioned PSKs through its `key_id` at activation time. A Receiver MUST fail activation if the provided `key_id` is not provisioned in the device or is not included in the Receiver's `ext_privacy_key_id` transport parameter constraints.

## Enabling/Disabling Privacy Encryption

As indicated in [TR-10-13][], the enabling and disabling of privacy encryption in devices supporting PEP technology remains under the control of the device manufacturer.

An NMOS API MUST NOT allow changes to the enabling or disabling of privacy encryption.

Refer to the section "SDP Transport File Parameters / NMOS Transport Parameters" of [TR-10-13][] for more details about enabling/disabling privacy encryption.

The enabling and disabling of privacy encryption is intentionally under the control of the device manufacturer, providing flexibility in implementation methods to accommodate diverse deployment environments and security requirements.

> Note: The security postulate for a Sender is that privacy-encrypted content remains protected and is never transmitted in clear by any Sender within a device. For a Receiver, the postulate is that content can be trustworthy only when received with privacy encryption, requiring that privacy-encrypted content is never composited, mixed, or multiplexed with content received in clear by other Receivers.

> Note: Privacy encryption is not a content protection mechanism, and providing access to a low-quality stream violates the privacy objective.

## Parameters

The [TR-10-13][] technical recommendation defines the following parameters, which are accessible both as IS-05 extended transport parameters and constraints, and as `privacy` attribute parameters in an SDP transport file.

For transport protocols using an SDP transport file: a Sender MUST communicate privacy encryption parameters in the SDP transport file associated with the privacy-encrypted stream, and MUST also communicate these parameters using the extended NMOS transport parameters.

For transport protocols that do not use an SDP transport file: a Sender or Receiver MUST communicate the privacy encryption parameters using the extended NMOS transport parameters.

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

A Sender/Receiver implementing [TR-10-13][] and supporting ECDH modes MUST also provide the following IS-05 extended transport parameters in the `active`, `staged`, and `constraints` endpoints: `ext_privacy_ecdh_sender_public_key`, `ext_privacy_ecdh_receiver_public_key`, and `ext_privacy_ecdh_curve`.

The `ext_privacy` transport parameters MAY be used with any transport supporting privacy encryption and having a protocol adaptation specified in [TR-10-13][], [TR-10-14][], other VSF/IPMX technical recommendations, or this specification.

### IS-05 Transport Parameters Constraints

Each `ext_privacy` transport parameter MUST have an associated constraint that indicates either that the parameter is unconstrained, allowing any valid value, or that it is constrained to a specific set of allowable values. A parameter identified as `read-only` in the parameter definitions table MUST always be constrained to a single value. A Sender/Receiver MUST fail an activation if any IS-05 `ext_privacy` transport parameter violates its defined constraints.

The `constraints` endpoint of the parameters `ext_privacy_protocol` and `ext_privacy_mode` on Senders and Receivers MUST declare all the supported protocols and modes. These parameters MUST NOT be unconstrained. The constraints of these parameters MUST NOT change when the `master_enable` attribute of a Sender/Receiver `active` endpoint is `true`.

The `constraints` endpoint of the parameter `ext_privacy_ecdh_curve` on Senders and Receivers MUST declare all the supported curves. This parameter MUST NOT be unconstrained. The constraints of this parameter MUST NOT change when the `master_enable` attribute of a Sender/Receiver `active` endpoint is `true`.

> Note: The constraints on the parameters `ext_privacy_protocol`, `ext_privacy_mode`, and `ext_privacy_ecdh_curve` are expected to remain the same unless the device's Privacy Encryption Protocol support is reconfigured by a User through a vendor-specific mechanism.

### Protocol

The `protocol` parameter MUST be one of: "RTP", "RTP_KV", "USB", "USB_KV", or "NULL".

If privacy encryption is disabled or not supported by a Sender/Receiver and the `ext_privacy` transport parameters are present, the "NULL" protocol MUST be used for the active and staged `ext_privacy_protocol` extended transport parameters to indicate that privacy encryption is not available or disabled. The associated constraints MUST allow only the "NULL" protocol when an active or staged `ext_privacy_protocol` value is "NULL".

The `protocol` "RTP" MUST be supported by all devices implementing [TR-10-13][] for the `urn:x-nmos:transport:rtp`, `urn:x-nmos:transport:rtp.mcast`, and `urn:x-nmos:transport:rtp.ucast` transports.

The `protocol` "RTP_KV" MAY be supported by devices supporting the "RTP" `protocol`.

The `protocol` "USB_KV" MUST be supported by all devices implementing [TR-10-13][] and [TR-10-14][] for the `urn:x-nmos:transport:usb` transport.

The `protocol` "USB" MAY be supported by devices supporting the "USB_KV" `protocol`.

### Mode

If privacy encryption is disabled or not supported by a Sender/Receiver and the `ext_privacy` transport parameters are present, the "NULL" mode MUST be used for the active and staged `ext_privacy_mode` extended transport parameters to indicate that privacy encryption is not available or is disabled. The associated constraints MUST allow only the "NULL" mode when an active or staged `ext_privacy_mode` parameter is "NULL".

#### For protocols "RTP" and "RTP_KV"
The `mode` parameter MUST be one of: "AES-128-CTR", "AES-256-CTR", "AES-128-CTR_CMAC-64", "AES-256-CTR_CMAC-64", "AES-128-CTR_CMAC-64-AAD", "AES-256-CTR_CMAC-64-AAD", "ECDH_AES-128-CTR", "ECDH_AES-256-CTR", "ECDH_AES-128-CTR_CMAC-64", "ECDH_AES-256-CTR_CMAC-64", "ECDH_AES-128-CTR_CMAC-64-AAD", or "ECDH_AES-256-CTR_CMAC-64-AAD".

The `mode` "AES-128-CTR" MUST be supported by all devices implementing the "RTP" or "RTP_KV" protocols.

A Sender configured with a 256-bit or 512-bit PSK MUST support only modes based on AES-256. A Sender configured with a 128-bit PSK MAY support modes based on AES-128, AES-256, or both.

> Note: If the constraints on the IS-05 `ext_privacy_mode` transport parameter of a Sender only allow modes based on AES-128, it indicates that only 128-bit PSKs are used.

A Receiver configured with a 256-bit or 512-bit PSK MUST support modes based on AES-256. A Receiver configured with a 128-bit PSK MUST support modes based on AES-128. A Receiver MAY support both AES-128 and AES-256-based modes simultaneously. A Receiver MUST fail activation if a `key_id` associated with a 256-bit or 512-bit PSK is used with a `mode` that is not based on AES-256.

> Note: If the constraints on the IS-05 `ext_privacy_mode` transport parameter of a Receiver only allow modes based on AES-128, it indicates that only 128-bit PSKs are allowed.

The PEP "urn:ietf:params:rtp-hdrext:PEP-Full-IV-Counter" and "urn:ietf:params:rtp-hdrext:PEP-Short-IV-Counter" RTP Extension Headers MUST be declared in the SDP transport file. The declaration MUST be performed as per [RFC-8285][] using the "sendonly" direction.

#### For protocol "USB" and "USB_KV"
The `mode` parameter MUST be one of: "AES-128-CTR_CMAC-64-AAD", "AES-256-CTR_CMAC-64-AAD", "ECDH_AES-128-CTR_CMAC-64-AAD", or "ECDH_AES-256-CTR_CMAC-64-AAD".

The `mode` "AES-128-CTR_CMAC-64-AAD" MUST be supported by all devices implementing the "USB" or "USB_KV" protocols.

### Elliptic Curve Diffie-Hellman (ECDH)

The ECDH mode allows **Perfect Forward Secrecy**.

The `ecdh_curve` parameter MUST be one of: "secp256r1", "secp521r1", "25519", "448", or "NULL".

If the ECDH mode is not supported by a Sender/Receiver and the `ext_privacy_ecdh` transport parameters are present, the "NULL" curve MUST be used for the active and staged `ext_privacy_ecdh_curve` extended transport parameters to indicate that the ECDH mode is not available. The associated constraints MUST allow only the "NULL" curve when an active or staged `ext_privacy_ecdh_curve` parameter is "NULL".

The `ecdh_curve` "secp256r1" MUST be supported by all devices implementing the ECDH mode.

The ECDH functionality is available exclusively through the IS-05 extended transport parameters only. There are no ECDH parameters in the `privacy` attribute of an SDP transport file. The ECDH modes of operation are optional and none of these modes are required to be supported by an implementation conforming with [TR-10-13][], [TR-10-14][], or other VSF/IPMX technical recommendations.

> Note: A Sender/Receiver generates a new public key whenever it explicitly or implicitly becomes inactive.

## IS-04, IS-05, IS-11 Senders

A Sender compliant with this specification MUST provide a `privacy` Sender attribute to indicate that privacy encryption and the PEP protocol are used by the Sender. This attribute MUST be `true` if a `privacy` attribute is present in the Sender's SDP transport file, and MUST be `false` if no `privacy` attributes are present. If an SDP transport file is not currently available because the Sender is inactive, this attribute indicates whether such an SDP transport file would contain a `privacy` attribute if the Sender were active at that time. If the Sender's transport protocol does not use an SDP transport file, this attribute indicates whether privacy encryption and the PEP protocol are used by the Sender. 

A Sender implementing privacy encryption and the PEP protocol MUST provide IS-05 `ext_privacy` extended transport parameters and constraints that specify the extent of support for the features defined in [TR-10-13][].

If the Sender's `privacy` attribute is `false`, the `ext_privacy_protocol` and `ext_privacy_mode` extended transport parameters MUST be "NULL".
If the Sender's `privacy` attribute is `true`, the `ext_privacy_protocol` and `ext_privacy_mode` extended transport parameters MUST NOT be "NULL".

> Note: A Sender not providing the `privacy` attribute is either not supporting privacy encryption and the PEP protocol, or declaring itself as not implementing this specification.

A Sender MAY provide a `urn:x-nmos:cap:transport:privacy` capability to indicate that privacy encryption and the PEP protocol are supported. A Sender MAY support either the `true` or `false` value. 

> Note: A Sender is not allowed by [TR-10-13][] to support both values. An NMOS API is not allowed to change the enabling or disabling of privacy encryption.

A Controller MAY use a Sender's `urn:x-nmos:cap:transport:privacy` capability and the IS-05 `ext_privacy` transport parameters constraints to verify Receivers compliance with a Sender, and if necessary, constrain the Sender to make it compliant with the Receivers. It is not allowed to constrain a Sender's `urn:x-nmos:cap:transport:privacy` capability, as privacy encryption is a protection mechanism under the control of the Sender only. However, a Controller MAY select the values of IS-05 `ext_privacy` parameters within the limits of the associated constraints.

> Note: A Sender is configured to produce either privacy encrypted streams or non-encrypted streams. The Sender's `urn:x-nmos:cap:transport:privacy` capability indicates the current configuration.

### SDP Transport File

The `privacy` attribute of [TR-10-13][] is not yet registered with IANA. If it were, the definition would indicate "Usage Level: session, media", meaning that a session-level `privacy` attribute represents the default value for any media-level `privacy` attribute that is not specified. The SDP transport file may provide the `privacy` information at either the session-level or media-level.

The PEP specification uses the expression "a privacy session attribute or a number of privacy media attributes" to clearly indicate this "Usage Level: session, media" usage.

### Consistency

If the `urn:x-nmos:cap:transport:privacy` capability only allows the value `true`, then the Sender's associated SDP transport file, if any, MUST have a `privacy` attribute, and the IS-05 `ext_privacy_protocol` and `ext_privacy_mode` transport parameters MUST have a value that is not "NULL".

If the `urn:x-nmos:cap:transport:privacy` capability only allows the value `false`, then the Sender's associated SDP transport file, if any, MUST NOT have a `privacy` attribute, and the IS-05 `ext_privacy_protocol` and `ext_privacy_mode` transport parameters, if present, MUST have a "NULL" value.

The `urn:x-nmos:cap:transport:privacy` capability MUST NOT allow both `true` and `false` values.

## IS-04, IS-05, IS-11 Receivers

A Receiver implementing privacy encryption and the PEP protocol MUST provide IS-05 `ext_privacy` extended transport parameters and constraints that specify the extent of support for the features defined in [TR-10-13][].

A Receiver SHOULD provide a `urn:x-nmos:cap:transport:privacy` capability to indicate its support for Senders that use privacy encryption and the PEP protocol. A capability value of `true` indicates support for privacy encryption and the PEP protocol, while a value of `false` indicates that they are not supported. A Receiver MAY support either the `true` or `false` value.

> Note: A Receiver is not allowed by [TR-10-13][] to support both values. The NMOS API is not allowed to change the enabling/disabling of privacy encryption.

## Controller

A Controller MUST verify the compliance of Receivers with an active Sender using privacy encryption and the PEP protocol by referring to the Sender's SDP transport file `privacy` attribute or by checking the Sender's associated `privacy` attribute, and by verifying the IS-05 active `ext_privacy` extended transport parameters. The presence of a `privacy` attribute in the SDP transport file, or the value `true` for the associated `privacy` Sender attribute, indicates that the stream is privacy-protected. The presence of the Sender's IS-05 active `ext_privacy_protocol` and `ext_privacy_mode` transport parameters with a value other than "NULL" also indicates that the stream is privacy-protected. Only Receivers that support privacy encryption and the PEP protocol MAY consume such streams.

A Controller is responsible for assessing the compatibility of Receivers with a Sender in regard to privacy encryption. This process is performed at both the IS-04 and IS-05 levels. If the Sender and the Receivers implement the `urn:x-nmos:cap:transport:privacy` capability, a Controller MAY perform an initial compatibility check using this capability. If the Sender and Receivers are compatible at the IS-04 level, or if the `urn:x-nmos:cap:transport:privacy` capability is not implemented by all the parties, a Controller MUST perform a final compatibility verification using the IS-05 `ext_privacy` transport parameters and associated constraints. A Controller MAY constrain the Sender to use privacy encryption parameters compatible with the Receivers.

A Controller MUST ensure that the `protocol` and `mode` parameters are identical between the subscribing or connecting Receivers and the Sender. When an ECDH `mode` is used, the Controller MUST also ensure that the `ecdh_curve` parameter is identical between the subscribing or connecting Receiver and the peer Sender, and MUST exchange the ECDH `public_key` parameters between the peers.

A Controller MUST forward the `key_generator`, `key_version`, and `key_id` of a Sender to the subscribing or connecting Receivers.

If a mismatch is detected in the `protocol`, `mode`, or `ecdh_curve` parameters, or if the ECDH `public_key` parameters cannot be exchanged, the Controller MUST prevent activation and SHOULD notify the User.

> Note: IS-11 operates at the IS-04 capabilities/constraints level and cannot be used to constrain privacy encryption, which must be managed using IS-05.

### IS-05 Sender Activation

The effective values of the read-only IS-05 `ext_privacy` transport parameters `iv`, `key_generator`, `key_version`, and `key_id` and the associated `privacy` attribute parameters of the SDP transport file of a Sender are not fixed until activation, when `master_enable` becomes `true` at the `active` endpoint. A Controller MUST NOT assume final values for the IS-05 `ext_privacy` transport parameters of a Sender prior to activation. A Controller MUST NOT assume final values for the SDP transport file `privacy` attribute parameters of a Sender prior to activation.

The values of the parameters of the `privacy` attribute of the SDP transport file of an active Sender MUST match the values of the active `ext_privacy` transport parameters of that active Sender.

The [TR-10-13][] expression "becomes inactive", in the context of the ECDH private/public keys pair, MUST be interpreted as an activation with `master_enable` set to `false`, resulting in `master_enable` remaining or becoming `false` at the `active` endpoint of a Sender.

The [TR-10-13][] expression "becomes inactive", in other contexts, MUST be interpreted as either (a) internally becoming momentarily inactive during an activation with `master_enable` set to `true`, resulting in `master_enable` remaining `true` at the `active` endpoint of a Sender (re-activation), or (b) becoming inactive during an activation with `master_enable` set to `false`, resulting in `master_enable` remaining or becoming `false` at the `active` endpoint of a Sender (de-activation).

During an activation (`master_enable` becomes true) or re-activation (`master_enable` remains true), a Sender MAY change all the privacy encryption parameters, but the Sender's ECDH private/public keys pair MUST remain the same.

At activation (`master_enable` becomes true) and re-activation (`master_enable` remains true), a Sender MUST update the `ext_privacy` transport parameters at the `staged`, `active` and `constraints` endpoints, and the `privacy` attribute parameters of the SDP transport file at the `transportfile` endpoint, prior to completing the activation.

#### With ECDH

The ECDH mode is possible only in peer-to-peer mode, where one Receiver connects or subscribes to one Sender.

The de-activation of a Sender (with `master_enable` set to `false`), MUST regenerate the value of the `ext_privacy_ecdh_sender_public_key` transport parameter, provided that the ECDH mode is supported.

At de-activation (`master_enable` becomes or remains false), a Sender MUST update the `ext_privacy_ecdh_sender_public_key` parameter at the `staged`, `active` and `constraints` endpoints prior to completing the activation. To change the value of the `ext_privacy_ecdh_curve` transport parameter of a Sender, a Controller MUST set `master_enable` to `false` during an activation in order to trigger regeneration of a new value for the `ext_privacy_ecdh_sender_public_key` transport parameter.

A Controller MUST read the value of the `ext_privacy_ecdh_sender_public_key` transport parameter after the activation of the Sender with `master_enable` set to `true`. 

A Controller MUST provide the value of the peer Receiver's `ext_privacy_ecdh_receiver_public_key` transport parameter to the Sender during activation, with `master_enable` set to `true`.

With ECDH, a Controller MUST exchange the Sender and Receiver public keys to activate an ECDH session. The ECDH functionality is available for peer-to-peer connections only. A Sender becomes associated with a peer Receiver at activation, when `master_enable` becomes true.

### IS-05 Receiver activation

For transports supporting an SDP transport file, if the ECDH mode is not used, the process of activating a Receiver is the same with and without privacy encryption. A Controller SHOULD retrieve the SDP transport file of a Sender and provide it to the Receivers at activation. The privacy encryption parameters of the Sender are automatically taken from the SDP transport file.

The [TR-10-13][] expression "becomes inactive", in the context of the ECDH private/public keys pair, MUST be interpreted as an activation with `master_enable` set to `false`, resulting in `master_enable` remaining or becoming `false` at the `active` endpoint of a Receiver.

The [TR-10-13][] expression "become inactive", in other contexts, MUST be interpreted as either (1) internally becoming momentarily inactive during an activation with `master_enable` set to `true`, resulting in `master_enable` remaining `true` at the `active` endpoint of a Receiver (re-activation), or (2) becoming inactive during an activation with `master_enable` set to `false`, resulting in `master_enable` remaining or becoming `false` at the `active` endpoint of a Receiver (de-activation).

During an activation (`master_enable` becomes true) or re-activation (`master_enable` remains true), a Receiver MAY change all privacy encryption parameters, but the Receiver's ECDH private/public keys pair MUST remain the same.

At activation (`master_enable` becomes true) and re-activation (`master_enable` remains true), a Receiver MUST update the `ext_privacy` transport parameters, with the exception of `ext_privacy_ecdh_sender_public_key`, at the `staged`, `active`, and `constraints` endpoints prior to completing the activation.

#### With ECDH

The ECDH mode is possible only in peer-to-peer mode, where one Receiver connects or subscribes to one Sender.

The de-activation of a Receiver (with `master_enable` set to `false`) MUST regenerate the value of the `ext_privacy_ecdh_receiver_public_key` transport parameter, provided that the ECDH mode is supported. 

At de-activation (`master_enable` becomes or remains `false`), a Receiver MUST update the `ext_privacy_ecdh_receiver_public_key` parameter at the `staged`, `active`, and `constraints` endpoints prior to completing the activation. To change the value of the `ext_privacy_ecdh_curve` transport parameter of a Receiver, a Controller MUST set `master_enable` to `false` during an activation in order to trigger regeneration of a new value for the `ext_privacy_ecdh_receiver_public_key` transport parameter.

Once a Controller reads the `ext_privacy_ecdh_receiver_public_key` transport parameter of a Receiver and provides its value to a Sender, it MUST NOT perform any subsequent activation of the Receiver with `master_enable` set to `false`, as this would change the value of `ext_privacy_ecdh_receiver_public_key`.

A Controller MUST provide the value of the peer Sender's `ext_privacy_ecdh_sender_public_key` transport parameter to the Receiver during activation, when `master_enable` is set to `true`.

With ECDH, a Controller MUST exchange the Sender and Receiver public keys to activate an ECDH session. The ECDH functionality is available for peer-to-peer connections only. A Sender becomes associated with a peer Receiver at activation, when `master_enable` becomes`true`.

## RTP Transport Adaptation

This `protocol` is used for `urn:x-nmos:transport:rtp`, `urn:x-nmos:transport:rtp.mcast`, and `urn:x-nmos:transport:rtp.ucast`.

See the [TR-10-13][] technical recommendation for further details.

## USB-IP Transport Adaptation

This `protocol` is used for `urn:x-nmos:transport:usb`.

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
