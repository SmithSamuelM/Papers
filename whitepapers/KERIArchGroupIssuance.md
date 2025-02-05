# KERI Architectures for Group Issuance (KAGI)

This is a discussion about the architecture trade-offs and choices for different types of group issuance using the KERI stack. 

For this discussion, the *KERI stack* includes the set of libraries, i.e., the codebase that is integrated with and/or built upon the reference implementation of keri.py and licensed under either the Apache2 or the OWF FLOSS licenses. This includes the reference implementations of CESR, KERI, and ACDC or other less formal specifications such as KRAM, SAID, IPEX, BADA-RUN, and OOBI, as well as supporting libraries like Keria.py, SignifyTS, KAPI, and others that rely on or are dependencies of keri.py, such as hio.py.

By *group*, we mean a group of cooperating controllers who sign and/or seal issuances with and/or to the key state of their associated AIDs.

The motivation for this discussion comes from other discussions around IPEX and group multisig and the limitations of simple KRAM as applied to exchange, `exn` message used to negotiate over the issuance of ACDCs.

See for example:

KERI Request Authentication Method [KRAM](https://github.com/SmithSamuelM/Papers/blob/master/whitepapers/kram.md)

Human Agreement for Multisig Issuance [HAMI](https://github.com/WebOfTrust/keripy/issues/911)

Reliable End-to-End Application Layer Transactions [REEALT](https://github.com/WebOfTrust/keripy/discussions/909)

Issuance and Presentation EXchange [IPEX](https://trustoverip.github.io/tswg-acdc-specification/#issuance-and-presentation-exchange-ipex)



## Background

### Exigency and Expediency in the Evolution of the KERI Stack
One of the sources of frustration for those who have adopted KERI is that not all components in the KERI stack are at the same level of maturity. In the US Military, there is a formal classification for technology maturity called a Technical Readiness Level [TRL](https://asc.army.mil/docs/pubs/alt/2004/3_MayJun/articles/10_Use_of_Tech_Readiness_Levels_for_Software_Dev_200403.pdf). The reason for the unevenness in the maturity of KERI components is a result of the relative paucity of directly contributed resources towards its development. This is not to say that significant resources have not been contributed so far, but that the resources have been meager relative to the scope and scale of problems the KERI stack solves.

We want to carefully narrow the scope of what contributed resources mean. In this case, we are talking about contributions that have resulted in functioning code and features in the actual codebase of the KERI Stack. This excludes proprietary or reciprocal licensed implementations as well as deployments of the KERI stack.  Indeed, there have been sizeable investments in the use of the KERI stack, including both deployments and proprietary products built on top of the KERI stack. However, these are not directly contributing to the FLOSS libraries' feature set. Clearly, adoptions of KERI provide a virtuous feedback loop that may eventually result in direct contributions to the code base. But when talking about direct contributions to the FLOSS KERI Stack codebase, we measure those contributions in terms of market rates for the developer time of the associated developers.

One of the primary contributors to the codebase is GLEIF. GLEIF brought the vLEI to production in a relatively short period of time. The exigency of that implementation was limited time and limited budget relative to the scope and scale of implementation. This meant that the implementation of bare-bones features needed for the vLEI became expedient, whereas other features were not. This resulted in the uneven maturity of many of the features in the KERI codebase. GLEIF and the broader vLEI community continue to make significant direct contributions to the code base in support of the vLEI, which further contributes to continuing unevenness in the maturity levels of features with respect to non-vLEI applications or even non-GLEIF relevant components in vLEI applications.

This explains why some of the specified and/or design features of KERI are not as mature as some adopters would like. One purpose of this discussion is to make the case for which components would be most expedient for others to contribute to in order to leverage the KERI Stack better.

### Primary Security over an AID
Without loss of generality, unless confusing otherwise, we will refer to the *KERI Stack* as KERI.

The primary security of KERI is based on the management of cryptographic non-repudiable digital signing key pairs of the form (public, private) which key pair(s) control an AID (Autonomic IDentifier) which is itself a cryptographic pseudonym (cryptonym) derived from its incepting controlling key pair(s). 

The key state for the AID is then maintained by an append-only forward and backward-chained cryptographically verifiable data structure called a KEL (Key Event Log). This enables the secure rotation of signing keys that have been weakened by exposure or have been compromised. The forward chaining uses a post-quantum resistant pre-rotation recovery mechanism that enables recovery of control in spite of compromise of the current signing keys.

### Multi-Sig AID
When multiple keys are used to control a given AID, we call it a software-thresholded multi-sig AID or multi-sig AID for short because each key event in its KEL must be signed by a threshold-satisficing number of its controlling keys. This provides a threshold structure to protect the key state.  

The primary use case for a multi-sig AID is for each private key in the set of controlling key pairs to be stored on a different device, all in the locus of control of its controller. This way, an attacker must simultaneously compromise a threshold satisficing number of those devices in order to wrest control of the AID from its controller. This exponentially multiplies the degree of difficulty of an attack. Importantly, in this primary use case, there is one and only one controlling entity, so that we may assume tight coordination in the use of the set of devices.

### Group Multi-Sig AID
Notably, because the security of a multi-sig AID is based on maintaining each member of the set of private keys on a different it naturally can be extended to what we call a Group Multi-Sig AID where each controlling key pair is contributed by a different controller (controlling entity). Each device that stores a given private key from the set of controlling key pairs resides in the locus of control of a different distinct controlling entity (controller). The set of controllers we call a group which gives rise to the term group multi-sig AID.

Each member of the group needs to manage the single-sig key pair it contributes to the group at any time in the evolution of the key state of the group multi-sig AID. For this purpose, each participating member controller of the group creates a single-sig AID with its own KEL. Each single-sig AID may append events to its KEL as desired and may rotate its key when necessary. When the group multi-sig AID needs to append an event to its KEL, it uses the current contributed key state from each of its member AIDs. When it needs to rotate, each of its members must first rotate their single-sig AID key states and then contribute the new single-sig key state to the new group multi-sig key state.

This requires coordination between the members of the group multi-sig.

The KERI stack (keri.py) includes a higher-layer protocol for eliciting the signing of group multi-sig interaction events, the rotation of single-sig member KELs, and the subsequent contribution and signing of group multi-sig rotation events.  This higher-layer protocol provides a bare-bones coordination mechanism between members of the group.  

It takes advantage of the fact the signatures on key events for KELs may be collected asynchronously. A given proposed multi-sig key event only needs to be signed by one key in the set of controlling keys to be saved in escrow. Later signatures may be collected until such time as the threshold is reached upon which the event is accepted into its KEL. 

The only bound on this asynchronicity is the timeout of a given escrow entry. However, this timeout can be made large enough to accommodate the tight coordination among the group members.

Thus, for group multi-sig AIDs, we can leverage the security properties of the multi-sig threshold to protect both the key state of the group multi-sig AID and the group membership. Only a controller who controls the associated private key from the current key state of the group multi-sig AID can prove membership in the group.

By default, KERI follows the practice of least disclosure. In the group multi-sig case, this means that the KEL does not publicize the single-sig member AIDs that contribute to a group multi-sig AID. In other words, the KEL does not distinguish between a multi-device single controller multi-sig or a multi-device multiple controller multi-sig (group).  

The only place the member AIDs may be explicitly disclosed is out-of-band to the group multi-sig KEL itself. Each member of the group maintains a list of the contributing member AIDs and then may use these in the higher layer group multi-sig coordination protocol described above. But no third party not in the group can discover this from the group multi-sig AID's key events themselves. This must be discovered some other way, such as a member of the group publicizing it or by searching all KELs and correlating the public keys of single-sig KELs to the group multi-sig public keys.

Obviously, in applications where group membership benefits from publicity, such as employees of an enterprise interacting with employees of other enterprises, then a verifiable way of publicizing the member AIDs of a given group multi-sig AID is for the group multi-sig AID to issue an ACDC that discloses its member AIDs at the time of its issuance. Later changes in group membership may be verifiably disclosed by revoking and re-issuing a new ACDC with the new membership list.



### Delegation, Witnesses, and Watchers

Besides multi-sig, two other threshold mechanisms may be employed to protect key states. Each exponentially multiplies the degree of difficulty of attack. These are the multi-factor authentication of events to be accepted by the witnesses of a witnessed KEL and delegated AIDs. When a given AID employs all three threshold structures, multi-sig, multi-factor witness event acceptance, and AID delegation, an attacker must simultaneously compromise a majority of three different thresholds. This provides an unprecedented level of protection.

Validators of controller key states are protected by a watcher network designed to incentivize validators to share watched key states to detect duplicity. With KERI, both controller key compromise and malicious controller behavior exhibit provable key state duplicity. In other words, the KERI key state is duplicity evident. The watcher network protects validators from both. Upon detection, it does not matter initially to the validator why there is duplicity, but only that duplicity has been detected so that issuances from duplicitous key states are not to be trusted. This protects the validators. A compromised controller may then use KERI recovery mechanisms that, when satisfactory to the validators, may be used to reconcile the evident duplicity while preventing malicious controllers from benefiting from their evident duplicity.

### KRAM

Besides Key Event Messages, KERI supports other message types that are used to manage supporting infrastructure. These include query and reply messages and most relevant to this discussion exchange `exn` message. Exchange message provide an interoperable generic message format for peer-to-peer exchange of information in support of other activities. 

KERI is a sign-everything approach to authentication. This maximally leverages the fundamental security properties of KERI. KERI maintains the current non-repudiable digital signing key state. This means that when a given message is signed by the private key(s) and verified with the associated public key(s) from the current key state of the AID then that verifier can reasonable attribute that message as being sourced by the controller of that AID. We call this secure attribution to the source AID.

This means that any over-the-wire message can be securely attributed, i.e. authenticated as originating at the time of signing from the controller of the private keys for that AID.  This is an ephemeral type of authentication. Should the key state change between the time of origination and the time of verification, then the verifier can no longer assume the signature as a secure form of attribution because one of the primary reasons for a given controller to change its key state is to recover from key compromise. This means that by default, any signatures produced with the stale key states are not trustworthy.

A message sent over the wire that includes the source AID with an attached signature is a type of verifiable issuance from the controller who signed it. But it is, at best, an ephemeral issuance given the dynamic key state of the AID.

That ephemerality is perfectly suitable for protecting over-the-wire messages that must be authenticated at the time of reception to protect from impersonation fraud. The caveat is that the message signature does not use a stale key state. Including the source AID in the message header in the clear enables a load balancer or other filter or firewire to drop messages that are not verifiably signed. Verification of signatures only requires the public key of the AID. So, there is no security weakness induced by this leading-edge verification as a result of sharing secrets.

An additional problem when using ephemerally signed messages on asynchronous channels is that signed messages may be captured by adversaries and replayed out of order. If the only protection is that the key state must not have changed since the message originated, then the adversary could intercept the messages en-route and then reorder the reception of these securely attributed messages at the destination. 

In some cases, depending on what type of transaction the messages are used for, this out-of-order replay may result in a harmful exploit. These are called replay attacks. So, in addition to signing the over-the-wire messages, we need a replay attack protection mechanism.  

The main advantage of using asynchronous messaging protocols is performance at scale. So, we don't want to hamper its scalability with a non-scalable replay attack protection mechanism. The three main types of replay attack protections are:

 1. Authenticated sessions on a dedicated synchronous channel. 
 2. Interactive challenge-response authentication 
 3. Timeliness ordered authentication. 

Obviously, 1. is precluded for asynchronous networks, and 2. is fundamentally less scalable than 3. since it requires a minimum of two times the message traffic for each message to be authenticated. This generally leaves 3. as the only viable replay attack protection mechanism for asynchronous messaging.

KERI Key Event messages and their associated KELs have built-in ordering mechanisms that are tied to the current key state, so they are self-protecting from replay attacks.

However, the generic exchange, query, reply, prod, and bare messages do not have any built-in replay attack protection. For this purpose, the KRAM (KERI Request Authentication Mechanism) was designed. 

#### Simple KRAM

Unfortunately, due to the exigency of limited resources in implementing support for the vLEI, only the most expedient version of KRAM, called simple KRAM, was implemented. This has now become problematic and is largely the root cause of the associated issues and discussions.

Concisely, simple KRAM works by including in the over-the-wire message a timestamp referenced to the network time clock of the recipient. Network time means time synchronized to well-known ubiquitous network time servers. 

Because the timestamp is referenced to the time as seen by the recipient, the originator of the message can't lie about time. The recipient has a moving time window whose size is some small multiple of the average network latency and drift of network time and surrounds the current time as seen by the recipient. If the timestamp of the received message lies outside this window, it is dropped. This window limits the time during which a replay attack can be mounted. If the average time between subsequent messages sourced by any given originator is larger than this window, then no replay attack is possible. Therefore the protective efficacy of simple KRAM is better the smaller the window.

For example, let `t` represent the current time seen by the recipient, let `d` represent clock drift and skew, let `l` represent average network latency, and let `m` represent some interger multiple. Then the simple KRAM window would be the interval `[t-d-m*l, t+d]`.  The timestamp of the received message must lie within that window. Typical values could be `d = .01` seconds, `l=1` second, and `m=3`.

However, simple KRAM may be problematic for multi-sig AIDs for several reasons, which will be discussed later. One has to be careful when building exchange-message-based protocols using multi-sig to avoid violating a very small time window. This is not a problem for full KRAM.


#### Full KRAM

Full KRAM employs a strictly monotonically ordered timeliness cache to protect from replay attacks. It includes protection from retrograde attacks on the recipient's clock. The timeliness cache can be granularized to have different caches per transaction type so that interleaved messages from multiple transactions do not interfere with each other. Moreover, when the messages include their own non-timeliness, i.e., numeric ordinal, then both ordinals, timeliness, and numeric, can be employed in concert to account for asynchronous networks that deliver messages out-of-order but that are not replay attacks. 

Simply speaking, a timeliness cache stores the timestamp of the last message received from any source AID. Whenever a new message is received, its timestamp must be later than the timestamp of the stored message. If so, the later message is accepted and the cached timestamp for that AID is updated. This timeliness cache may  employ a stale cache release where it includes a lagging time window relative to the current network time seen by the recipient.  When an AID timestamp cache is empty, new entries in the cache must be later than the oldest end of the time window. Any cached timestamps older than this lagging time window can, therefore, be safely deleted. A replay attack of an already received message will not be later than a cached timestamp, and if the latest received message is already older than the lagging time window, then any received messages must be sooner than that, so it is not actually a replay. 

To protect against retrograde attacks on the receiver's time server or internal clock, the receiver saves a single timestamp of the latest time it sees. If the network time is even older than the latest saved time, the receiver can refuse to accept messages until the clock catches up.

The timeliness cache can be made more granular to save timestamps not merely by source AID but by some vector of attributes such as all of or a subset of the following: (source AID, message type, transaction type, transaction ID).  

The timeliness cache can be stored in memory only or persistently on disk. The latter consumes more resources and may be less performant but does not induce retries in the event of a reboot of the receiver's process.

Importantly, the lagging time window for releasing cached timestamps can be quite long in duration. It is really only bounded by memory or disk resources for the cached time stamps.  This means it could be days or weeks long. 

This can largely relieve any latency or ephemerality problems with group multi-sig AID signed over-the-wire messages. Especially if the cache is granular, i.e. per the vector (AID, transaction type or transaction ID). This way, if a given exchange message is signed with a given timestamp by one member of the group and then not fully signed by a threshold number of other members of the group until days later (assuming the lagging window is days in duration), it could still be accepted by Full Kram. 

For example, let `t` represent the current time seen by the recipient, let `d` represent clock drift and skew, let `l` represent the lag duration. Then the full KRAM window would be the interval `[t-d-l, t+d]`.  The timestamp of the received message must lie within that window. [NTP Clock Skew](https://www.sciencedirect.com/topics/computer-science/network-time-protocol#:~:text=However%2C%20path%20delay%20asymmetry%20is,its%20skew%20below%200.01%20ppm.)  Typical values for for clock skew are around 10 milliseconds with worst case usually 100 milliseconds. So a value for d would be some integer multiple  of 10 milliseconds, say `d = .1` seconds that is 100 milliseconds.  We could let `l`  be on the order of days or weeks, say `l=2` weeks.

If we have two networks nodes with really bad network time clock skew we could set `d` to be an integer multiple of the worst case clock skew of 100 milliseconds, like say 200 or 300 milliseconds.

The clock's resolution determines how many messages can be received in order in any timeframe before the sender must block to avoid running past the leading edge of the receiver's time window. The timestamp in KERI exchange messages uses a microsecond resolution, which support up to 1 million messages with unique monotonically increasing time stamps per second per timeliness cache entry. This is discussed in more detail below in the comment labeled time resolution. 

When a message is received, the first filter is to test if its timestamp lies within the interval. If not the message is dropped. The next filter is to check if a cache entry already exists for the AID and transaction ID and message type. If so then check to see if the timestamp of the received message is earlier than that of the cached message, if so drop the received message. If the timestamp is the same or and the message is the same then verify attached signatures. If verified, then accept (idempotently for the message) and process attachments. If the timestamp is later, then verify the attached signatures signatures and accept if verified and then update the cache, replacing the existing message entry with this new one. One a periodic basis scan the cache and prune any entries the lie outside the interval  `[t-d-l, t+d]`.

The above approach could work with group multi-sig in the following manner: A given message sourced by a multi-sig AID must be verifiably signed by at least one member of the group. If the message is the first received message, it must lie within the receiver's lagging time window with respect to its current time. A new cache is created with the time stamp, message type, and transaction ID. The message itself and its attached signatures are added to a partially signed escrow.

If the message is not the first message received for a given timestamp, message type, and transaction ID, but is the same as the cached one, then the message is accepted by full KRAM, and any attached and verified signatures are added to the partially signed escrow for that message. The same message but with different but verified member signatures is not considered a replay attack. The signatures are added to the database in an idempotent manner i.e., if a signature already exists, nothing changes. As long as the lagging time window nor the escrow timeout expires while collecting a threshold satisficing number of signatures, the message will be accepted and moved out of escrow.

In many cases, merely implementing full KRAM in this way for group multi-sig non-key-event messages, namely, exchange, query, reply, prod, and bare, will solve the current problems.

#### Complications of multi-sig AID sourced messages and KRAM

The primary complication of multi-sig sourced AID messages with KRAM is deciding what policy a given KRAM instance applies to accepting a message sourced from a multi-sig AID. One policy would be to accept the message if any one of the members signed it and then escrow the message in order collect signatures from copies of the message signed by other members. The second would be to require that the received message include a threshold satisficing number of signatures which in turns requires a pre-protocol to collect signatures before sending. The pre-protocol obviates the need for an escrow to collect signatures. 

Both policies depend on a lagging time window that is long enough to collect signatures from the members. The timestamp in the message must lie within the lagging window, but this lagging window may be set to be large enough that loose coordination between the members is practical. 

For simple KRAM, the only practical policy is to accept the message if it is signed by any one of the group members and its timestamp lies within the much tighter time window.  But then the message is not protected by the threshold of the group multi-sig.  Employing an escrow to collect signatures after acceptance does not help because all the to-be escrowed signatures must still arrive within the narrow time window, which means the coordination of the group members must be tight enough to fit in this window. This is the root problem of using simple KRAM with group multi-sig AIDs. 

#### Two-Level Simple KRAM
A viable modification to the single-sig acceptance policy with simple KRAM is to use a hierarchical or two-level approach.  In the two-level approach the first level employs simple KRAM to authenticate the over-the-wire exchange message as a wrapper using single-sig from any member and the second level authenticates the embedded payload using the full multi-sig threshold. The second level does not employ KRAM. To clarify, the authentication of the over-the-wire conveyance using the exchange message as a wrapper is different from the authentication of an embedded payload so wrapped.

To further elaborate, in the two-level method, the over-the-wire conveyance is authorized, given a single signature from any group member, and must satisfy the simple KRAM replay attack protection window. Whereas the embedded payload, which is not subject to KRAM replay attack protection, is separately signed from its exchange message wrapper and is only authenticated when the signatures on the embed satisfy the multi-sig threshold. This requires each member to generate and sign an exchange wrapper with a timestamp separately, which is subject to KRAM, and also sign the embedded payload, which is not subject to KRAM. Once past simple KRAM the embedded payload goes into an escrow to collect signatures asynchronously from the group members. 

The embedded payload may be a tunneled exchange message that just leverages the processing of exchange messages. The tunnel is meant to cross through simple KRAM. 

A limitation of this approach is that the escrow timeout for exchange messages must be long enough to support loose coordination when collecting signatures from the group members. If this is days or weeks, then the escrow must also keep partially signed messages in escrow for days or weeks.

This is the current supported approach. This allows escrow of the embedded payloads to collect signatures asynchronously until the threshold is satisfied. It has the drawback that each member must sign twice, the wrapper and the embedded payload. Whereas the full KRAM solution for group multi-sig messages described above only requires a signature on the wrapper.

#### Two-Level Simple KRAM with pre-protocol
A variant of the two-level method with simple KRAM is to use a pre-protocol to collect the signatures on the embedded payload so that a designated member can send a single-sig signed wrapped with multi-sig signed payload all at once. This is the motivation for the above-referenced HAMI issue. This approach avoids having to deal with the escrow timeout while collecting signatures. The HAMI protocol uses a different mechanism for collecting signatures asynchronously given very loose coordination time frames.

#### Summary
Any of the policies works with full KRAM, be it single-level with escrow or two-level policy with single attached signatures and embedded payload signature or with a threshold and pre-protocol to collect signatures. Moreover, with granular full KRAM each transaction type could employ a different acceptance policy.


### KEL Sealed (Anchored) Issuances

In addition to ephemeral authentication by signing over-the-wire messages with the current key state, KERI supports non-ephemeral authentication by binding issuances to the key state at the time of issuance via seals (anchors) in the KEL of the AID. 

Seals benefit from the built-in replay attack protection mechanisms of KELs. They also enable an issuance to be verifiable in spite of later changes to the key state. For authentication, instead of attaching a signature to the message, one attaches a reference to the sealing (anchoring) event. The validator then looks up the seal and verifies it against the message. The seal is a strong cryptographic hash of the content of the message. The cryptographic strength of signing a seal of the message is equivalent to signing the message itself. Because the seal is included in the key event and the key event itself is signed with the current key state, the issuance is verifiably strongly cryptographically bound to the key state at the time of its issuance. 

Moreover, KERI key recovery mechanisms preserve the verifiability of previous seals in events not affected by a given recovery action. They remain verifiable.

The disadvantage of anchoring messages is that it tends to bloat KELs. This can be ameliorated by using delegated AIDs. Also, load balancers, firewalls, and other perimeter protection mechanisms now have to walk KELs to look up and find anchors before making an accept or deny decision on a given over-the-wire message. 

Nevertheless, indirect signing via anchoring may be more suitable than direct signing for many applications. Or a combination of both, where anchored issuances are conveyed with ephemerally signed wrappers. This enables more performant accept/drop decisions by load balancers in response to DDoS attacks at the perimeter but preserves the ultimate multi-sig thresholded protection of the embedded payload. The latter accept/drop decision is made inside the load-balanced perimeter.

A caveat: when collecting signatures on group multi-sig key events in order to anchor something, the current code uses a partially signed escrow to collect those signatures. The default escrow timeout, i.e., the time an event may remain in escrow before being pruned, is currently on the order of hours, not days or weeks.  So if the group coordination time frame needs to be looser than hours to complete a group multi-sig collection, then the partially signed escrow timeout needs to be extended. In addition, the escrow timeout could be made granular by group AID. One can then have a different escrow timeout per each group multi-sig AID so that groups with looser coordination timeframes get correspondingly longer escrow timeouts.


### IPEX

The Issuance and Presentation EXchange protocol is designed to support negotiation between two parties over the contents of an ACDC. ACDCs support something called graduated disclosure, which supports negotiation of the extent and terms of issuance and or disclosure prior to making the agreed upon issuance or disclosure. This enables, for example, privacy-protecting partial disclosure and/or contractually protected issuance and/or disclosure. Because its a back-and-forth negotiation that may be originated by either party, the peer-to-peer exchange message is used for an IPEX negotiation. 

The exchange message header includes several fields that facilitate such peer-to-peer exchanges. Below is a JSON serialization of an example exchange message:


```json
{
  "v": "KERICAAJSONAACd.",
  "t": "exn",
  "d": "EF3Dd96ATbbMIZgUBBwuFAWx3_8s5XSt_0jeyCRXq_bM",
  "i": "EMF3Dd96ATbbMIZgUBBwuFAWx3_8s5XSt_0jeyCRXq_b",
  "ri": "ECRXq_bMF3Dd96ATbbMIZgUBBwuFAWx3_8s5XSt_0jey",
  "x": "EF3Dd96ATbbMIZgUBBwuFAWx3_8s5XSt_0jeyCRXq_bM",
  "p": "EDd96ATbbMIZgUBBwuFAWx3_8s5XSt_0jeyCRXq_bMF3",
  "dt": "2021-11-12T19:11:19.342132+00:00",
  "r": "/echo/back",
  "q": {},
  "a": 
  {
    "msg": "test echo"
  }
}
```

The `a` field is the embedded payload. The other fields may be considered the header to the exchange message. Notable is the `r` field, which contains the route. This used to differentiate the purpose of the exchange message. Combined with a modifier in the `q` field block, this allows an arbitrary level of differentiation.

The IPEX protocol pre-defines some routes for generic exchanges: `offer`, `apply`, `grant`, `agree`, `admit`, and `spurn`. The spurn is a type of negative acknowledgement that rejects a prior `offer`, `apply`, or `agree`, thereby forcing the negotiation to restart.  

The IPEX protocol entails a potentially unbounded timeframe for completing any given negotiation.  

It uses signed exchange messages. Simple KRAM applied to signed IPEX exchange messages in a negotiation may be problematic in two significant ways. Firstly, when the exchange message is only signed, verifiable proof of the commitment by either party to a given stage in the negotiation is ephemeral. Sometime later, when the key state changes, that commitment is no longer verifiable by a third party. Secondly, if either party is group multi-sig and simple KRAM is in force, then it's impractical to collect the group multi-sig on the exchange messages in time to satisfy the simple KRAM window.

In the first case, one solution is to anchor the final issued or presented ACDC that results from the negotiation in the KEL of the respective issuer/presenter. This ACDC then survives key state changes to the Issuer/Presenter's AID. Once the final ACDC is anchored, the negotiation evolution becomes immaterial. Some corner cases that do not exist in an issuance have to be accounted for in a presentation. 

In the second case, one solution is to use two-level authentication, with singly signed exchange messages subject to KRAM and multiply signed embedded payloads that satisfy the multi-sig threshold but are not subject to KRAM. In spite of key state changes, one would still anchor the final ACDC to satisfy third-party verifiability. The limitation on coordination timeframes is the duration of the escrow collecting signatures on the embedded payloads.


## Problems and Solutions

### Summary
In this section we review the problems and solutions and suggest paths forward for group multi-sig. Single-sig does not pose a problem in general.

The existing code supports simple KRAM with a two-level acceptance policy where the over-the-wire exchange message is singly signed by any member of the group and acts as a wrapper that must satisfy simple KRAM. The embedded payload is signed separately and is escrowed to collect signatures to reach the threshold and does not need to satify KRAM.

To eliminate the escrow, a pre-protocol that collects signatures on the embedded payload could be developed. This is the HAMI proposal. Then the wrapper exchange message is not generated until a threshold satisficing number of signatures on the payload have already been collected.

Both of these are essentially stop-gaps that arise from the constraints of simple KRAM. Given that simple KRAM was developed as a stop-gap itself, we are piling stop-gaps on top of stop-gaps, and at some point, the level of effort may no longer be expedient.

The alternative is to pay the price of implementing full KRAM. The existing code would still work with full KRAM with, at most, minor modifications.  The advantage of full KRAM is that it unlocks more policy options and obviates the need for stop-gap solutions.

With full KRAM one can still build a variant that uses a pre-protocol for collecting signatures that avoids the escrow. And one can avoid using a two-level solution where both the wrapper and its payload are signed separately. This may still be desirable in some cases. The advantage of full KRAM with a granular timeliness cache is that the acceptance policy can be transaction type-specific.

Therefore, we recommend that Full KRAM be given a high priority for implementation and hope to find someone in the community interested in implementing it.

### Other Options

In a recent discussion, Daniel Hardman pointed out a concern with group multi-sig IPEX negotiations that place a heavy burden on group participants to sign multiple messages during the course of one negotiation transaction. The burden on signers and validators may be different, thereby complicating the trade-offs between policies and approaches. The trade-offs are not simply a matter of finding the optimum solution with respect to bandwidth, computation, and storage but must also consider the human coordination and signing intervention burdens. 

Validation (which includes business logic and cryptographic verification) may be largely automatable, whereas signing requires human intervention. Therefore, increasing the computing burden on the validator may make sense in some applications to reduce the human signing intervention burden on the signer.

Described below are some alternative proposed approaches up for discussion. Some include trade-offs that may significantly increase the validator burden but enable looser coordination among group members and/or reduce group member signing intervention burden.

#### Agent Delegated Negotiation

Avoid multi-sig on each stage of IPEX by using a delegated agent and then only require the group to sign in order to anchor the finalized ACDC. 
Each side of the negotiation designates an Agent. Use OOB comms amongst group members and with agents to guide agent actions in the IPEX negotiation. Agent-issued `agree` for one side and `grant` for the other are valid only if they include a verifiable reference to a corresponding anchored ACDC. This protects the group members from malicious agents. This means all negotiation transaction exchange messages are single-sig signed by the agent. 

Group members only sign two things. An anchoring key event message for the ACDC authorizing their respective agent and the anchoring key event message for the issued "agreeing" ACDC for one side or the issued "granting" ACDC for the other side.

#### Anchoring instead of Signing

Recall that in the two-level approach, there are two signatures for each message. One that signs the wrapper, which is single-sig, and one that signs the payload, which is multi-sig. Notice that although there are two signatures, each signer needs only one signing intervention. Both signatures are authorized at the same time. The two-level approach could be a hybrid. Instead of creating an ephemeral commitment to the payload with a signature, the payload commitment could be to anchor the payload in the group KEL. This still only requires one signing intervention but does add the latency of a cycle of collecting key event signatures. This leverages the existing code for entering new key events into a group multi-sig KEL.  

Therefore, each negotiating message need only be sourced by one and only one group member. The other group members merely endorse it by signing the anchoring key event message. This allows one group member to be the lead negotiator, and the others don't communicate in the exchange, just concur with the lead.  This approach can also use a delegated agent instead of the lead group member.

The payload of the exchange message includes a reference to the anchored key event message that anchors the payload. Therefore, the commitment in the payload could be to an ACDC or an ACDC with a TEL. In the latter case the included reference is for the TEL, which holds a reference to the KEL anchoring event.

This has the advantage that each stage of the negotiation is anchored (not ephemeral). This may be beneficial in some types of high stakes transactions. Because each stage is anchored, there is no consummating anchoring signature collection at the end. This has the disadvantage of  the added latency of group multi-sig interactions to sign event at each stage. But the interventions by signers is the same, whether, it be to sign the payload and wrapper or to signe the anchoring event. But because the IPEX messages are all single-sig, either by the lead or by the agent, there is no incompatibility with Simple KRAM.  

As caveated in the anchoring section above, the partial signature escrow may need to be granularized per group multi-aid so that partially signed key events for groups remain in escrow long enough to collect signatures for loose time coordinating groups, i.e. days or weeks, not hours.

#### Joint Issued ACDCs

For loosely coordinating groups, i.e. groups where the time to collect signatures may take days or weeks one option is to not use the group multi-sig kel for IPEX issuances.

In this case the group multi-sig KEL is only used to establish and protect the group membership and its key state. The only group multi-sig issuances related to IPEX would be to issue an ADCD declaring the AIDs of the group members, which is revoked and replaced anytime group membership changes. When using agents to negotate another issuance would be an agent authorization to conduct the negotiations.  

One advantage of this approach is that group members could more conveniently participate in IPEX negotiations becasue the result of the negotiation is not a group multi-sig issuance but a joint issuance by the group members. 

Joint issuance is a new concept. Indeed there were some discussions early on in the evolution of the ACDC specification about so-called joint issuance or multiply issued credentials.

One way to implement joint issuance is that a given ACDC that differs only is its top level fields related to the Issuer and registry, is singly issued by each member of a set of AIDS. This requires no changes to the existing ACDC spec for ACDC structure. 

An attribute block is included in the attribute section of the ACDC that defines the joint issuance from the group as indicated by a group multi-sig AID and the sequence number of the key state that is determinative for the joint issuance. In addition each joint issued copy includes an edge that points to the group multi-sig issued ACDC that declares the group member AIDs at that sequence number. The AID member list declaration ACDC is really just a more normative discovery mechanism that avoids an ambiguity attack. The same public key could be used by its controller to control more than one AID. So merely correlating public keys with KELs does not uniquely determine which AID is a group member. The declaration does this. This could also be accomplished with  signed OOBIs of the member AIDs.

The validation logic for the validator is that it must receive a verifiable reference to a threshold satisficing number of singly issued ACDCs whose issuee, attribute, edge, schema, and rule sections are the same and only differ at the top level (issuer is different). The validator then looks up and validates the references. The threshold is defined by the designated key state of the group multi-sig AID in the joint issuance attribute block of the jointly issued ACDC. The backward reference is not a problem since the multi-sig KEL is not anchoring the joint issued ACDC. The joint issued ACDC is making a backward commitment to the key state that is determinative for the joint issuance in terms of threshold, public keys (and hence verifiable group membership).  

This can be further automated by requiring that at presentation a given presenter present a self-issued joint issuance summary ACDC whose edges point back to the set of singly issued ACDC copies.

To recapitulate, the group multi-sig AID is only responsible for managing group membership and thresholds which are then normative for joint issued ACDCs as well as issuing declarations of group membership or delgations when applicable. But joint ssuances by the group are a set of single-sig issuances anchored separately in each members KELs/TELs and validated but looking up and validating a threshold satisficing subset of those jointly issued copies.

This means that each member of the group may cooperate very loosely except on group formation, group membership update publication and/or group delegations. Which happens rarely. Joint issuance cooperation is single-sig by each particpant with extremely loose coordination possible.

For example, when desirable, joint issuance would allow each stage of a negotiation to be anchored singly in each single-sig group member KEL. The other party would have more work to validate, having to look up anchors in multiple KELs instead of one. But that is an automatable effort. The final issued ACDC would be a joint issued ACDC as defined above. A joint issuance summary ACDC could also be issued by the negotiator at each stage of the negotiation to help automate the validation of the jointly issued ACDCs.

## Implementation

### Full Kram

We suggest defining a vector of attributes as the key for each cache entry will work. In keripy lmdb we use a tuple to derive the key for an given database entry. So as long as the tuples are well structured you can mix and match in the same database. Some entries have more elements in thier tuple than others. This just changes the b-tree location. The only place this complicates things is the the pruning of the cache has to walk the tree to find all the entries. Which again is not hard to do. We do it in other places.

So for those cache entries that need custom window sizes you define a tuple that includes sufficient detail to differentiate them. Like source AID, message type, and for exns, then also transaction type. the final element in either case (exn or not) is the window duration. or the window duration goes in the database entry not its key space. In a b tree, essentially anything in the key is also retreivable as a value. Onc can retrieve whole branches as a collection of values.

We need to be careful to avoid transaction-specific windows. These have the potential for an attack.  If some transactions of a given type need longer or shorter KRAM cache windows with respect to other transactions of the same type, then we want to add a modifier to transaction types, that is, the window type.  This way, any transaction of a given window class gets the same window size.

So we would have two tables. A lagging window size table. Indexed hierarchically as follows.
For the non transactioned message types (qry, rpy, pro, bar) we have:
KEY = MessageType.WindowType and VALUE = window size
For the transactioned message type, (exn) we have:
KEY = MessageType.TransactionType.WindowType  and VALUE = window size

The corresponding replay caches have the following structure:
For non-transactioned message types

KEY = MessageType.WindowType.MessageID and VALUE=Timestamp

For transactioned message types

KEY = MessageType.TransactionType.WindowType.TRansactionID.MessageID and VALUE = timestamp 

The TransactionID and MessageID are SAIDs (ie hashes) Each message has a SAID, `d` field  and each exchange message has a prior message SAID, `p` field.   Which means the SAID of the first message in a multi-message exchange transaction can be used as the TransactionID.  

Putting a salty nonce in the modifier `q` block of the first exchange message of a transaction makes the transaction ID universally unique even when all the other fields are the same. Why not then use universally unique transaction IDs for replay attack protection?  Unfortunately, we still need a timestamp to know when we can prune any given transaction. Otherwise, we must store all transactions forever in order to prevent replay attacks. The timestamp orders transactions monotonically so that we create the property of "stale" transactions that can both be pruned and not replayed.

In KERI v2, we have a `xip` transaction inception message that normatively defines the first message in a given transaction. The exchange `exn` message then populates its `x` field with this transaction ID. But in v1 we just have to book keep which exchange message was first and verify the other messages by walking the prior hash links back to the first.

The reason we want window durations to be classified is that when durations are per transaction, not a class of transactions, then when a given transaction is not in the cache, we don't know what default window to apply to decide if we can create a new cache value.  If we apply a default that is longer than the one the transaction itself uses, then when we prune it, we have a gap where the replay is possible.   

So we want to unambiguously classify every message as belonging to a window duration class.  If we can assume that all messages of a given type have the same duration, then the message type is synonymous with the window type. Likewise, if all transactions of a given type have the same window duration, then the transaction type is synonymous with the window type. 

However when they are not synonymous, then we need a way to explicitly unambiguously associate a given message with a window size from information in the packet itself.  One way to do this would be to use the route, `r` field, or a field in the route modifier `q` block when we can't simply use either the message type or the combination of message type and transaction id to determine the window size.

Typically we would use the route `r` field for the transaction type. So we would either extend the route path to have a window type differentiator or better use a field in the route modifier `q` block to provide that differentiator.



## Time Resolution and Throughput

In a timeliness cache used for replay attack protection, their is a limit on throughput that is a function of the clock resolution. For KERI exchange messages the clock resolution of the timestamp is is microseconds.  Most modern operating systems support clocks with resolutions in nanoseconds. But microseconds should be good for the forseable future. When they no longer are then we can version KERI to support higher resolution timestamps.

### Limitations

Given a microsecond clock there are 1M time slots available per second for monotonically increasing timestamps in a given cache. Each cache entry gets its own 1M per second set of slots. So its inherently parallelizable. 

Lets say worst case we only have 1 cache entry per source AID for all exchange messages from that source AID. This means that in any given second that Source AID can send at most 1M messages to a given Dest AID.  When it runs out of slots it stops sending and waits for the clock to tick up, and then a new set of slots are created.  If we have a cache entry per transaction, then each transaction gets 1M time slots per second. So highly parallelizable.

Let's be more specific. Recall that the receiver's KRAM acceptance window is `[t-d-l, t+d]`  where `d` is some multiple of the clock skew, and `l` is the lagging window which could on the order of days or weeks. 

So the worst case scenario is that the sender sends enough messages fast enough to fill all the time slots and runs ahead of the leading edge of the window at `t+d`.  This is only going to happen on very high speed networks.  Given there are 1M messages per second and the minimum size of a signed exchange message is about 300 bytes then we have a throughput of 300 MegaBytes per second. Or 2.4 Gigabits per second. So, a high-speed network.

 Let's suppose the worst case, which is that on such a high-speed network, there is zero network latency (every millisecond of network latency effectively provides an extra 1000 time slots because the receiver's clock has incremented by a millisecond after the source message was timestamped and before it was received. On such a high-speed network, a reasonable worst-case clock skew is 10 ms, so we can set `d` to 100 ms.  

Given the 10 ms worst-case clock skew, the senders' clock could be 10 ms ahead of the receivers' clock. The leading edge of the receiver's window is at `t+d`  = `t + 100 ms`.  This means that there are 90 ms worth of microsecond time slots ahead of the sender before it runs ahead of the receiver's leading edge. This means the sender could send as many as 90,000 messages in that 90 ms to the receiver before the sender ran ahead of the leading edge and would have to block waiting for network time to tick up.  But this is only true if network latency is truly zero. Recall that each ms that a message takes to traverse the network, the receiver's clock has added another 1000 time slots. 

So the sender has to batch send at least 90,000 messages at once, each with a monotonically increasing timestamp where it increments each message's timestamp by one microsecond before any messages have timestamps that could possibly run ahead of the leading edge of the receiver's time window and would therefore be dropped by the receiver's full KRAM timeliness clock

### Practical considerations

High-volume transaction infrastructure is usually measured in tens of thousands of transactions per second. So, with reasonable values for the time window clock skew, we have a capacity of 90,000 pipelined transaction messages per second if we do not partition the traffic by transaction but lump all transactions into one timeliness cache per source AID.  This means we are multiplexing and pipelining the transactions into one ordered batch.  

Consider that in a practical, interactive transaction, at each stage of the transaction, the sending party waits for a response from the other party before sending its next message. So the one-way turnaround and traversal time (parsing and verifying SAIDs and signatures) on a stage in an interactive exchange would have to take less than 1 microsecond per one way to consume all the available 1-microsecond slots and, therefore, be blocked at the sender or dropped at the receiver. A typical turnaround time for an exchange message will be in the tens of milliseconds. So, it is highly unlikely that any given transaction would ever run into any clock skew based limitation. We are, therefore only worried about the case where there are 90,000 simultaneous transactions between a given sender and received that are all concurrent and share the same timeliness cache. 

If we reach that limit, the receiver simply makes its timeliness cache more granular. Instead of one cache entry per AID per message type, it can have one cache entry per AID per message type per transaction type or even more granular to one cache entry per message type per transaction type per transaction ID.  Now, it would have 90,000 slots per transaction. It would be entirely surprising that anytime soon, any infrastructure would need to process a single transaction between two parties with 90,000 interactive stages that all happen in 10 ms, i.e., a full round trip since we are not pipelining transactions on a single cache entry. Therefore, each round trip must happen in less than 1 microsecond to fully consume all the available time slots for that cache entry.

Therefore we can safely conclude that microsecond clock resolution on our timeliness caches is more than adequate for the foreseeable future.


### Asynchronous Out-of-order Messages

The above analysis assumes that message from the same sender are delivered in order at the receiver. In an asynchronous network with routers and gateways using the Internet there may be multiple paths through the network, so a given set of messages when conveyed asynchronously may show up out of order due to different latency for different paths. This means that the timeliness cache would drop a message with an earlier timestamp that showed up later than a message with a later timestamp.  

There are several ways to address this problem. 

The first is to recognize that  interactive transactions induce a natural ordering because the two parties take turns. One solution therefore is to use a timeliness cache entry per transaction. That means that a given sender won't send a subsequent message unless:

1. a retry timer has timeout out so it sends a retry of a previously sent message. 
2.  the other party has responded 
3. a transaction stage timeout timer expires so the sender gives up on the transactions and sends a Nack. 
 
So in the case of interactive transactions with one transaction per cache entry, 

1.  is not a problem because if it's truly a retry, it won't matter to the transaction if it's the first try or some later retry that is delivered first.  Since there are retries, then it won't matter if a message is lost because it will send a retry.
2. It is not a problem because if the other party has responded, then the first party knows that its original try was already accepted by KRAM before it sends another message. 
3. in this case, the transaction timeout needs to be long enough to account for the worst-case path latency difference, and the sender, once it decides to cancel a transaction, can't reverse that decision. It must restart the transaction. This way even if the first message is not lost and the nack shows up after the first message, and the other party responds, the response is ignored, and then when the other party gets the nack it also cancels the transaction.

Therefore out-of-order messages are only a problem when we are pipelining messages from multiple transactions into the same timeliness cache entry to maximize throughput. In this case out-of-order messages between transactions could interfere with each other. But then again this is only a problem if the transactions have no reliable retry mechanism. If the transaction itself is unreliable, then one should be using a reliable synchronous channel where one is guaranteed that messages are delivered in order and no messages are lost.

But for the use case where the transactions include retry capability, i.e. the transaction is reliable, then even if multiple transactions are pipelined interleaved with messages from other transactions it won't matter if some messages are delivered out of order as long as the rate of out-of-order messages is small relative to the number of interleaved transactions because the retries will repair any dropped messages.

To conclude, when one is forced to use asynchronous transport then it is recommened that one use non-interleaved cache entries, i.e. one cache entry per transaction. When one is forced to use interleaved  transactions on the same cache entry then it is recommended that the one use a reliable channel where messages are delivered in-order.  Othewise one has to depend on a reliable transaction with retries to overcome any out-of-order messages on an unreliable channel when using and interleaved cache entry.











