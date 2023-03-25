---
tags: TSP, KERI
email: sam@samuelsmith.org
version: 1.00
---

# Privacy Given Strongest Authenticy and Confidentiality (part 2)

[HackMD](https://hackmd.io/rVStgMc_T0GFyXV_e2e_aQ)


## Unbounded-Term AIDs instead of Long-Term Public Keys

[KERI](https://keri.one) AIDs (autonomic identifiers) have a novel and unique feature called persistent controllability. With KERI the controlling key pairs of an AID may change via rotation or in other words the control is transferable to a new set of controlling key pairs. This enables the controller to maintain persistent control over the AID in spite of compromise of the controlling key pairs. Specifically, the KERI pre-rotation mechanism uses one-time pre-rotated rotation keys to recover control of the AID by transfering that control to a new un-compromised set of key pairs. Because the pre-rotated keys are not exposed they are much harder to attack. At inception a persistable (transferable) AID makes a forward commitment to the rotation key-pairs via cryptographic digests of the public keys. These keys are never used until and unless there is a rotation. Only the pre-rotated keys will verify against their digests. With each rotation, a forward commitment is made to a new set of one-time only pre-rotated keys thereby always staying one step ahead of an attacker. 

KERI's pre-rotation may be characterized as what many in the academic literature now label a meta-cryptographic system. KERI itself only uses well established cryptographic operations, like digests and digital signatures but uses them within a system that provides useful features like transferable or persistent control over an identifier.

Many public key cryptographic algorithms employ the concept of *long-term* public keys and *short-term* public keys. Often long-term keys have less exposure then the short-term keys so can maintain their cryptographic strength longer. Typically this is to better manage the friction of key exchange where higher friction authentication of long-term keys are used to enable or bootstrap cryptographic operations using short-term or ephemeral keys. 

A long-term public key is often used as the primary identifier for a party in an interaction or exchange. With this usage the long-term public key may be called the address of the party. The problematic limitation of using long-term public keys as party identifiers (addresses) is that the public key is ***only*** long-term not unbounded-term. Eventually any long-term public key must be abandoned due to exposure. NIST recommends that public authentication keys have a [cryptoperiod](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-57pt1r5.pdf) (time before key must be replaced) of no more than 1 to 2 years . Which means that any authentication or reputation associated with a public key as address by a relying party (public key is an identifier for a given party) must also be abandoned. This forces the two parties to start over authenticating or re-registering a new public key as a replacement address or identifier. Moreover there is no inherent way for a relying party to know that some other party has abandoned its public key as identifier. This exposes the two parties to attacks that expoit this uncertainty via the used of stale public-keys and/or attack the process of re-registration. 

This is one of the reasons why we consider key rotation the hard problem of key management. KERI solves the hard problem of key rotation using key pre-rotation in combination with a cryptographically verifiable append only data structure based on a cryptographic root of trust that provide proof of key state for any persistable (transferable) AID. This data structure is called a Key Event Log (KEL). A controller publishes its current key state via its KEL, thereby enabling any relying party to verify the current state and also to detect when a key as been rotated (abandoned). Stale usage of compromised keys by adversaries is thereby detectable. Pre-rotation protects the re-registration process. Identifiers (addresses) therefore maintain their reputation in spite of key rotations. Indeed transactions can be securely bound to key state by anchoring them in the KEL. 

Another significant advantage of using unbounded-term AIDs instead of long-term public keys is post-quantum safe cryptographic agility.  KERI's pre-rotation mechanism is post-quantum safe because cryptographic strength digests such as Blake3, Blake2b, Sha3 are also strong against quantum attack [Hash Collisions](https://cr.yp.to/hash/collisioncost-20090823.pdf). This means that pre-rotated keys are protected from quantum attack exposure and inversion by their digests. This provides sufficent cryptographic agility to be able to rotate to post-quantum safe digital signatures and public keys. This makes the pre-rotated keys safe from surprise quantum attack. In contrast, long-term public keys as identifiers are vulnerable to surprise quantum attack.

With KERI, AIDs as primary identifiers may be valid for an unbounded-term instead of merely long-term. This changes the way meta-cryptographic operations that provide authenticity and confidentiality may be expressed. In the following development, in many cases, where a long-term public key would normally be used, we may use an AID with unbounded-term instead. Given the AID, its latest public key may be looked up from its KEL. This applies to both signing public keys and encrypting public keys. Moreover, with certain caveats, the encrypting public key may be cryptographically derived from the signing public key so only the signing poblic key need be managed in the KEL. 

Using an unbounded-term AID with a KEL solves the problem of the un-detectability of abandoned long-term public keys. If the provided message is using stale keys for a given AID the associated cryptographic operations will fail against the current key state and the relying party can drop the message. Likewise, when the controller updates its key state and uses new keys in a message but the relying party has a stale copy of the KEL, the associated cryptographic operations will also fail, which indicates to the relying party the need to update its copy of the key state. The controller can also send a hint of its current key state along with any message.

One simplification that results from this approach is that many protocols can be simplified to remove repeated setups using ephemeral (short-term) keys whose purpose is to reduce the exposure of the long-term keys.

## End-Verifiabilty and End-Only-Viewability

One of the key principles to building secure systems is to minimize the number of different attack vulnerabilities and for those that remain make them as invulnerable as is practically possible. The approach that KERI takes is to minimize the attack vulnerabilities to key management and nothing else and then to make that key management as strong as possible. The principle is make it so that in any exchange of data (communication):

* If the edges (ends) are secure, then the security of the middle doesn’t matter.

The correlary to that is that:

* It's much easier to protect one’s private keys than to protect all internet infrastructure.

This is shown in the following diagram:

![](https://hackmd.io/_uploads/r1j9gjPlh.png)


For authenticity, the way to remove vulnerabilities in the middle is called end-to-end verifiablity of authenticity or end-verifiability for short. This naturally leads to what we call ***ambient verifiability*** where any-data can be verified any-where, any-time by any-body.

There is a dual property for confidentiality.
For confidentiality, the way to remove vulnerabilities in the middle is called only-at-end viewability for confidentiality or end-only viewability for short. This naturally leads to the concept where any one's confidential data is only viewable one-where, one-time by one-other-body.

Together these properties mean we only have to secure key management at the ends (edges) not everywhere in between. Insisting on these principles greatly simplifies and strengthens any protocols we may choose to build.


To reiterate, a confidentiality overlay is the dual of an authenticity overlay. 
An end-verifiable authenticity overlay is based on one or more asymmetric key pairs for signing, where only the end the private key can sign, and any end with the public key can verify. To clarify, only the key pair controller can sign with the private key, any recipient can verify with the public key.
An end-only-viewable confidentiality overlay is based on one asymmetric key pair for en-de-cryption, where any end with the public key can encrypt and only the end with the private key can decrypt. To clarify, only the key pair controller can decrypt with the private key, any sender can encrypt with the public key. 
Any identifier can have a key state that includes both an asymmetric signing key pair and an asymmetric decryption key pair. 
Either the key pairs can be the same, or the decryption key pair can be derived from the signing key pair
Thus an efficient approach is that only one key state, the signing key state, needs to be maintained (with caveats).
Given the identifier and the security overlay’s mapping to look up key state, any other party, can either verify with the signing public key that a message was non-repudiably sourced by the controller of the identifier (authenticity) (non-repudiable any-end-verifiable ) or can encrypt a message using the encryption public key to ensure that only the controller of the identifier can view it (confidentiality) (restricted only-end-viewable)

To analyze confidentiality we use a 3 party model where the 1st party is sender, the 2nd party is the intended receiver, and the 3rd party is any unintended receiver.

A system with strong end-only viewable confidentiality may have the following properties: 

* 3rd party non-viewability
* 1st party non-viewability (this protects 1st party from liability wrt to 2nd party and protects 2nd party from exploit by 1st party)
* 2nd party partition-ability by 1st party
* Detectable leakage (2nd party to 3rd party)
* Detectable collusion of 2nd party against 1st party (2nd party with 2nd party, 2nd party with 3rd party)
* Detectable collusion of 1st party against 2nd party (1st party with 1st party or 1st party with 3rd party)


## Adoptability via Dumb Crypto

There is an explosion of activity towards developing new cryptographic algorithms. Elliptic curve cryptography has proven to be both highly secure and highly performant compared to other approaches. Unfortunately, new cryptographic algorithms take time to season, as it were, that is, to be battle tested for weaknesses and exploits. Likewise it takes time for new cryptographic algorithms to be supported widely across programing languages and operating systems. It does not help much, practically speaking, to design a protocol that needs advanced crypto that is not yet readily available or sufficiently adoptable for some time. It becomes a theoretical protocol not a very usable one. Hence, a design criteria we employ is that only highly adoptable, well proven, cryptographic operations from broadly supported libraries be used. If a required cryptographic operation is not in those libraries then its not a viable candidate for the protocol.

The best example, of best practices highly adoptable crypto algorithnms is [NaCL](https://nacl.cr.yp.to) see also . The most popular implementation of NaCL is the [libsodium](https://libsodium.gitbook.io/doc/) library which is supported almost universally across programming languages and operating systems. This does not mean that other equivalently adoptable libraries are ruled out but merely that we use libsodium as a starting base-line or departure point. Furthermore, the signing and encryption algorithms in NaCL (Ed25519 and X25519) have been accepted into NIST's roadmap for approved government use (see [Curve25519](https://en.wikipedia.org/wiki/Curve25519)).

In this developent we only consider pre-quantum safe algorithms. Given we are using 

If we can propose a protocol that only needs the operations found in libsodium then we have a practically adoptable baseline for that protocol. 



## Best Authenticity (Public Key Signing)

The standard for strong public key signing algorithms is SUF-CMA (Strong UnForgeability under Chosen Message Attack). When a digital signature has SUF-CMA with at least 128 bits of cryptographic strenght then it is computationally infeasible to forge a signature or to discover the private key by choosing messages. The Libsodium implementation of the NaCL Ed25519 signture scheme has SUF-CMA [Provable Security of Ed25519](https://eprint.iacr.org/2020/823.pdf). So this is our baseline. In comparison the popular ECDSA signature scheme only has the weaker EUF-CMA (Existential UnForgeability under Chosen Message Attack). Any scheme with SUF-SMA also has EUF-CMA This relative weakness of ECDSA has already been exploited in bitcoin. Consequently some additional care must be taken when using signature schemes, like ECDSA, that are not SUF-CMA. 

The Ed25519 in libsodium has some additional security properties that make it a good starting baseline. The following table lists those properties.

|    scheme    | EUF-CMA | SUF-CMA | S-UEO | M-S-UEO | MBS |
|--------------|---------|---------|-------|---------|-----|
| Ed25519-LibS |      X  |      X  |     X |       X |  X  |

In addition to SUF-CMA, the S-UEO (Strong Universal Exclusive Ownership) and M-S-UEO (Malicious Strong Universal Exclusive Ownership) properties provide additonal resiliency against key substitution attacks. Whereas the MBS (Message Bound Security) property ensures a given signature verifies a unique message even for malicious keys.

In addition Ed25519 avoids private key memory leakage attacks and its key clamping mechanism makes the re-use of Ed25519 key pairs for the bi-rationally equivalent X25519 encryption key pairs avoids key leakage attacks via the X25519 keys.

In summary, the baseline for strong authenticity is to use the libsodium Ed25519 signature scheme



## Best Confidentiality (Public Key Encryption)

The standard for strong public key encryption (PKE) algorithm is IND-CCA2 (Indistinguishabilty under Adaptive Chosen Ciphertext Attack). A scheme that has IND-CCA2 also has the weaker IND-CCA (Indistinguishabilty under Chosen Ciphertext Attack) which also has the weaker still IND-CPA (Indistinguishability under Chosen Plaintext Attack). What IND-CCA2 means is that an adversary can't submit ciphertexts in such a way to discover the private key used to decipher those text even when the adversary has access to an oracle that tells the adversary whether or not the ciphertext was a valid ciphertext. In addition an adversary can't construct a valid ciphertext unless the adversary is aware of the associated plaintext. 

For a more in depth description of IND-CCA2 see  [IND-CCA2 part 1](https://blog.cryptographyengineering.com/2018/04/21/wonk-post-chosen-ciphertext-security-in-public-key-encryption-part-1/), and [IND-CCA2 part 2](https://blog.cryptographyengineering.com/2018/07/20/wonk-post-chosen-ciphertext-security-in-public-key-encryption-part-2/)

Earlier forms of public key encryption (PKE) only had IND-CPA or IND-CCA. To get IND-CCA2 requires what is called an Integrated Encryption System (IES) that provides a public key encryption scheme. An IES that uses elliptic curve is called an ECIES. A notable example of an ECIES that provides a public key encryption scheme with IND-CCA2 is the NaCL/Libsodium *sealed box*. What is special about an ECIES like *sealed box* is that it provides the performance of symmetric encryption but the end-only viewability  of public key (asymmetric) encryption. What we mean by end-only viewablity is that only the recipient to whom the box is sealed can decrypt the ciphertext. The sender cannot. The way *sealed box* does this is that it creates a non-interactive Diffie-Hellman (DH) en-de-cryption key using an ephemeral X25519 key pair with the long-term X25519 public key of the receipient. The sender never sees the ephemeral X25519 private key so it can't reconstruct the DH en-de-decryption key. The *sealed box* includes the ephemeral X25519 public key so that the receipient can reconstruct the DH DH en-de-cryption key using its known long-term X25519 private key and the provided ephemeral X25519 public key. The ciphertext is encrypted using symmetric encryption so its high performant and only the holder of the long-term X25519 private key for the long term X25519 public used in the *sealed box* to encrypt, can decrypt the ciphertext.

The following references might be helpful. [ECIES properties](https://ieeexplore.ieee.org/abstract/document/5604194), [ECIES](https://cryptopp.com/wiki/Elliptic_Curve_Integrated_Encryption_Scheme), [IES](https://en.wikipedia.org/wiki/Integrated_Encryption_Scheme), [IES More](https://cryptopp.com/wiki/Integrated_Encryption_Scheme),
[ECIES is IND-CCA2](https://crypto.stackexchange.com/questions/66260/does-elliptic-curve-integrated-encryption-scheme-ecies-provide-ind-cca2-securi), [Crypto Box Seal is ECIES](https://eprint.iacr.org/2020/243.pdf)

More recently the more descriptive term HPKE (Hybrid Public Key Encryption) has been used to describe IES that support public key encryption (PKE). This is memorialized in the [IETF RFC-9180](https://www.rfc-editor.org/rfc/rfc9180.html). RFC-9180 extends basic HPKE with 4 modes of operation in a standardized approach. The Libsodium sealed box has equivalent properties to RFC-9180 base mode [HPKE Analysis](
https://eprint.iacr.org/2020/243.pdf ). For a good introduction to HPKE see [HPKE](https://www.franziskuskiefer.de/p/tldr-hybrid-public-key-encryption/) and [HPKE Blog](https://blog.cloudflare.com/hybrid-public-key-encryption/)

Unfortunately implementations of RFC-9180 are not yet broadly available. So our baseline for IND-CCA2 is the Libsodium Crypto Box Seal or Sealed Box. This uses the  X25519 key pairs and the venerable high performant [XSalsa20](https://libsodium.gitbook.io/doc/advanced/stream_ciphers/xsalsa20) stream cipher with the Poly1305 MAC that together provide the  [XSalsa20-Poly1305](https://en.wikipedia.org/wiki/ChaCha20-Poly1305) AEAD that underlies the sealed box HPKE. 

Unfortunately IND-CCA2 is insufficient by itself to provide strong authenticity of the cipher text. As section 9.1.1 of [RFC-9180](https://www.rfc-editor.org/rfc/rfc9180.html#name-key-compromise-impersonatio) points out, all varients are vulnerable to key-compromise impersonation attacks that may arise from compromise of the receipient's private key (see [KCI](https://eprint.iacr.org/2006/252.pdf)). [RFC-9180](https://www.rfc-editor.org/rfc/rfc9180.html#name-key-compromise-impersonatio) suggests that applications that require resistance against key-compromise impersonation should take extra steps to prevent this attack such as using a digital signature generated with the sender's private key over the HPKE container, which means signing the *sealed box* (when using Libsodium). Essentially a KCI attack enables a compromise of the recipient's keys to impersonate the recipient and perform a type of MITM (man-in-the-middle) attack when performing a key exchange where they substitute a different keypair from that intended by the sender. As a result, sender may not detect the impersonation and therefore will encrypt to an impersonated key-pair see ([KCI explained](https://www.cryptologie.net/article/372/key-compromise-impersonation-attacks-kci/)). This also allows the attacker to purport that the confidential information was meant for the impersonated key pair thus also breaking authenticity. 


## Best Combinded Authenticity and Confidentiality

As developed above for combined stong authenticy and confidentiality we need both hybrid public key encryption to the receiver's key pair (with IND-CCA2) and public key signing from the sender's key pair (with SUF-CMA). There are several ways to combine a cipher text and a signature. In order to protect against key-compromise impersonation attacks we must sign the encryption with the receipients public key in plaintext. This is called encrypt then sign including receiver's key. But merely signing the encryption does not provide complete protection to all attacks. An attacker can strip the signature and resign the ciphertest with the the attacker's key but having never seen the plaintext of the cipher. In some circumstances this might lead the recipient to falsely believe that the sender had knowledge of the plain text when it did not.  For that we must not only bind the receipients public key to the signature in plaintext but also bind the sender's public key inside the ciphertext. 

### Modified ESSR

An approach that protects against both KCI and sender impersonation of the ciphertext is called ESSR for Encrypt Sender's key then Sign Receiver's key. The ESSR approach is detailed here [ESSR](https://eprint.iacr.org/2001/079). For a more accessible explanation see the following [PKAE1](https://neilmadden.blog/2018/11/14/public-key-authenticated-encryption-and-why-you-want-it-part-i/), [PKAE2](https://neilmadden.blog/2018/11/26/public-key-authenticated-encryption-and-why-you-want-it-part-ii/), [PKAE3](https://neilmadden.blog/2018/12/14/public-key-authenticated-encryption-and-why-you-want-it-part-iii/)

The properties that protect against key compromise impersonation attacks are called TUF-PTXT (Third-party UnForgeability of PlainText), TUF-CTXT (Third-party UnForgeability of CipherText), RUF-PTXT (Reciever UnForgeability of PlainText), RUF-CTXT (Receiver UnForgeability of CipherText). Simply using the encrypt then sign approach provides TUF-PTXT, TUF-CTXT, RUF-CTXT but not RUF-PTXT. Signing the receiver's public key in plaintext outside the ciphertext portion of the message binds the receiver's public key making a key substitution attack on the underlying plaintext to the signed ciphertext detectable which gives RUF-PTXT. One weakness of encrypt then sign in that case of a malicious receiver is that if the reciever can substitute a new key pair it can encrypt any message using the sender's public key to the new keypair and purport that the sender indended to encrypt to that new key pair. Signing the ciphertext still leaves a malicious receiver free to find a combination of plaintext and receiver key pair that produces the same ciphertext. This is hard but not computationally infeasible. So we need a way to limit how malicious ciphertexts may be generated and the ESSR way to do that is to bind the intended receiver's public encryption key to the sender's signature. This way the ciphertext must decrypt using the intended reciever's key pair or its invalid. The security of an encryption system ensures that it is computationally infeasible that given the same key two different plaintexts do not encrypt to or decrypt from the same ciphertext, i.e. a given plaintext must roundtrip through the cryptsystem given the same key. This is a property of a secure stream cipher such as xSalsa20 used for sealed box (see [XSalsa20](https://cr.yp.to/snuffle/xsalsa-20110204.pdf)).   

This is shown in the following table: (see [ESSR](https://eprint.iacr.org/2001/079))

|    scheme         | TUF-PTXT | TUF-CTXT | RUF-PTXT| RUF-CTXT | 
|-------------------|----------|----------|---------|----------|
| Encrypt-then-Sign |      X   |      X   |         |       X  | 
| ESSR              |      X   |      X   |     X   |       X  | 


In summary, as a result of binding the sender's public key inside the ciphertext and binding the receiver's public key in the enclosing signed plain text an adversary is prevented from forging messages that compromise either authenticity or confidentiality. Thus ESSR provides both strong authenticity and strong confidentiality.

We apply ESSR to KERI by modifying the basic ESSR approach to support unbounded-term identifiers by replacing the sender's long-term public key with its AID and the receiver's long-term public key with its AID. The public keys are then securely looked up from the respective KELs of the sender and receiver. This is diagrammed below.

![](https://i.imgur.com/kmhipqT.png)



As shown in the diagram above, the sender's AID appears both in the verifiable payload plaintext and the restricted payload ciphertext portion. The first appearance in the plaintext portion is so that the receiver or any party can validate the signature without first decrypting the ciphertext. This essential for upstream DDOS protection using load balancers and firewalls which do not have access to the receiver's private decryption key and therefore can't make a drop decision based on the sender via a whitelist or blacklist. To reiterate sharing the private decryption key with a load balancer or firewall upstream of the recipient in order to discover the sender's public verification key would break confidentiality. 

Another limitation of removing the plain-text Sender AID is that the message's ciphertext is not publically verifiable. This means the authenticity of the ciphertext is not verifiable merely using public information. In some cases a 3rd party may need to verify the authenticity of the ciphertext without seeing the plaintext from which the ciphertext was generated.

That said there may be some applications where upstream DDOS and/or firewall protection and/or public verifiabilityis not needed. Because the sender AID appears twice, once in plain text and once In that caseDDOS or firewall protection (security) could be traded for not exposing the sender's AID (privacy) without compromising authenticity or confidentiality of the message. The following diagram shows the message without the sender's AID in plaintext only in ciphertext.

![](https://i.imgur.com/3HX48nX.png)


A subtle advantage of using AIDs instead of public keys is that it may make reciever key compromise impersonation more difficult. This allows us to make modifications to the baseline ESSR approach. This is because the only valid receiver's keys are those in the receiver's KEL. An attacker can't substitute a different key not in the KEL, it must use a stale key. To elaborate, using AIDs, an attacker can't purport any plaintext to have been encrypted to a different key pair other than one that appears in the receiver's KEL. So any ciphertext must decrypt using one of the receiver's keypairs either the current one or a stale one. A malicious receiver can't encrypt any plaintext with any keypair it chooses.  The malicious receiver must either find a combination of stale key pair and plaintext that generates the same signed ciphertext as its current keypair or confuse the sender into signing ciphertext generated by the receiver with its current encryption keypair for plain text not yet seen by the sender. 

Given that the only choices for key-pair are those in the receiver's KEL the ability of a compromised receiver to generate any signed ciphertext using some combination of key-pair and plaintext is more limited. This looks like a type of birthday table attack [BDay Attack](https://en.wikipedia.org/wiki/Birthday_attack). Given that the key choice is so limited, a malicious reciever may not gain any appreciable advantage in being able to construct  the same ciphertext using a stale keypair as the ciphertext constructed with the current key pair but already signed by the sender.  When that is the case, (i.e. not vulnerable to BDay attack) and there is also a way for an honest reciever to look up the intended long-term public decryption key without having its AID in the message, then the destination AID may be removed from the message. This variant is shown in the diagram below.

![](https://hackmd.io/_uploads/B1y-omLen.png)

In the case where the receiver follows the policy of only one relationship identifiers (OORI) then the sender AID in the ciphertext may not be necessary. With an OORI policy the receiver only accepts messages to a given receiver AID from a given sender AID. This means that any other sender can not strip the signature for a given ciphertext to a given receiver AID and replace it with its own because the reciever will drop it if the sender AID does not correspond to that allowed for that receiver AID. This policy may remove the need to include the sender's AID in the ciphertext. This variant is diagrammed below.  

![](https://hackmd.io/_uploads/BJLi2m8xh.png)


Furthermore, adding a replay attack prevention mechanism against reuse of the stale keys would provide additional protection against receiver KCI by both the sender and any third parties. A simple replay attack protection mechanism would be to adopt a policy that once the key state has been updated no messages signed with stale keys may be accepted. A more complicated replay attack protection mechanism would be for the sender to anchor a digest (content address) of the message in the sender's KEL. In this case the signature attached to the message is not needed only a reference to the anchor in the sender's KEL. 


In summary, the baseline for best security for messages follows the full ESSR message shown in the diagram above (labeled Authenticatible Confidential Message). Any variation from the full ESSR format may make a security trade-off induced by removing one of its protections. This trade-off should be explicated in the protocol design so as to inform its proper use.


## Relationships

### Basics

The final preliminary before directly addressing privacy is to define what we mean by relationships. Suitable relationship properties will give us a handle on managing privacy with respect to exploitably correlatable identifiers. We define a relationship as two-tuple (duple) or pairing between two AIDs. The pairing may also have a direction (bi-directional or uni-directional). Thus for any two AIDs there are three different relationships. Let `A` and `B` denote each aid and let `<>`, `->`, `<-`, and  denote directionality. Then we can represent each of the three relationships as follows:

```
(A, <>, B)
(A, ->, B)
(A, <-, B)
```

The first one above is bi-directional, the second is uni-directional from `A` to `B`, and the third is uni-directional from `B` to `A`.

We also use the convention that a given party that controls an AID in a relationship always puts its own AID as the first AID. This we call the `(local, remote)` convention. When party `A` always denotes itself as `local` with respect to its relationship with `B` as ``(A, ->, B)`` and party `B` denotes itself as local with respect to its relationship with party `A` as `(B, <-, A)`. This convention makes the implementation of relationship graphs consistent in that they are always directed from the perspective of the `local` party.

When a given party has multiple AIDs then we can compactly denote those by subscripting. Say party *X* has three different AIDs we denote them *X<sub>0</sub>*, *X<sub>1</sub>*, *X<sub>2</sub>*

Diagrams of showing the relationship conventions are shown below:

![](https://hackmd.io/_uploads/B11GSBIlh.png)

![](https://hackmd.io/_uploads/BJYMrB8g2.png)

![](https://hackmd.io/_uploads/HJ5mrBLxn.png)

Now that we have introduced the concept of a relationship we define more formally its properties:


* A cryptonym is a cryptographically derived pseudonym with at least 128 bits of entropy in the derivation.  
* An AID is a cryptonym that is also securely attributable to one or more key pairs.
* A relationship is a pairing of two AIDs, one each from a different controller.  
* Two different AIDs are by themselves uncorrelated in an information-theoretic security sense in that knowledge of one by itself provides no information about the other.  
* A context is a set of events.  
* Two contexts are disjoint with respect to an AID when that AID appears in one or more events in one set but does not appear in any event in the other set. 
* A relationships is not a communication channel but the events sent over a communication channel may be a context for the aids in a relationship.
* A partition is a set of contexts with mutually disjoint relationships.
* Any set of relationships using ORIs (One Relationship Identifiers) may form a partition.
* Any two members (contexts) of a partition may be correlatable due to other information associated with any pair of contexts but the partitioned relationships by themselves provide no correlatable information i.e. partitioned relationships are by themselves not correlatable
* An AID that is common to any subset of events within a context provides a perfectly correlatable feature across those common event in that context.
* The secure attributability of an AID together with its perfect correlatability within a context enable secure reputational trust (good or bad) in that AID within that context
 * partitions balance the concerns of:
    - strong authenticity and strong confidentiality within a context with,
    - sufficiently strong privacy between contexts.


### Relationship Graphs

Given the definition of a relationship we can define a relationship graph. Each controller can manage its AIDS and the associated relationships with AIDs form other controllers. This is shown below for 4 controllers denoted *A*, *B*, *C*, *D*. The graph is all includsive showing both sides of each relationship for all controllers, but a given controller may in actuality only see a partial version of the graph.

![](https://hackmd.io/_uploads/BkQ5D-Yx2.png)


A relationship graph is actually a subset of what I call an identity or more acurately and identifier graph. A relationship graph is the intersection of the indentifier graphs for multiple controllers. An identifier graph shows the correlatable connections between the identifiers a given controller uses for itself and the identifiers other controller use for themselves. The theory of identifier graphs (and relationship graphs as a subset) was first explicated in the [Open Reputation] (https://github.com/SmithSamuelM/Papers/blob/master/whitepapers/open-reputation-low-level-whitepaper.pdf) whitepaper (circa 2015). It has been prominently mentioned since then in various presentations (primarily on reputation systems) and other whitepapers [identity system essentials](https://github.com/SmithSamuelM/Papers/blob/master/whitepapers/Identity-System-Essentials.pdf). A self-identity graph is shown below:

![](https://hackmd.io/_uploads/H1g8YWFg3.png)


Given a relationshp graph with partioned relationships (AIDs used in partioned contexts) a given controller can isolate the correlatability of those AIDs. We now have the preliminaries necessary to start exploring how to design protocols that as a point of departure make no compromises on authenticity and confidentiality but provide sufficient privacy by effectively mitigating exploitation due to identifier correlation.

## Syntax

In order to more efficienty represent the messages or packets for any given protocols we adopt the following textual syntactical conventions.  

###  AIDs

AIDs from a controller are denoted by a capital letter representing the controller with a unique subscript for each AID. For example, the controller *X* may have AIDS,
*X<sub>0</sub>*, *X<sub>1</sub>*, *X<sub>2</sub>*

### Signatures

The content signed by a signature is delimited with angle brackets, *<>* surrounding the data being signed and is followed by the AID of the signer which denotes the attached signature itself. For example,  
*< data >X<sub>0</sub>*,  
means *data* is signed with attached signature using the current signing keys for AID, *X<sub>0</sub>*. When more than one signature is attached then the signatures are delimited by a comma separated list surrounded by parenthesis as follows:  
*< data >(X<sub>0</sub>, X<sub>1</sub>)*.

### Ciphertext

Ciphertext is delimited with curly brackets surrounding the ciphertext and is followed by the AID whose current en-de-cryption keypair provides the public encryption key and private decryption key. For example,  
*{ data }Y<sub>0</sub>*,  
means *data* is ciphertext encrypted to *Y<sub>0</sub>*. Fields within cipher text are comma separated and may be grouped using parenthesis.

### Messages

A message is delimited by square brackets surrounding the components. Fields may be comma separated and may be grouped with parenthesis. Nested messages are delimited by nested square brackets. For example,  
*[src X<sub>0</sub>, dst Y<sub>0</sub>, data]*,  
denotes a message with three plaintext fields, the first is the src address given by the AID *X<sub>0</sub>*, the second is the dst address given by *Y<sub>0</sub>*, the third is a data field.

A more complicated example is as follows:  
*<[src X<sub>0</sub>, dst Y<sub>0</sub>, {src X<sub>0</sub>, data}Y<sub>0</sub>]>X<sub>0</sub>*,
which denotes a message signed by X<sub>0</sub> with embedded ciphertext encrypted to Y<sub>0</sub> which ciphertext includes src and data. The message also includes  plaintext fields for the src and dst addresses. This is an example of message in full ESSR format.

### Header

Any practical protocol requires a header with one or more header fields in plaintext that provide metadata about the packet. This metadata typically includes version, protocol type, packet type, and depending on the protocol may include service type and transaction state support like sequence numbers etc. For security this header should be protected by being placed in the signed portion of any packet. The protected header enables a parser to interpret the packet without exposing it to attack.  A practical version of the example above would include such a header as shown below:

*<[header, src X<sub>0</sub>, dst Y<sub>0</sub>, {src X<sub>0</sub>, data}Y<sub>0</sub>]>X<sub>0</sub>*

However, for the purposes of understanding how to analyze the trade-offs of the fundamental properties of authenticity, confidentiality, and privacy the header details are largely irrelevant. Without loss of generality then, it is assumed that any practical packet will have a header with whatever additional metadate is required by the specific protocol. But for the purposes of this exposition we will simplify the message descriptions by leaving out the header metadata unless it is relevant. 

### IP Addresses and Header

In order to route a message over the internet, the packet must have an IP (UDP or TCP) header with a src (source) address and port and a dst (destination) address and port.  We assume that the IP header is not protected by the signature. This assumption is still valid for other transports besides IP. We assume that there will be a transport header with  src and dst addresses or more generally src and dst service endpoints. Therefore without loss of generality we simply label the transport endpoints with the modifier *ip*. We also assume that for each AID there may be a default corresponding transport endpoints for source and destination. Of course a given AID may have multiple endpoints and this may be distinguished with *role* label but this is a management detail that does not impact the primary issue of correlation.

Suppose for AID, X<sub>0</sub>, we denote its transport source endpoint as, *src ip X<sub>0</sub>*, and we denote its destination transport endpoint as *dst ip X<sub>0</sub>*. The transport source and destination addresses may appear in a message in its protected (signed) metadata besides the unprotected (unsigned) IP header as needed for different virtual routing purposes.


A transport header (ip) is delimited with vertical brackets, *||*. Thus for a packet between source AID *X<sub>0</sub>* and destination AID *Y<sub>0</sub>* the transport header is given by,

*|src ip X<sub>0</sub>, dst ip Y<sub>0</sub>|*.

An example of a full packet with prepended transport header is show below,  

*|src ip X<sub>0</sub>, dst ip Y<sub>0</sub>|<[src X<sub>0</sub>, dst ip Y<sub>0</sub>, dst Y<sub>0</sub>, {src X<sub>0</sub>, data}Y<sub>0</sub>]>X<sub>0</sub>*

Please note that in this particular example, the destination transport (ip) address appears twice. Once in the unprotected transport header and once more in the protected message body.

Unlike an AID an IP address is public by nature. This means each IP address comes with an inherently correlatable global context. It is therefore not private by default. IP addresses are assigned largely on a geographic basis so the mere appearance of an address provides potentially exploitable correlatable information. IP addresses must appear in plaintext in the IP header in order for the associated IP packet to be routable over the internet. This appearance inherently correlates sources to destinations. As a result the challenge for mitigating exploitation by aggregators of identifiers starts with mechanisms that mitigate correlation of the IP addresses first not the AIDs. Indeed, as we shall see below. Mechanisms that mitigate IP address correlation may, with little or no extra effort, also mitigate AID correlation.


## OOB Setup

Any secure over-the-wire protocol on a given communications channel between two parties requires at least one out-of-band-authentication (OOBA) factor be provided in order to protect against a man-in-the-middle (MITM) attack. The OOBA is out-of-band (OOB) with respect to the communications channel. The OOBA does not have to be directly between the two parties but may involve a third party. There just needs to be information available to the two parties that they obtain out-of-band with respect to the MITM so that the MITM does not have access to that information.

This means that without loss of generality we can make the assumption that every protocol has at least one OOB setup. To simplify the discussion we can assume that the two parties exchange identifiers OOB. We can always complicate things by having more elaborate OOB setups. But this does not change the requirement that at least one OOBA factor happens OOB.

Given at least one OOBA factor in the setup solves a second problem. This problem is called cheap pseudonymity. Cryptonymous identifiers may be created on the fly. Because they are derived from high entropy capture (i.e are long strings of pseudorandom  characters) there are a practically infinite number of them. This makes them cheap. So any protocol that wants to be protected from DDOS where a malicious entity just creates a new identifier in order to bypass a filter, needs some way to manage cheap pseudonymity. An OOBA method makes any identifier relatively expensive. An attacker can't create a replacement without repeating the OOBA for that new identifier.

Finally, an OOBA (out-of-band-authentication) may be simultaneously strongly authentic, confidential and private. This provides a secure basis for the in-band transmission without yet compromising any of authenticity, confidentiality, or privacy.

As a result we will use an initial OOBA setup that is an exchange of identifiers as the default departure point for protocol design. Any other setup may then be analyzed by comparison for the trade-offs in security with respect to authenticity, confidentiality, privacy, ddos protection, friction etc. that it makes.


## Baseline Protocols

### Direct 

Suppose we have a pair of entities denoted by AIDs *A* and *B*. An OOB exchange of identifiers provides to entity *A*, *B*'s' AID *B<sub>0</sub>* and *B*'s service endpoint *ip B<sub>0</sub>*, it also provides to entity *B* with *A*'s' AID *A<sub>0</sub>* and *A*'s service endpoint *ip A<sub>0</sub>*. *A* and *B* form a bidirectional relationship *(A<sub>0</sub>, <>, B<sub>0</sub>)*. The context of this relationship is direct communication between *A* and *B*

An authenticatable message from *A* to *B* with confidential content data using ESSR is denoted as follows:  

*|src ip A<sub>0</sub>, dst ip B<sub>0</sub>|<[src A<sub>0</sub>, dst B<sub>0</sub>, {src A<sub>0</sub>, data}B<sub>0</sub>]>A<sub>0</sub>*

Simlarly authenticatable message from *B* to *A* with confidential content data using ESSR is denoted as follows:  

*|src ip B<sub>0</sub>, dst ip A<sub>0</sub>|<[src B<sub>0</sub>, dst A<sub>0</sub>, {src B<sub>0</sub>, data}A<sub>0</sub>]>B<sub>0</sub>*

The ip addresses and AID are correlatable by any ISP or intermediary the watches the messages since they all appear in plaintest. But the confidential content data is not observable. The only correlatable information is that *A<sub>0</sub>* is talking to *B<sub>0</sub>* using IP endpoints *ip A<sub>0</sub>* and *ip B<sub>0</sub>*. This may be monetizable by an ISP for advertising aggregation. ON the other hand, because it is strongly authentic and confidential, it by itself, is not very exploitable by an identity thief. 

So its strongly authentic and confidential but not private.

Now suppose that *A* and *B* want to communicate about a business deal using a relationshp that is unique to the business deal context, but not the communication context.
For that business deal context they use the relationship *(A<sub>1</sub>, <>, B<sub>1</sub>)*. Since this context is not used for communication it does not have any associated ip addresses. Instead it must be transported over a communication context relationship.

Suppose there is an OOB exchange of *A<sub>1</sub>* and *B<sub>1</sub>* between *A* and *B*. Now *A* and *B* can communicate totally authentically, confidentially, and privately using the relationship associated with the business deal context but leveraging the communications context relationship. These are shown as follows:

From *A* to *B*:

*|src ip A<sub>0</sub>, dst ip B<sub>0</sub>|<[src A<sub>0</sub>, dst B<sub>0</sub>, {src A<sub>0</sub>, <[src A<sub>1</sub>, dst B<sub>1</sub>, data]>A<sub>1</sub> }B<sub>0</sub>]>A<sub>0</sub>*

Embedded in this confidential message is an authenticatable message of content data from *A<sub>1</sub>* to *B<sub>1</sub>* and signed by *A<sub>1</sub>*.


Similarly from *B* to *A*:

*|src ip B<sub>0</sub>, dst ip A<sub>0</sub>|<[src B<sub>0</sub>, dst A<sub>0</sub>, {src B<sub>0</sub>, <[src B<sub>1</sub>, dst A<sub>1</sub>, data]>B<sub>1</sub> }A<sub>0</sub>]>B<sub>0</sub>*

Embedded in this confidential message is an authenticatable message of content data from *B<sub>1</sub>* to *A<sub>1</sub>* and signed by *B<sub>1</sub>*.


No third party observer can view that there is a relationship *(A<sub>1</sub>, <>, B<sub>1</sub>)* via the communication's context messages. Thus there is no leakage via correlation to any other use of the *(A<sub>1</sub>, <>, B<sub>1</sub>)* relationship. The *(A<sub>1</sub>, <>, B<sub>1</sub>)* relationship is a no compromise, authentic, confidential, and private relationship betwen *A* and *B*. 

### Relationship Formation Protocol

Instead of using another OOB setup to form *(A<sub>1</sub>, <>, B<sub>1</sub>)* we can leverage the OOB setup used to form the pre-existing communications context relationship *(A<sub>0</sub>, <>, B<sub>0</sub>)*. Essentially, *(A<sub>0</sub>, <>, B<sub>0</sub>)* acts as a third party reference to form *(A<sub>0</sub>, <>, B<sub>1</sub>)*. This suffices to protect against MITM attacks and cheap pseudonymity. We just need a new protocol type with message types for forming a new relationship. This new protocol could include a relationship formation initiation message that could have semantics something to the effect of:  

I the controller of *A<sub>0</sub>* wish to establish a new relationship using *A<sub>1</sub>* with the controller of *B<sub>0</sub>*. This would be signed by *A<sub>1</sub>* as well as the *A<sub>0</sub>* signature wrapping the message.

Then the controller of *B<sub>0</sub>* can either refuse the intitation with a NACK or can accept with an relationship formation acceptance message that could have semantics something to the effect of:  

I the controller of *B<sub>0</sub>* accept a new relationship using *B<sub>1</sub>* with the controller of *A<sub>1</sub>*. This would be signed by *B<sub>1</sub>* as well as the *B<sub>0</sub>* signature wrapping the message. The accept message can include a digest (said) of the initiation message which digest cold be used as the transaction ID. Then the AIDs in the initiation message do not need to be repeated in the accept message. Adding a nonce to the first message (initiation) means that its context cannot be guessed using a rainbow table attack which could be used when logging transcations. When the digest of the first message in a digest (hash) chained set of messages has high entropy then so does the digest of each of the subsequent chained message.

Such hash chaining of multi-message protocols where the messages are also signed makes the set of message in the transaction a verifiable data structure. This is virtually impervious to attack. Morever because cryptographic digests are universally unique,  using a digest for the transaction ID or a chain of hash chained digests as a transaction chain enables a given transaction to span different communication contexts and still be easy to lookup in a transaction database that is indexed by such digests. Finally, this approach makes it easy to detect replay attacks.

The relationship formation protocol would allow any pre-existing relationship to bootstrap a new relationship without having to perform yet another OOB setup to exchange identifiers. This could also be used to establish new communications context relationships by adding the exchange of the service endpoint ip addresses in addition to the AIDs for the newly formed relationship. This is all accomplished using a no compromise secure authentic, confidential, and private exchange between the two parties.

The reference relationship does not leak any correlatable information about the newly formed relationshp. The only correlatable information is that each communications relationship AIDs are correlated the that relationships ip addresses. To reiterate, There is no correlation to any other relationship including to other communication relationships unless they both use the same or related ip addresses.

Therefore if we can solve ip address correlation we have solved third party exploitablity of identifiers for aggregation advertising.



### One Shared Intermediary

The direct messaging protocol above does not provide correlation privacy with respect to identifiers using in the the communications context relationship *(A<sub>0</sub>, <>, B<sub>0</sub>)*  between *A* and *B*. Both the ip addresses and the AIDs or in plaintext and may be correlated by either of the ISPs for *A* and *B*. This section will detail how a single shared intermediary may be used to provide privacy for the AIDS and ip addresses associated with a communications context relationship between *A* and *B*. The intermediary may provide sufficient herd privacy so that the communications between *A* and *B* are largely unexploitable for advertizing aggregation purposes.

Let the intermediary controller be denoted by *C*. 

Suppose *A* forms a communication context relationship with *C* given by *(A<sub>2</sub>, <>, C<sub>0</sub>)* with ip endpoints *ip A<sub>2</sub>* and *ip C<sub>0</sub>*. The initial exchange of these identifiers is performed OOB so that the initial setup of the relationship is securely authentic, confidential, and private.

Suppose *B* forms a communication context relationship with *C* given by *(B<sub>2</sub>, <>, C<sub>1</sub>)* with ip endpoints *ip B<sub>2</sub>* and *ip C<sub>1</sub>*. The initial exchange of these identifiers is performed OOB so that the initial setup of the relationship is securely authentic, confidential, and private.

Further suppose that *C* has communication relationships with a large number of other entities (say > 1000).

We assume that *A* and *B* have formed at least one relationship by exchanging identifiers. This could have been formed using their direct communications relationship or using some OOB exchange of identifiers. For the sake of convenience let that relationship be denoted *(A<sub>1</sub>, <>, B<sub>1</sub>)*. Now *A* and *B* can communicate using this relationship in a private (non-correlatable identifier) manner by leveraging each of their communication's relationships with C. This requires that *A* and *B* also exchange OOB the AIDs of of the intermediary's side of their respective communication's relationships with *C*. In this case *A* also knows *C<sub>1</sub>* and *B* also knows *C<sub>0</sub>*. 

Further, *C* maintains a database that maps the AIDs of each of its communications relationshps to the corresponding ip service endpoints. this allows *C* to lookup the current service endpoint given an AID. So for example *C* has the following signed entries in its database:

*<[C<sub>0</sub>,<[A<sub>2</sub>, ip A<sub>2</sub>]>A<sub>2</sub>]>C<sub>0</sub>*  
*<[C<sub>1</sub>,<[B<sub>2</sub>, ip B<sub>2</sub>]>B<sub>2</sub>]>C<sub>1</sub>*  

This makes the database zerotrust because it is signed at rest. An attacker can't forge or impersonate the ip service endpoint for any given relationship nor impersonate the AID of an  associated relationshp and thereby fool the intermediary into misdirecting traffic to the wrong endpoint or AID or to the right endpoint but using the wrong relationship (either of these would be the equivalent of a DNS hijack). 


From *A* to *B* via *C*:

First hop from *A* to *C*:

*|src ip A<sub>2</sub>, dst ip C<sub>0</sub>|<[src A<sub>2</sub>, dst C<sub>0</sub>, {src A<sub>2</sub>, dst C<sub>1</sub>, <[src A<sub>1</sub>, dst B<sub>1</sub>, {src A<sub>1</sub>, data}B<sub>1</sub>]>A<sub>1</sub> }C<sub>0</sub>]>A<sub>2</sub>*

Upon receipt of this message *C* verifies the source as coming from one of its authenticated relationships. In this case *A<sub>2</sub>*. Then *C* can extract the destination *C<sub>1</sub>* to lookup first *B<sub>2</sub>* and then the associated dest *ip B<sub>2</sub>*. 

Last hop from *C* to *B*:

Finally *C* forms a new message to *B* as follows:

*|src ip C<sub>0</sub>, dst ip B<sub>2</sub>|<[src C<sub>0</sub>, dst B<sub>2</sub>, {src C<sub>0</sub>, <[src A<sub>1</sub>, dst B<sub>1</sub>, {src A<sub>1</sub>, data}B<sub>1</sub>]>A<sub>1</sub> }B<sub>2</sub>]>C<sub>0</sub>*

Now *B* can verify that this came from *C<sub>1</sub>* and then decrypt the contents which it can then verify as coming from *A<sub>1</sub>*. No 3rd party every sees the identifiers from *(A<sub>1</sub>, <>, B<sub>1</sub>)*. Therefore this relationship is  private with respect to correlatable identifiers. 

All any 3rd party ISP sees is that  has *A<sub>2</sub>* has relationship with *C* and *B<sub>2</sub>* has a relationship with *C* but can't correlate those relationships to *(A<sub>1</sub>, <>, B<sub>1</sub>)* given sufficient herd privacy of *C*'s relationships with others. This provides secure authenticity, confidentiality and privacy between *(A<sub>1</sub>, <>, B<sub>1</sub>)* that only requires that *A* and *B* trust *C* not to reveal that *(A<sub>1</sub>, <>, B<sub>1</sub>)* exists. 

Note that the content data from *A<sub>1</sub>* to *B<sub>1</sub>* is not seen by *C*, only the AIDs. Moreover, the AIDS in *(A<sub>1</sub>, <>, B<sub>1</sub>)* are never stored on disk by *C*. They are only ever in memory. Which means that an attacker who compromises the disk storage of *C* cannot leak *(A<sub>1</sub>, <>, B<sub>1</sub>)*.  An attack on *C*'s protected memory processes is much more difficult than an attack on *C*'s disk storage. This is more than sufficient to protect against 3rd party correlation for advertising aggregation.

Furthermore note that *A* does not need to know the *B* side of *B*'s relationship with *C* and *B* does not need to know the *A* side of *A*'s relationship with *C*. So neither can leak that information. An attacker has to get that by attacking *C* not either *A* or *B*.

The primary limitations of this approach are as follows:
* Both *A* and *B* must trust *C*. This may be problematic in general.  
* The ISP of *C* has access to both the source and destination IPs of all *C*'s packets. With some extra work, the ISP could correlate observable statistics (not identifier per se) about the packets such as time-of-arrival and time-of-departure, packet size, packet type, protocol types etc. Given enough traffice between *A* and *B* this might be enough for the ISP to get some degree of correlation. It is unlikely however that such a weak correlation would be monetizable by the *C*'s ISP for advertising aggregation.   

In the next section we solve both of the limitations above by using two intermediaries. One trusted by *A* and one trusted by *B*. As long as both intermediaries do not share the same ISP then there would not be event weak correlation between *A* and *B* solely based on their joint traffic. This of course assumes sufficient herd privacy of each of the intermediaries.

### Non-Shared Intermediaries

This differs from the previous protocol in that *A* and *B* each have an intermediary that they each trust. Let *C* denote *A*'s intermediary and *D* denote *B*'s intermediary.

Suppose *A* forms a communication context relationship with *C* given by *(A<sub>2</sub>, <>, C<sub>0</sub>)* with ip endpoints *ip A<sub>2</sub>* and *ip C<sub>0</sub>*. The initial exchange of these identifiers is performed OOB so that the initial setup of the relationship is securely authentic, confidential, and private.

Suppose *B* forms a communication context relationship with *D* given by *(B<sub>2</sub>, <>, D<sub>0</sub>)* with ip endpoints *ip B<sub>2</sub>* and *ip D<sub>0</sub>*. The initial exchange of these identifiers is performed OOB so that the initial setup of the relationship is securely authentic, confidential, and private.

Further suppose that *C* and *D* each have independent communication relationships with a large number of other entities (say > 1000).

Further, both *C* and *D*  maintain a database that maps the AIDs of each of its communications relationshps to the corresponding ip service endpoints. this allows each to lookup the current service endpoint given an AID. 

For example *C* has the following signed entry in its database:

*<[C<sub>0</sub>,<[A<sub>2</sub>, ip A<sub>2</sub>]>A<sub>2</sub>]>C<sub>0</sub>*  

Likewise, *D* has the following signed entry in its database:

*<[D<sub>0</sub>,<[B<sub>2</sub>, ip B<sub>2</sub>]>B<sub>2</sub>]>D<sub>0</sub>*  

This makes the databases zerotrust because there are signed at rest. An attacker can't forge or impersonate the ip service endpoint for any given relationship nor impersonate the AID of an associated relationship and thereby fool the intermediary into misdirecting traffic to the wrong endpoint or AID or to the right endpoint but using the wrong relationship (either of these would be the equivalent of a DNS hijack). 

We assume that *A* and *B* have formed at least one relationship by exchanging identifiers. This could have been formed using their direct communications relationship or using some OOB exchange of identifiers. For the sake of convenience let that relationship be denoted *(A<sub>1</sub>, <>, B<sub>1</sub>)*. Now *A* and *B* can communicate using this relationship in a private (non-correlatable identifier) manner by leveraging each of their communication's relationships with their respective intermediaries, *A* with *C* and *B* with *D). This requires that *A* and *B* also exchange OOB the AIDs of the intermediary side of their respective communication's relationships with their own intermediares. In this case *A* also knows *D<sub>0</sub>* and *B* also knows  *C<sub>0</sub>*.  

Finally, in order for *C* and *D* to communicate with each other, they must both have an AID and corresponding ip service endpoint that will send and receive authenticated traffic from other intermediaries. If this is public then there is no need for an OOB exchange of this information. If not then *A* and *B* must also exchange the cross intermediary AID and ip service endpoint of each of their intermediaries. Lets *C*'s be denoted *C<sub>2</sub>* with *ip C<sub>2</sub>* and *D*'s be denoted *D<sub>2</sub>* with *ip D<sub>2</sub>*. Now any previously unathenticated intermediary is indistinguishable from a DDOSing cheap pseudonym with respect a given intermediary. Therefore some form of reputation is required. One approach would be for *A* to authorize (*D<sub>2</sub>*, *ip D<sub>2</sub>*) to *C* and for *B* to authorize (*C<sub>2</sub>*, *ip C<sub>2</sub>*)* to *D*. Since *A* is already reputable from *C*'s standpoint and *B* is already reputable from *D*'s standpoint. A signed letter message acting as a letter of reference should suffice.

As a result *C* will have a database of approved intermediaries with the following entry:

*<[A<sub>2</sub>, D<sub>2</sub>, ip D<sub>2</sub>]>A<sub>2</sub>*.  

Intermediary *C* is thereby DDOS protected and can whitelist any messages from D<sub>2</sub>. Morever *C* has the forwarding *ip ip D<sub>2</sub>* for any messages destined for *D*.

Likewise *D* will have a database of approved intermediaries with the following entry:

*<[B<sub>2</sub>, C<sub>2</sub>, ip C<sub>2</sub>]>B<sub>2</sub>*.  

Intermediary *D* is thereby DDOS protected and can whitelist any messages from C<sub>2</sub>. Morever *D* has the forwarding *ip C<sub>2</sub>* for any messages destined for *C*.

Given this setup we can now send a fully secure authentic, confidential, and private message from *A* to *B* for relationship *(A<sub>1</sub>, <>, B<sub>1</sub>)* through intermediarys *C* and *D*.

From *A* to *B* via *C* and *D*:

First hop from *A* to *C*:

*|src ip A<sub>2</sub>, dst ip C<sub>0</sub>|<[src A<sub>2</sub>, dst C<sub>0</sub>, {src A<sub>2</sub>, dst D<sub>2</sub>, dst D<sub>0</sub>, <[src A<sub>1</sub>, dst B<sub>1</sub>, {src A<sub>1</sub>, data}B<sub>1</sub>]>A<sub>1</sub>}C<sub>0</sub>]>A<sub>2</sub>*  

Upon receipt of this message *C* verifies the source as coming from one of its authenticated relationships. In this case *A<sub>2</sub>*. Then *C* can extract the nearside intermediary destination *D<sub>2</sub>* to lookup *ip D<sub>2</sub>* as well as the farside intermediary destination *D<sub>2</sub>*.

Next hop from *C* to *D*:

Next *C* forms a new message to *D* as follows:

*|src ip C<sub>2</sub>, dst ip D<sub>2</sub>|<[src C<sub>2</sub>, dst D<sub>2</sub>, {src C<sub>2</sub>, dst D<sub>0</sub>, <[src A<sub>1</sub>, dst B<sub>1</sub>, {src A<sub>1</sub>, data}B<sub>1</sub>]>A<sub>1</sub>}D<sub>2</sub>]>C<sub>2</sub>* 

Now *D* can verify that the message come from an approved source *C<sub>2</sub>* and then decrypt to extract the farside destination *D<sub>0</sub>*. Given *D<sub>0</sub>*, *D* can lookup B<sub>2</sub> and ip B<sub>2</sub>]. 

Last hop from *D* to *B*:

Finally *D* forms a new message to *B* as follows:

*|src ip D<sub>0</sub>, dst ip B<sub>2</sub>|<[src D<sub>0</sub>, dst B<sub>2</sub>, {src D<sub>0</sub>, <[src A<sub>1</sub>, dst B<sub>1</sub>, {src A<sub>1</sub>, data}B<sub>1</sub>]>A<sub>1</sub> }B<sub>2</sub>]>D<sub>0</sub>*

Now *B* can verify that this came from *D<sub>o</sub>* and then decrypt the contents which it can then verify as coming from *A<sub>1</sub>*. No 3rd party every sees the identifiers from *(A<sub>1</sub>, <>, B<sub>1</sub>)*. Therefore this relationship is  private with respect to correlatable identifiers. 

All any 3rd party ISP sees is that  has *A<sub>2</sub>* has relationship with *C* and *B<sub>2</sub>* has a relationship with *D* but can't correlate those relationships to *(A<sub>1</sub>, <>, B<sub>1</sub>)* given sufficient herd privacy of *C*'s and *D*'s relationships with others. This provides secure authenticity, confidentiality and privacy between *(A<sub>1</sub>, <>, B<sub>1</sub>)* that only requires that *A* trust *C* that *C* trust *A* and *B* trust *D* and *D* trust *B*. *A* and *B* both trust that *C* and *D* do not reveal that *(A<sub>1</sub>, <>, B<sub>1</sub>)* exists. 

Note that the content data from *A<sub>1</sub>* to *B<sub>1</sub>* is not seen by either *C* or *D*, only the AIDs. Moreover, the AIDS in *(A<sub>1</sub>, <>, B<sub>1</sub>)* are never stored on disk by either *C* or *D*. They are only ever in memory. Which means that an attacker who compromises the disk storage of *C* or *D* cannot leak *(A<sub>1</sub>, <>, B<sub>1</sub>)*.  An attack on *C*'s or *D*'s protected memory processes is much more difficult than an attack on *C*'s or *D*'s disk storage. This is more than sufficient to protect against 3rd party correlation for advertising aggregation.

Furthermore note that *A* does not need to know the *B* side of *B*'s relationship with *D* and *B* does not need to know the *A* side of *A*'s relationship with *C*. So neither can leak that information. An attacker has to get that by attacking *C* and *D* not either *A* or *B*. An attack on *C*'s disk does not expose that *A* has a relationship with any of *D*'s relationships because the farside destination is only exposed in memory. Likewise an attack on *D*'s disk does not expose that *B* has any relationships with *A* because the farside destination is only exposed in memory.

This approach is fully zerotrust because all table lookups by *C* and *D* are signed at rest so an attacker can't misdirect traffic between the two or between their associated edge relationshps.

We can conclude that we can have secure authenticity, confidentiality, and privacy (identifier correlation), as well as DDOS protection. The only OOB setups are one time setups between *A* and *B*, *A* and *C*, and *B* and *C*. Indeed if there is trust anchor that provides reputation to intermediaries then there would be no need for *A* and *B* to provide letters of reference to their own indermediary for the other's intermediary.



### Multi-sig Extension

When any of the AIDS in the protocols above are thresholded multi-sig which means the set of controlling key-pairs for the AID is more than one. Then the basic protocol templates need to be modified.  For signing this modification is that at least one signature from the set of controlling keys must be attached or the message is dropped. In order to indicate which key was used to generate any signature, and indexed signature is used. This is special encoding that embeds the index in the derivation code for the signature. The edge may then collect in escrow single-signed messages, where each is signed by at least one of the keys, until the threshold is reached or drop the message if escrow times out before the threshold is reached. Any intermediary may do the collection and escrow before passing along the message. This enables the intermediary to load balance and reduces the memory and processing requirements of the edge.

For encryption, each key in a multisig provides an unique encryption destination. This means that for each of the message examples where there is an encryption destination with decryption key, that the ciphertext must appear as a list of ciphertexts each with a destination that correponds to one of the multi-sig destination AIDs controlling keys. This may require two destinations, one that is the AID in order to look up the key state and the second an index into the key list in order to look up the specific key used to decrypt. Using the index is a much more compact approach and separately listing each destination key. 


## Appendix DDOS

### DDOS Protection Efficiency

DDOS protection mechanisms can be compared using what may be called a DDOS protection efficiency metric. In general DDOS protection efficiency is increased by being able to identify as far upstream as possible any information about a given packet or connection that allows the protection mechanism to drop the packet. The further down stream the packet goes before that decision can be made the less effecient the DDOS protection mechanism. For example, if a DDOS protection mechanism has to first decrypt a packet in order to verify a signature on that same packet before it can make the decision to drop the packet, and given that decryption and signature verification have roughly the same processing load, then the DDOS protection efficiency is half what it would be for a packet where the decision to drop only requires a signature verification. If the protocol has nested (encryption signature, encryption signature ...) layers that must all be processed before the decision to drop may be made then the DDOS protection efficiency is multiplicatiivly reduced by the addition of each nested layer.  This is true no matter where those nestings occur in the processing chain. It doesn't matter whether the nested processing occurs only at the final processor or is divided across a series of upstream processors. The DDOS efficiency of the overall system is exponentially reduced by each nested layer. So multiplying nesting layers directly impacts DDOS efficiency. This is a harsh trade-off that should be considered when designing protocols.

Drop decisions that require intimate knowledge of identifiers besides the IP packet headers usually have significantly reduced protection efficiency. This is because the first line of DDOS defense are firmware load balancers that can make drop decisions merely on the IP source in combination with the protocol type. The next line of defense are software load balancers like nginx that can make more complicated drop decision by looking into the packet. But the protection efficiency of a software load balancer may be 1/10 or less than the protection efficiency of a firmware load balancer. The third line of defense is the edge host itself running on a server. This host may be able to understand the packet in more detail. But the protection efficiency of the edge host may 1/10 or less that of a software load balancer. As a result any protocol that can only make a drop decision at the edge host may have a protection efficiency that is 1/100 of one that can make a drop decision at a firmware load balancer. Any protocol that requires a decryption in order to make a drop decision can only make that drop decision at the edge host because only the edge host has the decryption key. Sharing thost keys with upstream load balancers break confidentiality and exposes the protocol to much worse attacks than mere DDOS. But drop decisions that require a signature verification given a public key can be performed upstream of an edge host (in a software load balancer like nginx) with a corresponed multiplicative increase in protection efficiency.

This same analysis applies in general to overall system DDOS efficiency. If any intermediary can make a drop decision because it has a whitelist of other intermediaries from which it will accept packets then it only has to process a signature verification. If instead the intermediary has to first decrypt before it can use its whitelist to verify it already has 1/2 the DDOS efficiency and if each of N intermediaries in the path of the packet has 1/2 the efficiency for the same reason, then in comparison the DDOS efficiency is 1/2N versus a protocol that can drop based on a signature without decrypting first. So if no decrypt must be done in order to drop (as in at end-only-viewabilty) then we can have very efficient DDOS protection.

This does not mean that protocols can't have nested encryptions. The DDOS protection efficiency is a ratio of malicious packets that a DDOS adversary can throw at a given endpoint that can be dropped without degrading the endpoint vs the number of honest packets that the endpoint can sustain absent a DDOS attack. So if an endpoint can sustan X honest packets per second and can also drop N*X packets per second then the DDOS protection efficiency is N times the sustained throughput. 

### Security-by-obscurity

In protocol design for security with respect to DDOS protection, we also have to be careful not to make the mistake of security-by-obscurity. Any endpoint of value will eventually be found out because its value provides incentives to search for that endpoint. The mere fact that a potential DDOSer does not easily know up front what services to DDOS attack has nothing to do with DDOS protection efficiecy or DDOS vulnerability. It has to do with DDOS susceptbility which protection is at best ephemeral analogous to security-by-obscurity. 

For example, mix networks by design make it difficult for a load balancer to correlate malicious packets to a given IP source. So interposing mixers as intermediaries usually reduces significantly the DDOS protection efficiently of the edges of that network. Mix networks are also vulnerable to what are called sleeper attacks where a malicious attacker can correlate across mixers and discover the eventual endpoint. Once they have performed this correlation, the adversary can DDOS the endpoint directly thereby avoiding any load balancing protection provided by the mixer. Unless of course the mixer has a whitelist that allows it to drop any unpermissioned injection of traffic by a sleeper attacker.


### Amplification Attacks

Of particular concern are protocols that have an amplification attack vulnerability. Any exponential reduction in DDOS efficiency in any protocol stack becomes a juicy target for an amplification attack. The attacker constructs malicious packets that amplify the resources needed to make the drop decision. Amplification attack vulnerabilities are the sign of a poorly designed protocol. The less processing that must be done in order to make a drop decision the better. Any processing that is amplified by maliciously crafted packets is disproportiantely vulnerable to DDOS attack.

For example, A DDOS amplification attack vulnerability makes WCC VCs based on json-ld RDF with RDF integrity proof problematic, especially at scale. One can easily construct a malicious JSON-LD VC that triggers an NP hard amplification of the time to verify the JSON-LD signature integrity proof (when using URDNA-2015) on the receivers end. This is because blank nodes can induce recursive loops that are of order (NP) hard. Its trivial to construct a malicious json-ld document that induces these loops. The receiver can't make a drop decision until after is has transformed the json-ld into an RDF graph and then computed an integrity proof on that graph. Any malformed packet that exploits any extra processing not merely in the RDF integrity proof but the RDF expansion including any @context or @vocab substitutions make the reciever disproportionately vulnerable to DDOS amplification attacks. A mitigation would be to wrap any json-ld based VCs in an external proof (such as an ACDC container) of the over-the-wire packet with a drop decision based on verifying the external proof (container) such as a simple signature against a white list of approved senders. The packet gets dropped before the container is opened.

One of the business models for DDOS attackers is DDOS ransomware. According to CloudFlare 16% of DDoS attacks are ransomware. Thus an attacker need only mount a short term attack to prove a vulnerability that a more intensive attack could exploit in order to monetise via ransomware.


## Appendix: Using Crypto in Ways that is Too-Clever-By-Half


***[too clever by half](https://en.wiktionary.org/wiki/too_clever_by_half)***:   
idiomatic, of a person, plan, theory, etc. Shrewd but flawed by overthinking or excessive complexity, with a resulting tendency to be unreliable or unsuccessful.

One of the repeated mistakes that protocol designers make is in thinking that they can mix and match cryptographic constructs like encryption and signatures in clever ways in order to solve all of the different variations of higher level communication tasks. The ESSR framework basically tells us that there is really only one secure approach to mixing encryption with signatures. Any other approach exposes vulnerabilities that either have been or will be exploited given sufficient incentive and time. 

Higher level communications tasks may require repeated application of the ESSR construction in combination with OOB setups and intermediaries in order to achieve the desired goals.  The combined effect of multiple out-of-band setups with in-band mechanisms may be required to provide the richness essential to some tasks without exposing them to vulnerabilities that make the protocol useless once attacked. Unfortunately, these combined schemes may become too-complex with too much friction to be widely adoptable so it may be a problematic trade-off. There may not be a viable solution in general for all desired communication tasks especially for privacy protection against state actors who have too many attack vectors vis-a-vis practically adoptable protection mechanisms.  



