# Authentication Factors in IPEX

v1.1 2026/08/15

As of version 1.1, this discussion supersedes the “Multiply Endorsed” discussion (see https://github.com/WebOfTrust/keripy/discussions/1555).    

## Introduction

An IPEX grant has two distinct authentication-factor purposes associated with the disclosure presented in the `grant` routed exchange message. One factor authenticates the Issuer of any ACDC disclosed (granted) by the IPEX `grant` message. The other factor authenticates the Grantor (Discloser) of the ACDC so disclosed by the IPEX `grant` message. This second purpose may be extended to include additional endorsers of the `grant` message. Each of these additional endorsers is thereby also a Grantor. Each must be individually authenticated and hence provide an authentication factor for the Grantor authentication purpose. When there are multiple endorsers (Grantors) of the `grant` message, then we call this a multiply endorsed presentation or equivalently a multiply granted presentation.   To elaborate, the second authentication factor provides proof of the freshness of control over the keystate of each Grantor at the time of the grant (presentation) via the  `grant` exchange message itself (`xip` or `exn`) of the ACDC(s) so disclosed (granted). For a given ACDC, a Grantor may be the Issuee, Issuer, both, or neither.  In common parlance, a Grantor is a Presentor, and the grant is a presentation. While there is some apparent ambiguity regarding the terms presentation and presenter in an IPEX because it is a peer-to-peer interaction or negotiation where both parties "present" information, we use the term Presentor and Grantor to mean the Discloser in the IPEX. Typically one AID is used to represent the Discloser (Grantor/Presenter). When an IPEX is multiply granted or endorsed, however, it means that a set of AIDS are needed to represent the Discloser (Grantor/Presenter).

The salient motivation for a multiply granted/endorsed IPEX disclosure is to protect a given AID from disclosure until after contractual protection is in place. This means that the AID used to negotiate the disclosure may not be the principal AID provided in the disclosure. In other words, a given controller may use one AID it controls as a proxy to negotiate on behalf of another AID it controls so that this other AID is not disclosed until after contractual protection is in place. This protects the other AID from correlation if a failed negotiation does not result in full contractual protection. Only the first AID needs to participate in the failed negotiation and is the only one exposed by it.

The purpose of an IPEX is to eventually disclose one or more ACDCs via its `grant` message. When multiple ACDCs are disclosed by an IPEX `grant`, then the disclosure is a DAG of ACDCs with a single source node called the origin.
Given a DAG disclosure, each authentication factor purpose may be separately satisfied for each ACDC in the DAG by a different authentication factor. The `grant` message attachments must provide all authentication factors for both purposes, either directly or by reference.  

This discussion delineates how the two authentication factor purposes are satisfied in an IPEX, specifically in the `grant` message.

## Authentication Factor Types

Three types of authentication factors are allowed for each purpose (Issuer and Grantor).  These are as follows:

- Registry (TEL) event anchor, which in turn is anchored in KEL
- KEL anchor
- Signature

The preferred authentication factor is a Registry Anchor, but some applications can use either a direct KEL anchor or a bare attached signature.  Both the registry and KEL anchor enable perpetual verifiability and detectability of impersonation fraud. Whereas a bare signature does not. In addition to perpetual verifiability and detectability of impersonation fraud, a registry anchor can also support ACDC's lifecycle state. Typical lifecycle states include revocation. For these reasons - perpetual verifiability, fraud detectability, and lifecycle state (revocability) the preferred authentication factor is a registry anchor.

In light of this preference, when multiple authentication factors are either required or provided for the same purpose, the highest-preference factor is used, and the others are ignored. For example, if a Registry anchor is required, then any provided KEL anchor and/or signature are ignored.  Otherwise, if a Registry anchor is not required but a KEL anchor is provided or required, then a signature is ignored. A signature is allowed only when neither a Registry anchor nor a KEL anchor is provided or required.  

In a given presentation, the authentication factor used by the Issuer and the Presenter of a given ACDC may differ.

##  Proof of Freshness (Timeliness) in an IPEX

### Background

An asymmetric-key digital signature's (signature for short) major weakness is its vulnerability to replay attacks.  A digital signature attached to the document it signs is an ersatz bearer token. Any holder of both the document and its signature can replay them, and the signature will verify. All the signature proves is that, at some point in the past, the holder of the private key created it. It does not, at the time of presentation, prove that the holder of that signature is the same entity that currently controls the private key originally used to generate it.

The Issuer-Holder-Verifier model of verifiable credentials (VCs) assumes asynchronous use of the VC. It makes no assumptions about a secure session that could protect against replay attacks. Thus, usage can't rely on a secure session to protect against replay attacks.  In KERI Suite land wrt IPEX, the term holder is somewhat ambiguous. The precise term is Grantor, where Grantor is the controller of the sender AID in the exchange message with the `grant` route, also called the `grant` message.

There is some terminological baggage associated with the terms Issuer-Holder-Verifier and presentation from other communities such as the W3C or Sovrin for VCs.  In these other communities, only the holder of a VC is not the Issuer in general but is the subject of the VC, and only the holder may present it. The Issuer does not. The holder is most like an Issuee in Keri/ACDC land.   In KERI Suite land, the term Presenter is a euphemism for Grantor and may either be the “holder” Issuee or the Issuer.  In KERI Suite land, an ACDC may be untargeted, i.e., does not have an Issuee.  While the specificity of Issuers and Issuees may seem complicated, it enables a more precise security policy for verification.

To clarify, the actual presentation/issuance happens via the `grant` message. The sender of the `grant` message is the Grantor, which is the Presentor in W3C land.

This typically means that the Grantor must prove at the time of the grant that it controls either the Issuer or Issuee AID of the associated ACDC (there are exceptions). This proof can be called a fresh or timely presentation, but more specifically a fresh or timely “grant”.

This gets complicated because other messages besides the `grant` also require proof of freshness to prevent replay attacks, but the sender's role and purpose may differ.  In general, however, the controller of the sender AID of a given exchange message must prove fresh (timely) control over its AID at the time of transmission to protect against replay attacks. 

So we have the following potentially confusing euphemisms for “presenter” in an IPEX: Applicant, Offerer, Agreent, Grantor, Admittant.  But Grantor is the important one, and unless otherwise specified, the Presenter is the Grantor. To further avoid ambiguity, an IPEX exchange has a Discloser and a Disclosee, and each IPEX message sent by the Discloser to the Disclosee may contain a limited disclosure leading up to the final disclosure via the Grant.  The Discloser uses ‘offer’ and ‘grant’ messages, so it is the Offerer and Grantor. The Disclosee uses ‘apply’, ‘agree’, and ‘admit’ messages, so it is the Applicant, Agreent, and Admittant.

In KERI Suite land, we use AIDs as cryptographic identifiers controlled by a keystate defined by a KEL; Typically, in a presentation exchange, the proof of control over the Presenter’s (Grantor’s) AID at presentation (grant) time is where the Presenter’s AID is the Issuee AID of the ACDC being presented (granted). However, in an Issuance exchange, the Presenter is the Issuer of the ACDC being presented. Therefore, in this case, proof-of-control over the Presenter’s AID is proof-of-control over the Issuer AID of the ACDC being presented.  But when a bespoke ACDC is issued for the presentation, then proof-of-control over the Presenter’s AID is proof-of-control over the Issuer AID of the bespoke ACDC, which must be the same as the Issuee AIDs of the ACDCs linked or chained to the bespoke issued ACDC.  Thus IPEX is an appropriate designation since any given presentation exchange may also be an issuance exchange.

To elaborate, the controller over the Presenter’s (Grantor’s) AID must, at the time of the IPEX exchange, provide fresh proof of control over its AID. That AID must be either the Issuer or Issuee AID of the origin node of the ACDC DAG being presented. Proof of control means making a verifiable cryptographic commitment using the current private keys controlling that AID. One such commitment is a signature using the current keystate of the AID. This commitment must be fresh relative to the time of use via presentation of that origin node ACDC. This commitment is an authentication factor.

###  IPEX Exchange Freshness (Timeliness) Proof via KRAM

V2 IPEX uses KERI exchange messages (`xip` or `exn`). A full transaction is a set of exchange messages initiated with a `xip` and then followed by zero or more `exn` messages hash-chained back to that `xip`. KRAM (Keri Request Authentication Mechanism) provides replay-attack protection, applied to each `xip` or `exn` individually and in combination to the full transaction set. 

KRAM uses the following fields: a message SAID `d` field, a sender AID `i` field, a receiver AID `ri` field, a salty nonce `u` field, and a datetime stamp relative to the receiver's clock `dt` field from a given exchange message. With KRAM, each message in a transaction has an individual timeliness window, and the set of messages in that transaction has a combined timeliness window. Simply put, KRAM is a scalable, noninteractive, replay-attack protection mechanism. KRAM ensures that a message cannot be replayed within a timeliness window and cannot be played at all outside that window. Thus, there is only ever one "play" of the message or not at all. That play must lie within a timeliness window; this ensures freshness.

A given exchange message is accepted by a layer above KRAM for further processing only if it provides proof of freshness or timeliness. The proof has the following properties:  

- The accepted message was signed with the current key state of the sender AID as the first play of that message within the timeliness window. That first play is cached until the window expires, preventing a replay within that window. This enables the timeliness window to be large. It could be hours or days, only limited by memory.
- The datetime stamp and timeliness window are both relative to the receiver's clock. The sender cannot therefore lie about the datetime in the message. It must be fresh (timely), meaning the datetime must lie within the window relative to the receiver's clock. A message that is delayed in transmission long enough to fall outside that window cannot be played, much less replayed. Reception of a replay of that message within that window is either ignored or, if the sender AID uses multisig, a "replay" with a different signature can contribute that signature to the escrowed message, which is not first played until the escrow collects a threshold-satisfying number of signatures. This elegantly solves the multi-sig problem without requiring a pre-protocol to collect signatures. The receiver's KRAM escrow does the signature collection.
- The signatures by the sender are passed along with an accepted message
- Any other signatures by other AIDs that were attached to the message are also passed along with any accepted message.

## Multiply Endorsed IPEX message

A KERI message can be multiply endorsed by attaching signatures from other AIDs that are not the sender AID, or by attaching seal source references to other AIDs that are not the sender AID. We call these non-sender endorsements.
A seal source reference provides an anchored or perpetually verifiable endorsement, while a bare signature provides an ephemeral endorsement. A bare signature (i.e. by itself) is not trustable (verifiable) once the key state used to create it is no longer valid (i.e. a key rotation has occurred). Perpetual verifiability requires another proof mechanism that remains verifiable despite key rotations.

A non-sender endorsement attachment uses one of several CESR group codes. 

#### Attached Signatures for Multiply Endorsed IPEX

When a non-sender endorsement is an attached signature or signatures, it uses one of the CESR group codes for attached signatures. Typically, ACDCs with perpetual verifiability use transferable AIDs, each with a KEL. So we can assume transferable AIDs for this discussion, but non-transferable AIDs are also supportable.

The appropriate group codes for attaching signature endorsements from a non-sender transferable AID are as follows:

```
TransIdxSigGroups: str = '-X'  # Trans Indexed Signature Group(s), pre+snu+dig+ControllerIdxSigs of qb64.
BigTransIdxSigGroups: str = '--X'  # Big Trans Indexed Signature Group(s), pre+snu+dig+ControllerIdxSigs of qb64.
TransLastIdxSigGroups: str = '-Y'  # Trans Last Est Evt Indexed Signature Group(s), pre+ControllerIdxSigs of qb64.
BigTransLastIdxSigGroups: str = '--Y'  # Big Trans Last Est Evt Indexed Signature Group(s), pre+ControllerIdxSigs of qb64.

```
These groups provide the endorser (signer) AID. This is necessary because these other endorsers are, by definition, not the sender of the message, so we can't use a group code that assumes the endorser (signer) is the sender.

The TransIdxSigGroup and BigTransIdxSigGroup also provide the sequence number and SAID of the establishment event from which the keystate was drawn to create the signature.

The TransLastIdxSigGroups and BigTransLastIdxSigGroups contain only the signer's (endorser's) AID. The key state is
then pulled from the last (current) establishment event in the KEL of the AID.

This full set of CESR group codes also supports both non-transferable AIDs. These are not shown here. However, as a special case, a non-transferable AID can provide a (non-sender) endorsement via the appropriate attached signature group code.

#### Attached Seal Source References for Multiply Endorsed IPEX

The appropriate group codes for attaching signature endorsements from a transferable AID are as follows:

```python
    SealSourceTriples: str = '-T'  # Seal Source Triple(s), pre+snu+dig of source sealing or sealed event.
    BigSealSourceTriples: str = '--T'  # Seal Source Triple(s), pre+snu+dig of source sealing or sealed event.
    SealSourceLastSingles: str = '-U'  # Seal Source Couple(s), pre of last source sealing or sealed event.
    BigSealSourceLastSingles: str = '--U'  # Big Seal Source Couple(s), pre of last source sealing or sealed event.
```

These groups provide the endorser (sealer/anchorer) AID. This is necessary because these other endorsers are, by definition, not the sender of the message, so we can't use a group code that assumes the endorser is the sender.

### KRAM Augmentation to Support Multiply Endorsed IPEX

When the AID in an attached endorsement group is the sender AID, then KRAM validates it. 

If that attached group provides a signature that specifies or defaults to the sender's current keystate, KRAM verifies the signature and, if verified, accepts it; if the signature does not verify against the current keystate, KRAM drops it. If the group specifies a different keystate than the current keystate, KRAM drops the whole group.

If that attached group provides a seal source reference for the sender, then KRAM validates the reference against the SAID of the message. If the reference does not validate, then KRAM drops the reference.

When the AID in the endorsement group (either signature or seal source reference) is not the sender AID, then the group is passed along.

This means that any AID, not only the message sender, can generate a set of endorsements and attach them to the message. These are passed along to IPEX processing.

Importantly, because non-sender endorsers endorse the same message as the sender, KRAM's timeliness (freshness) properties also apply to non-sender endorser AID commitments, with one caveat.

The caveat is that the exchange processor must validate non-sender endorsements against the specific exchange message to which they are attached. KRAM does not do the validation; it merely passes along the endorsement. 

Therefore, to support multiply endorsed IPEX, the IPEX processor MUST validate all non-sender endorsements after KRAM.

For non-sender signature endorsements, all signatures must be verified against the endorser's current key state as applied to the IPEX message. Drop any signatures that do not verify. If the set of verified signatures for a given endorser do not meet the threshold for its current key state, then the endorsement is invalid.

For non-sender seal source reference endorsements (anchors), the reference must be verified against the KEL of the endorser as applied to the SAID of the IPEX message. If the SAID of the IPEX message is not found in the referenced event in the endorser's KEL, the endorsement is invalid.

When both a set of signatures and a seal source reference are provided for a given non-sender endorser AID, then the seal source reference is preferred. If the seal source reference is valid, then the set of signatures is ignored. If the seal source reference is invalid, then the set of signatures is validated.


With that caveat, non-sender endorsers get all the properties of KRAM replay-attack protection and timeliness that sender endorsers do. This includes multisig and anchored endorsements.

From a security properties perspective, with this caveat satisfied, a multiply-endorsed `grant` exchange message provides a freshness proof for any number of endorsing AIDs, not merely the sender AID. This supports a simultaneous 'fresh' presentation of multiple ACDCs where proof of freshness is provided for all the endorser AIDs, not merely the sender. 

To reiterate, the only extra work needed is that the exchange processing has to be multiple-endorsement-aware.

Technically, in an IPEX, the "presentation" is the `grant` exchange message. A valid IPEX could consist of only a `grant` message. This means that multiple endorsements cannot depend on some combined effect of multiple messages. Multiple endorsements must be verifiable with only the attachments to a given IPEX message, typically the `grant` message. The other messages in an IPEX are meant to negotiate, using graduated disclosure, the terms and degree of disclosure, culminating in the `grant` message as the disclosure or presentation, and therefore do not need multiple endorsements. In this case, the controller of the sender AID may use that AID to negotiate on behalf of other AIDs under its control. Those other AIDs do not have to be disclosed until contractual protection is in place, which coincides with the disclosure of the multiply endorsed `grant` message.

## Timeliness Tethered Presentations

As described above, in an IPEX, KRAM provides a fresh (timely) proof of control over the sender AID of each message. 
With the augmented validation described above, this fresh (timely) proof of control may be extended to other non-sender endorser AIDs of a given message. All the endorsers, including the sender of the `grant` message, form a set of Grantors. Without loss of specificity, unless otherwise indicated to avoid ambiguity, the term Grantor refers to this set of Grantor AIDs.

To elaborate, the Grantor AIDs are the set of endorser AIDs to the `grant` message. This always includes the sender AID of the `grant` message. A fresh proof of control applies to any instance of any Grantor AID that appears in any document that the `grant` message cryptographically commits to. Primarily, this happens via the origin `o` field in the attribute `a` block of the `grant` exchange (`xip` or `exn`) message. The origin `o` field value is the SAID of the origin ACDC of the DAG (Directed Acyclic Graph) of ACDCs granted by the `grant` message. Because every ACDC in the DAG originating at the origin ACDC is also cryptographically linked via a set of cryptographically verifiable edges from the origin ACDC. The appearance of the origin ACDC's SAID in the origin `o` field of the `grant` message makes a verifiable commitment to every ACDC in its DAG. 

Therefore, when any Grantor AID appears anywhere in the DAG of ACDCs originating at the origin ACDC, then we say that that ACDC is *tethered* to the `grant` message in the exchange. This means that the fresh (timely) proof-of-control over any Grantor AID provided by KRAM/IPEX on the `grant` message applies to the appearance of that same AID in any ACDC in the DAG. These appearances are thereby *tethered* to that fresh (timely) proof of control. 

Tethering implies no other relationship besides fresh (timely) proof-of-control over an AID so tethered. Tethering is most relevant when the DAG of ACDCs granted is meant to convey one or more entitlements to the Issuees of those ACDCs.  Tethering means that, at the time of the grant (presentation), the entitled party has demonstrated fresh proof-of-control over their AID. This protects against a replay attack by an imposter of the "presentation" or grant of that entitlement.  The combination of KRAM/IPEX timeliness, multi-endorsement processing on the `grant` message for its Grantors, the cryptographic commitment to the DAG via the origin ACDC SAID in the origin `o` field, and the appearance of any Grantor AID in any of the ACDCs together provides a tether from the grant to that Grantor AID in that ACDC.

### Examples

Let's examine how tethering might work in practice.

#### Tethered Entitlement via Single ACDC Presentation
Suppose a party named Bob has been issued an entitlement from Amy via an ACDC. Let's say it's a coupon for a movie ticket. Let Amy be the Issuer of the ACDC that conveys the entitlement (coupon), and therefore Bob is the Issuee (Holder) of the coupon. Suppose Bob presents that entitlement to Cal (Verifier), who provides access to the resources authorized by that entitlement. In this case, that means Cal will let Bob enter the theater to view the movie.

When Bob presents the coupon as ACDC via IPEX, Bob is the Discloser and Cal is the Disclosee. The actual disclosure happens via the `grant` message, thereby making Bob the Grantor and Cal the Grantee.  All of Amy, Bob, and Cal would rather not let an imposter Ian steal the coupon from Bob and use it to view the movie. So Cal wants to verify that Bob controls the Issuee AID when Bob presents (grants) the coupon to Cal.  Otherwise, Ian may have stolen the coupon and is replaying it.  This is relevant for digital entitlements because they are not paper. Any party that observes a presentation (grant) of the ACDC could replay it later. So we need a timeliness proof to protect against such fraudulent replay.

Bob puts the SAID of the ACDC (coupon) in the origin `o` field of the `grant` exchange message. Assuming no negotiation, the IPEX starts with the `grant` using a `xip` message. Cal applies KRAM to that message, which forces timeliness on the transmission (presentation) of the `grant`.  Cal must verify that Amy authentically issued the coupon (we will skip those details for now).

Cal then checks that the Issuee of the ACDC Coupon is the sender AID of the grant (thereby verifying that it is tethered). Since it's tethered, Cal can assume the coupon isn't being played by anyone but Bob, preventing imposter fraud. Cal can then log that play. If the coupon is one-time use, Cal must record its first use. If it's a multi-use coupon, then Cal has ensured that each use is by Bob, not Ian.

#### Tethered Entitlement via DAG Presentation

This is the same case as before with Amy, Bob, Cal, and Ian but augmented because Bob wants Cal to agree not to assimilate his movie-viewing behavior before or as part of Bob's disclosure of the coupon to Cal. To do this, Bob creates (issues) a bespoke ACDC with those custom terms in its rule section. Bob is therefore the Issuer of that bespoke ACDC.  The bespoke ACDC is linked by an edge to the coupon Amy issued to Bob.  

Bob puts the SAID of the bespoke ACDC in the origin `o` field of the grant exchange message. If we assume no negotiation, then the IPEX starts with the `grant` using a `xip` message. Cal applies KRAM to that message, which forces timeliness on the transmission (presentation) of the `grant`.  

Cal must verify that the bespoke ACDC was authentically issued by Bob and that the linked coupon was authentically issued by Amy (we will skip those details for now).  Given that the sender of the `grant` message is Bob, and the Issuer of the bespoke ACDC is also Bob (same AID), one might be tempted to say that the `grant` message timeliness proves the authenticity of the Issuance of the bespoke ACDC. The problem is that the ACDC has its own mechanisms for determining its authentic issuance. So, for separation of concerns, we don't replace or supersede those mechanisms.

Cal then checks that the Issuee of the ACDC Coupon is the sender AID of the grant (thereby verifying that it is tethered). Given it's tethered, Cal can assume that the coupon is not being played by anyone but Bob and therefore prevents imposter fraud. Cal can then log that play. If the coupon is one-time use, then it's up to Cal to record its first use. If it's a multi-use coupon, then Cal has ensured that each use is by Bob, not Ian.

#### Tether Issuance via DAG Presentation

Let's revisit the example above, but consider the IPEX exchange that Amy and Bob engage in, where Amy issues the coupon ACDC to Bob with a `grant` message.  In this case, Amy is the Grantor, not Bob.

Amy puts the SAID of the ACDC (coupon) in the origin `o` field of the `grant` exchange message. If we assume no negotiation, then the IPEX starts with the `grant` using an `xip` message. Bob applies KRAM to that message, which forces timeliness on the transmission (presentation) of the `grant`.  Bob must verify that Amy authentically issued the coupon (we will skip those details for now).  As above, don't replace or supersede that authentication with tethering.

The question arises, then: does tethering provide any usefulness in this case?  Amy's AID appears as the Issuer of the origin ACDC in the grant. So it is tethered to the fresh proof of control. Because the ACDC (coupon) itself must have a separate proof of authenticity independent of the `grant` message, there may be no reason to apply tethering here. A given EGF for some types of ACDCs issuances may find a reason to validate tethering of the Issuer to prevent replay of the issuance. One case might be to provide an extra layer of defense against compromise of the Issuer Amy. This is most applicable when the authentication factor used to authenticate the issuance is a bare signature from Amy, not an anchored proof such as a registry (TEL) or a KEL anchor.  In that case, current proof of Amy's control over her keystate at the time of issuance is essential. Bob should refuse a stale signature captured from a prior issuance to protect himself.


#### Tethered Agreement via IPEX

The previous two examples covered the primary use case for tethering: providing fresh (timely) proof-of-control of the Issuee AID of an entitlement at the time of presentation (use) of that entitlement via a `grant` message.  

There may be other use cases where the proof-of-control needs to be applied to other AIDs (not Issuee) in the granted DAG. The EGF for those use cases would need to spell those out.  

In general, because each message in an IPEX (apply, offer, agree, grant, admit) is digitally signed, it could be a legally binding agreement by the sender to whatever is enclosed in that message or committed to by the enclosure of the SAID of some attached ACDC  (the ACDC is thereby cryptographically bound to the message).  In the latter case, where an attached ACDC is bound to the exchange message, then validation of *tethering* of a Grantor AID to an AID that appears in the bound and attached ACDC could be essential.

##  Perpetually Verifiable IPEX

A special case of an IPEX is when the parties want the agreements attested to by the IPEX to be perpetually verifiable. A message is perpetually verifiable if its authentication factor is either an attached reference to a direct KEL anchoring seal or a reference to a Registry (TEL) whose events are bound to the message and are in turn anchored via anchoring seals. 

As mentioned above, KRAM supports direct KEL-anchored messages, not merely signed messages. A direct KEL-anchored message authentication factor is indicated by the Sender by attaching a reference to the sealing KEL to the message via a seal source group code. This reference may be one of the following CESR codes (v2):

```python
    SealSourceCouples: str = '-S'  # Seal Source Couple(s), snu+dig of source sealing or sealed event.
    BigSealSourceCouples: str = '--S'  # Seal Source Couple(s), snu+dig of source sealing or sealed event.
    SealSourceTriples: str = '-T'  # Seal Source Triple(s), pre+snu+dig of source sealing or sealed event.
    BigSealSourceTriples: str = '--T'  # Seal Source Triple(s), pre+snu+dig of source sealing or sealed event.
    SealSourceLastSingles: str = '-U'  # Seal Source Couple(s), pre of last source sealing or sealed event.
    BigSealSourceLastSingles: str = '--U'  # Big Seal Source Couple(s), pre of last source sealing or sealed event.
```
When the endorser is not the sender, use one of the last four codes. These supply the non-sender AID. The first two codes assume the sender AID as default.

When the SealSourceCouple form is used, the sender AID is implied.  A SealSourceTriple includes the endorser AID, which may be the same as the sender. 

To pass KRAM/IPEX validation, an anchor must be created in a timely fashion relative to the IPEX exchange. Each exchange message as part of the IPEX must be created, then anchored, and then pass the receiver's KRAM. Each exchange message includes a timestamp relative to the receiver's clock, so the subsequent anchor in an endorser's KEL for that message must be timely with respect to that timestamp, or it won't pass KRAM/IPEX validation.

The limitation of KRAM by itself is that it gives neither party a way to signal to the IPEX that the relevant messages MUST be perpetually verifiable and hence anchored. Anchoring via KRAM is solely determined by the set of endorsers at the time the message is transmitted.

For the purposes of perpetual verifiability for an IPEX, the relevant messages that MUST be anchored are the `agree`, `grant`, and `admit` messages.  The `agree` message is where the disclosee formally *agrees* to the terms of the offer. Because the `offer` is hash-chained to the `agree` via its SAID appearing as the value of the agree's prior `p` field. It is therefore cryptographically verifiable that it is included in the `agree`.  

When the `agree` is anchored in the KEL of the Agreent (Disclosee or sender of the `agree` message), the Discloser (the receiver of the `agree` message) has a perpetually verifiable proof of that agreement prior to making its disclosure.

Likewise, when the `grant` message is anchored in the KEL of the Grantor (Discloser or sender of the `grant` message, the Agreer (Disclosee or receiver of the `grant` message) has a perpetually verifiable proof of what was granted and, as importantly, what was not granted.

Finally, when the `admit` message is in the KEL of the Admittant (Disclosee or sender of the `admit` message), the Discloser (receiver of the `admit` message) has perpetually verifiable proof that the Disclosee (receiver) did indeed receive the granted disclosure. This removes plausible deniability regarding the Disclosee's knowledge of the disclosed information and could trigger safe harbor protections for its use by the Disclosee (Admittant).

Anchored exchanges provide perpetually verifiable, contractually protected disclosure. This has several potential benefits to the ecosystem. Safe harbor provisions incentivize compliance with the terms of the executed contract. The Legal burden of proof for enforcement is easier to meet and automate when the agreement is perpetually verifiable.

The grant message authentication is complicated when it's multiply endorsed. In this case, a set of Grantors is provided, one of whom is the sender of the `grant`, and each endorsement may be either an anchor of a threshold-satisfying set of signatures.  Any IPEX grant message Grantor whose authentication factor is an anchor is also tethered to any ACDC whose Issuee matches the Grantor.  

### Registry Anchored Granted ACDCs

The issuer of a given ACDC can signal to a verifier that the `grant` message in an IPEX  MUST be anchored in a presentation registry controlled by the Issuee of that ACDC at the time of presentation by the Issuee of that ACDC when the Issuee is the Grantor of that IPEX. 

That signal is provided when both a non-empty `rd` field and a valid AID as the Issuee in the issuee `i` field are provided at the top level of the ACDCs attribute `a` section (not the top level of the ACDC itself) and the `rd` field at the top level of the ACDC is not empty.  To clarify, if no Issuee is provided via the issuee `i` field at the top level of the attributed `a` section of the ACDC, then the presence of the `rd` field is not a requirement for anchoring the grant message. Likewise, if the `rd` field at the top level of the ACDC is missing or empty, then the `rd` field at the top level of the `a` section refers to an ACDC state registry, not an IPEX presentation registry. 

This latter use case enables a given Issuer to hide the ACDC state registry ID inside the partially disclosable attribute `a` section. This means that a presentation registry may not be used with an ACDC that uses the `rd` field in the attribute `a` section as an Issuer-controlled ACDC state. This hiding case is unambiguously indicated by an empty or missing `rd` field at the top level of the ACDC. This hiding case is a special case. In general, the `rd` field at the top level of an ACDC need not be disclosed until contractual protection is in place and the ACDC that carries it does not need to be public. The registry is public, but the binding in the registry to the ACDC is blinded. Therefore, the only benefit of hiding the ACDC state registry identifier in its attribute `a` section is when the ACDC in compact form is made public, independent of any given presentation, and the fact of it having a registry must remain undisclosed. It is not clear that this is a worthy use case. So we might want to forbid using the `rd` field in the attribute section for an ACDC state registry and only allow it for other types of registries, such as a presentation registry. This would enable a presentation registry for an ACDC without an Issuer state registry. Not sure if either is a worthy use case. 

Currently, the ACDC v1 spec allows the `rd` field at the top level of the attribute `a` section of an ACDC to be used as a blinded registry for the ACDC itself, not merely for anchoring exchanges. One way to resolve that ambiguity is for the EGF for a given ACDC type to specify the purpose of the `rd` field in the attribute `a` section.  Then all use cases would be allowed. The default, if not otherwise specified, should be for anchoring presentation exchanges, i.e., using presentation Registries. The language in the spec could use some clarification.  
We may want to limit the use case to only presentation registries in the v1.1 spec.

In the presentation registry use case, the value of the registry `rd` field in the attribute `a` section MUST be the SAID of the `rip` event of the associated presentation Registry. The Issuee of the associated ACDC, who is also a Grantor of that IPEX, MUST control this registry. The value of the ACDC SAID field in the blinded attribute block of the latest non-vacuous event in that presentation Registry must be the SAID of the `grant` message.

Recall that in order to pass KRAM, a `grant` message anchor must happen in a timely fashion relative to the IPEX exchange. The `grant` message as part of the IPEX exchange must be created, then anchored, and then pass the receiver's KRAM. The `grant` message includes a timestamp relative to the receiver's clock, so the subsequent anchor in the presentation registry must be timely with respect to that timestamp, or it won't pass KRAM.

#### Fraud Protected Registry

A presentation registry provides the Grantor with an Issuer-Verifier-protected mechanism to detect compromise of its signing keys when used for fresh proof-of-control over its AID in IPEX.  Because the events in the Registry must be anchored in a Grantor's KEL, the Grantor can detect events it did not create, thereby indicating compromise of its signing keys. The Issuer must include this signal in the ACDC it issues so an imposter Grantor can't decide NOT to register the presentation (grant) unless the imposter also compromises the Issuer key state. To clarify, an imposter that merely compromises the Grantor's signing infrastructure can't avoid the requirement without also compromising the Issuer. For ACDCs that the Grantor self-issues, this is only a vulnerability if the Issuance authentication factor is merely an unanchored signature. An imposter who controls the Grantor's signing infrastructure can create ACDCs whose authentication factor is an unanchored signature.

The verifier then enforces the protection by not accepting presentations (grant messages) that are not so anchored whenever there is an `rd` field and `i` are provided in the issued ACDC.

When multiple ACDCs in the granted DAG each have a different presentation registry ID field value in the `rd` field in their attribute `a` section, the verifier must verify all of the anchors.

Notwithstanding the former, because the `grant` message `o` field value is the SAID of the origin node of the granted ACDC DAG, anchoring the exchange in one presentation registry provides a perpetually verifiable proof of the presentation of all ACDCs in that DAG, but not necessarily a fresh proof of control for all Issuee AIDS in that DAG.

Multiple anchors in different registries provide multiple vectors of detectability and multiple fresh proofs-of-control; the perpetual verifiability is redundant.


#### Correlation Protected Registry

The previous section noted that an impersonation-protected registry anchor only protects the Grantor from impersonation when the Issuer differs from the Grantor. Otherwise, the impersonator of the Grantor who has successfully compromised the Grantor's signing infrastructure could self-issue unanchored ACDCs that do not require a presentation registry and make an unanchored IPEX grant.

Notwithstanding the former, there is another reason for a Grantor to self-issue a bespoke ACDC as the origin node of a given granted DAG and include a required presentation registry by populating the Issuee `i` and Registry SAID `rd` fields in the top level of its attribute `a` section. This makes an anchored IPEX non-correlatable to the Grantor in the following ways.  The IPEX can be structured so the actual ACDC SAIDs in the granted DAG appear only in the `grant` message itself. An anchored `agree` or `admit` message does not include them. Therefore, a correlator could not walk the public KEL of the Disclosee and discover them. Likewise, when the anchor of the `grant` message uses a blinded state Registry, the SAID of the grant message only appears in the blinded attribute block of the associate registry (TEL) event. So a correlator walking the Grantor's KEL would not be able to observe it.

Consequently, a Grantor may choose to make its `grant` message perpetually verifiable yet uncorrelatable by anchoring via a presentation registry instead of a direct KEL anchor. This can be done even when none of the ACDCs in the DAG provide for a presentation registry with it as Issuee. The grantor just uses a self-issued bespoke ACDC as the origin of the granted DAG and provides for the presentation registry in it. This means the Grantor is both Issuer and Issuee of that bespoke ACDC. 

To clarify, the bespoke ACDC itself must use a blinded state Registry for its Issuer authentication so that the SAID of its ACDC is not correlatable. If it uses a direct KEL anchor, then that anchor itself would be correlatable. If it used a bare attached signature for Issuer authentication, then the ACDC itself would not be perpetually verifiable despite using a perpetually verifiable presentation registry. 


### New Anchored Exchange `ax` Field

Recall from above that KRAM by itself gives neither party a way to signal to the IPEX that the relevant messages MUST be perpetually verifiable and hence anchored. Anchoring via KRAM is solely determined by the sender at the time the message is transmitted.

We propose a new field in the attribute `a` section of the `apply`, `offer`, and `grant` messages to signal that the IPEX MUST be anchored for perpetual verifiability.

The `ax` field value, when present, is a boolean: `True` or `False`. The signal is truthy only when the field is provided and set to `True`; otherwise, it's falsy. To clarify, if the `ax` field is missing, then there is no requirement to anchor the exchange.

An IPEX can start with either an `apply`, `offer`, or `grant` message. Therefore, the anchoring signal (a truthy `ax`) is allowed in any of them.  Technically, an `offer` with a truthy `ax` is a commitment by the Discloser which when accepted with an `agree` method would provide the signal even if the associated `agree` or `grant` messages did not have an `ax` (missing) but to make it perfectly unambiguous, when an anchored exchange is required and agreed upon then the associated `agree`, `grant`, and `admit` messages when provided must include a truthy `ax` field.

Satisfying a requirement to anchor an exchange means that the Grantor must anchor its `grant` message and the Applicant must anchor both its `agree` and `admit` messages when either is provided. Because an IPEX may start with or include a `grant` without an enabling `agree`, there is an exception. When an IPEX starts with or includes a `grant` without an enabling `agree`, then only the `grant` and `admit` messages are anchored. 

To elaborate,  when an IPEX starts with an `apply` that includes a truthy `ax` field, then the Discloser signals its acceptance by responding with either an `offer` with a truthy `ax` or a `grant` with a truthy `ax`.  When an IPEX includes an `offer` with a truthy `ax` then the Disclosee signals its acceptance by responding with an agree with a truthy `ax` and also must include a truthy `ax` in its `admit`. 

Notwithstanding the presence or absence of the `ax` field, when an ACDC in a granted DAG meets the requirements for a Issuer signaled presentation anchor registry (see above) via populated `rd` and `i` fields at the top level of its attribute `a` field block when the Grantor of the IPEX is the Issuee then that `grant` message MUST be anchored in that registry for the `grant` to be valid. Otherwise, the presumption is that the grant is fraudulent. Any verifier (Disclosee) accepting such a `grant` is presumptively in violation of any contractual or regulatory protection afforded to the real Grantor by colluding with a fraudulent grantor.

As mentioned above, there are two ways to satisfy the anchoring requirement for the `grant` message. The first is to directly anchor the SAID of the `grant` message in the KEL of the Grantor. The second is to anchor the SAID of the `grant` message in a presentation registry controlled by the Grantor.  To use a presentation registry for the anchor, the registry must be pre-specified in one of the ACDCs in the granted DAG. Otherwise, we would need yet another field in the `grant` message to specify it. There is no security advantage to using a presentation registry specified by the Grantor, as the main purpose of a presentation registry is to allow the Grantor to detect a compromise of its signing key infrastructure for presentations (IPEX).  

Thus, the main use case for impersonation fraud detection of the Grantor requires a pre-specified presentation registry by an Issuer different from the Grantor, which must be prespecified in one of the ACDCs not in the `grant` message.

The other use case for a presentation registry is a more correlation-resistant, perpetually verifiable grant, though it does not provide fraud protection. In this case, either the grant message or a bespoke ACDC issued by the Grantor could be used. Given that we must provide tooling for a presentation registry specified in an ACDC, and we must provide tooling for bespoke self-issued ACDCs, we should not add special tooling for this special case; instead, we should use the existing tooling for a bulk-issued ACDC to handle it.

## Authentication Factor Provisioning

This section answers how to unambiguously identify which type of authentication factor is used for each authentication (Issuer and Issuee).

###  ACDC Issuer Authentication Factor

Every ACDC in the granted DAG for some Grantor MUST have an Issuer authentication Factor. The Issuer decides what type of authentication factor it will provide when it publishes the Issuance of that ACDC. Often, the allowed Issuer authentication factor type is specified in an EGF for a given ACDC type. Typically, the provided Issuer authentication factor is an ACDC state Registry controlled by the Issuer. But in some cases, an ACDC may provide a direct KEL anchor or a bare attached signature as its Issuer authentication factor.

A granted DAG provides the granted ACDC via attached nested CESR substreams inside the CESR attachments to the `grant` message itself. Each ACDC in the DAG gets its own nested substream. That nested substream starts with the appropriate nesting group code followed by the serialized ACDC followed by its attachments. The Issuer authentication factor is provided in the attachments to the ACDC in its nested substream. This independently and unambiguously associates authentication factors with each ACDC in the DAG.

#### ACDC State Registry Issuer Authentication Factor

An ACDC in the DAG that uses an ACDC State Registry must provide for that by specifying the Registry ID in the `rd` field at the top level of the ACDC. This binds the ACDC to the Registry. The Registry ID is the SAID of the `rip` event for that registry. The Issuer of the registry event must match the Issuer of the ACDC. The Registry is not bound to the ACDC until a `bup` event is issued in which the ACDC field value is the ACDC SAID. At this point, a two-way binding exists between the ACDC and the Registry.

In order for a verifier to verify this type of Issuer authentication factor, the attachments to the ACDC in its nested substream attached to the `grant` message must include the unblinded attribute blocks of the last non-vacuous `bup` event in the registry and all the subsequent vacuous `bup` events. This allows the verifier to contact the Registrar or Observer that publishes the Registry to download the registry events, confirm the unblinded attribute block `blid` field values match those in the Registry, and then verify the anchors of those events in the Issuer's KEL. This provides a stateful, perpetually verifiable proof of authenticity.   If the ACDC also provides a presentation registry, the attachments to the ACDC in its nested substream will also include unblinded attribute blocks from the presentation registry. There is no ambiguity between the two sets of unblinded attribute blocks because their associated `blids` are unique to each registry.

#### Direct KEL Anchored Issuer Authentication Factor

In this case, the ACDC is provided as an attachment to the `grant` message. That attachment is a nested substream. This nested substream must include, in its attachments, a source seal reference to the event in the issuer's KEL where the anchoring seal for that ACDC may be found. The anchoring seal includes the SAID of the ACDC.

#### Bare Signature Issuer Authentication Factor

In this case, the ACDC is provided as an attachment to the `grant` message. That attachment is a nested substream. This nested substream must include, in its attachments, a threshold-satisfying set of signatures using the issuer's latest key state at the time of presentation. This means that as soon as the Issuer rotates its keystate, any ACDCs it issued using a bare signature as the Issuer authentication factor with a prior keystate become unverifiable and must be reissued. This is only useful for truly temporary use (ephemeral) ACDCs. These are not perpetually verifiable.


###  ACDC Grantor Authentication Factor

This summarizes the Authentication Factor for the `grant` message in an IPEX.

The Grantor may authenticate the grant using one of three mechanisms. 
- An attached signature on the `grant` message. 
- A direct anchoring seal in the KEL of the Grantor of the `grant` message SAID with an attached seal source reference to that anchoring event.
- An indirect presentation Registry with a blinded attribute block bound to the `grant` message SAID where that registry is specified in one of the ACDCs in the granted DAG. In that case, the associated attached substream for that ACDC includes, as an attachment to the ACDC, the unblinded attribute block of the latest non-vacuous `bup` event along with all unblinded attribute blocks of any subsequent vacuous `bup` events. This allows the Grantee to verify that the `grant` message SAID has been bound to the last non-vacuous `bup` in the registry. These attachments are not ambiguous because they include the `blid` field value that is published in the Registry for the associated event. This `blid` is universally unique to a given event in a given Registry. Moreover, KRAM does not recognize or process a presentation registry-based authentication factor for the `grant` message. To be accepted by KRAM, the `grant` message must be endorsed by each Grantor; this endorsement is either a signature(s) or a direct anchor in the KEL of the Grantor. Thus, a presentation registry-anchored `grant` message is always double-authenticated. First, using an attached signature or direct anchor reference; second, using a presentation registry.

When a Grantor agrees to use an anchored authentication factor as signaled by the appropriate appearance of a truthy `ax` field value in the attribute `a` block of one or more IPEX messages, then the Grantor MUST anchor the `grant` message using one of the two anchored authentication types described above. Anchoring via a presentation registry requires that one of the ACDCs in the granted DAG provide for such a registry. Otherwise, absent such a provision, the anchor MUST be a direct anchor in the Grantor's KEL. This rule has exceptions defined below when multiply endorsed.

Regardless of the truthy value of the `ax` field in any of the IPEX messages. If any ACDC in the granted DAG provides for a presentation registry where the Issuee of that ACDC is equal to any Grantor, then the `grant` message may be doubly authenticated, both with an attached Grantor endorsement to the `grant` message and via a presentation Registry anchor.  The presentation registry anchor is made perpetually verifiable by first anchoring the grant message to it, then providing the appropriate attachments via the attached message substream for that ACDC.
When the Grantor as Issuee for the required presentation registry is also the grant sender, then the Grantor MUST be doubly authenticated; otherwise, the grant message will not pass KRAM.

#### Anchoring Requirement Complication For Multi-Endorsed Grants

The above rules are unambiguous when there is only one Grantor, the sender of the `grant` message. The complicating factor is how to interpret the anchoring requirement via the presence of a truthy `ax` field in the grant when there are multiple Grantors. I.e., the grant is multiply endorsed.

The important use case is when that sender AID does not appear as an Issuee in any of the ACDCs in the DAG because the Issuee AID was meant to remain undisclosed until after contractual protection is in place. In this case, the important Grantor is not the sender but another Grantor. 

In a general sense, a requirement by the Disclosee that the `grant` in the IPEX be anchored may be satisfied if any Grantor anchors it. Importantly, in the use case where a given Issuee AID is meant to remain undisclosed until the `grant` message in order to provide contractual protection via a preceding `agree`, the Disclosee (Agreent) can not specify which Grantor specifically must anchor the `grant`.  Therefore, a requirement to anchor a grant may, in some cases, apply only to Grantors whose AIDs appear in the disclosed ACDC DAG, and those cannot be disclosed prior to the grant itself. In these cases, this means that unless the sender AID appears in one of the disclosed ACDCs, it may not be required to anchor the grant.

The complication is that, in delegation chains, Issuee AIDs that are not at the tail (leaf) of a delegation chain (tree) do not need a fresh proof-of-control over that AID. These non-tail (leaf) AIDs have delegated authority that is subsequently delegated to another, eventually terminating in the Issuee of a tail (leaf) ACDC. Only that tail needs a fresh proof-of-control at presentation to protect against replay attacks on the entitlement the chain of authority is authorizing. 

In other non-delegative use cases, a given presenter (Discloser) may be disclosing an ACDC inside the DAG on a to-whom-it-may-concern basis, merely attesting to knowledge of the ACDC and the Issuee AID within it, but not needing to satisfy any other attestation or proof-of-control requirements with respect to a given ACDC within the DAG. The Dossier spec use cases include many of these where an ACDC is presented as evidence, but the Issuee AID is unrelated to the Discloser's AID.

Given the range of possible interpretations where the `grant` must be anchored but the sender is not the anchorer, we choose not to define every possible combination; we define only the most important and leave the rest to the EGF or future development.

A simple resolution we propose is as follows:

In the event that the `ax` field in the grant is truthy, then either the sender must anchor the grant or the Issuee or Issuer of the origin ACDC of the grant must anchor the grant.

This allows both the Issuer and Issuee of the origin ACDC to be a non-sender AID.  

The anchoring requirement may be met in one of three ways: The sender anchors the grant, the Issuee of the origin ACDC anchors the grant, or the Issuer of the origin ACDC Anchors the grant. Let's apply these in the given order of priority. If any of these is satisfied, then the grant is anchored. 

To elaborate, a verifier must first check whether the sender anchored. If not, then check if the Issuee of the Origin anchored. If not, then check if the Issuer of the Origin anchored. If none of the above, then the anchoring requirement is not met. The anchoring requirement is a loose business logic requirement that may be enforced at a lower level than other business logic. It is enforced after the grant passes KRAM and IPEX multiple endorsement but before other business logic requirements.

A given ACDC could independently specify in its rules section that when anchoring is required, and it is the origin ACDC, then it must be anchored. This would not override the default processing but would give the Disclosee an exception for any terms of use.

I believe this rule covers most use cases where a given sender AID negotiates on behalf of another AID.  The only restriction is that in this case a non-sender Granter must be either the associated origin ACDC's Issuee or Issuer. The sender AID need not appear at all in the disclosed DAG. And the origin ACDC's Issuee or Issuer AID need not appear in the IPEX until the grant.  Recall that the main reason for a fresh proof-of-control is the provider of an entitlement to an Issuee is protected from a replay attack of a presentation of that entitlement by the Issuee. So when the sender is not the Issuee, but the origin ACDC's Issuee is the Issuee of the entitlement in question the anchoring requirement is met by the responsible party.  Because a bespoke ACDC is useful in many presentations where the Issuer of the bespoke ACDC is the Issuee of the actual entitlement, then when the Sender is also not the Issuee of the origin but the bespoke ACDC as origin is issued by the Issuee of a linked entitlement, the requirement to have the Issuer be anchored still satisfies the intended requirement that the Issuee of the entitlement anchor the grant. 

Let's consider the example above of the movie ticket coupon issued by Amy to Bob and presented to Cal.

Using this rule, suppose Bob creates another AID he controls labeled Sam (sender). 
When the controller of both Bob and Sam negotiates with Cal, Bob does not use his Bob AID; he uses his Sam AID. The resulting grant is then multiply endorsed by both Sam as sender and Bob as a secondary endorser. From Cal's perspective, the important endorsement is from Bob since Bob holds the entitlement, not Sam. 

Cal can then verify the freshness proof for Bob as issuee of the coupon because Bob will be one of the endorsers accepted by KRAM/IPEX, given the new processing support for multiply endorsed KRAM/IPEX.

To elaborate, Cal internally verifies both endorsements, but Cal's business logic for accepting the coupon presentation requires not merely that the grant be endorsed by any sender, but that the grant provides fresh proof of control from Bob, the coupon Issuee. This is specific to the ACDC coupon type, not the presentation in general. Therefore, Cal's business logic requires Bob's endorsement. If Bob does not endorse the grant but only Sam does, then the Grant will pass KRAM/IPEX but not pass Cal's business logic for the coupon.

Suppose Cal requires in the negotiation that the grant be anchored. Recall that, from Cal's perspective, the important endorsement is from Bob since Bob holds the entitlement, not Sam. But the rules we have specified only require that the anchor be one of the sender AID, the Issuee AID, or the Issuer AID of the origin ACDC.

Suppose that Bob anchors the grant. KRAM/IPEX will accept the grant as satisfying the anchoring requirement. Cal's business logic is that Bob, the entitlement Issuee, must anchor the grant. Cal's logic is also satisfied.

Suppose only Sam anchors the grant and Bob does not. The grant will pass the KRAM/IPEX logic and the default anchoring logic, but it will not pass Cal's business logic for anchoring.

But Sam should know this. Cal's business logic for accepting a coupon, as specified in the EGF for that ACDC type, would indicate that when presentation anchoring is required, anchoring must include an anchor by the Issuee of the Coupon ACDC, which in this case is Bob. So Sam would know to multiply endorse the grant and anchor the grant in Bob's KEL. Otherwise, the presentation passes KRAM, but the natural person who controls both Bob and Sam will not be admitted to the movie theater.

The logic works for either a direct presentation by Sam of the coupon issued to Bob as origin or an indirect presentation where the origin is a bespoke ACDC issued by Bob that links to the coupon. In both cases, Sam must multiply endorse the grant with Bob, and if the grant must be anchored, anchor it by Bob.

Furthermore, suppose the coupon requires a presentation registry for Bob the Issuee. Then, regardless of how Sam provides the grant. If the grant is not anchored in Bob's presentation registry, Cal will not accept it.

In this latter case, the grant does not have to be multiply endorsed because Cal's business-logic requirement for fresh proof of Bob's control over his coupon is met by the presentation-registry anchor, which must be timely for the grant presented by Sam to pass KRAM.  Thus, in many cases, a presentation-registry requirement enables Sam to negotiate and grant without explicit multiple endorsement by Bob. The implicit anchor requirement of the presentation registry that Amy imposed on the coupon enforces it.