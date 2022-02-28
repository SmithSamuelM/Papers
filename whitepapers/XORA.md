---
tags: XORA, KERI, ACDC, Accumulator
email: sam@samuelsmith.org
version: 0.04
---

# XORA (Exclusive Or Accumulator)

[![hackmd-github-sync-badge](https://hackmd.io/ZjNWbybtRom4wGfkwyTyYg/badge)](https://hackmd.io/ZjNWbybtRom4wGfkwyTyYg)



## XOR and OTP for Blinding Digests


### Information Theoretic Security and Perfect Security

The highest level of crypto-graphic security with respect to a cryptographic secret (seed, salt, or private key) is called  *information-theoretic security* [[1]]. A cryptosystem that has this level of security cannot be broken algorithmically even if the adversary has nearly unlimited computing power including quantum computing. It must be broken by brute force if at all. Brute force means that in order to guarantee success the adversary must search every combination of key or seed. A special case of *information-theoretic security* is called *perfect-security* [[1]].  *Perfect-security* means that the cipher text provides no information about the key. There are two well-known cryptosystems that exhibit *perfect security*. The first is a *one-time-pad* (OTP) or Vernum Cipher [[2]][[3]], the other is *secret splitting* [[4]], a type of secret sharing [[5]] that uses the same technique as a *one-time-pad*. 

### Cryptographic Strength 

For crypto-systems with *perfect-security*, the critical design parameter is the number of bits of entropy needed to resist any practical brute force attack. In other words, when a large random or pseudo-random number from a cryptographic strength pseudo-random number generator (CSPRNG) [[6]] expressed as string of characters is used as a seed or private key to a cryptosystem with *perfect-security*, the critical design parameter is determined by the amount of random entropy in that string needed to withstand a brute force attack. Any subsequent cryptographic operations must preserve that minimum level of cryptographic strength. In information theory [[7]] the entropy of a message or string of characters is measured in bits. Another way of saying this is that the degree of randomness of a string of characters can be measured by the number of bits of entropy in that string.  Assuming conventional non-quantum computers, the convention wisdom is that, for systems with information theoretic or perfect security, the seed/key needs to have on the order of 128 bits (16 bytes, 32 hex characters) of entropy to practically withstand any brute force attack. A cryptographic quality random or presudo-random number expressed as a string of characters will have essentially as many bits of entropy as the number of bits in the number. For other crypto-systems such as digital signatures that do not have perfect security the size of the seed/key may need to be much larger than 128 bits in order to maintain 128 bits of cryptographic strength.

An N-bit long base-2 random number has 2<sup>N</sup> different possible values. Given that with perfect security no other information is available to an attacker, the attacker may need to try every possible value before finding the correct one. Thus the number of attempts that the attacker would have to try may be as much as 2<sup>N-1</sup>.  Given available computing power, one can easily show that 128 is a large enough N to make brute force attack computationally infeasible.  

Let's suppose that the adversary has access to supercomputers. Current supercomputers can perform on the order of one quadrillion operations per second. Individual CPU cores can only perform about 4 billion operatons per second but a supercomputer will employ many cores in parallel. A quadrillion is approximately 2<sup>50</sup> = 1,125,899,906,842,624. Suppose somehow an adversary had control over one million (2<sup>20</sup> = 1,048,576) super computers which could be employed in parallel to mount a brute force attack. The adversary could then try 2<sup>50</sup> * 2<sup>20</sup> = 2<sup>70</sup> values per second (assuming very conservatively that each try only took one operation).
There are about 3600 * 24 * 365 = 313,536,000 = 2<sup>log<sub>2</sub>313536000</sup>=2<sup>24.91</sup> ~= 2<sup>25</sup> seconds in a year. Thus this set of a million super computers could try 2<sup>50+20+25</sup> = 2<sup>95</sup> values per year. For a 128-bit random number this means that the adversary would need on the order of 2<sup>128-95</sup> = 2<sup>33</sup> = 8,589,934,592 years to find the right value. This assumes that the value of breaking the cryptosystem is worth the expense of that much computing power. Consequently, a cryptosystem with perfect security and 128 bits of cryptographic strength is computationally infeasible to break via brute force attack.


### Blinding with a One-Time-Pad

As mentioned previously, the *one-time-pad* (OTP) [[2]][[3]] may exhibit perfect security. The OTP is a venerable cyphersystem that has the advantage that it can be used manually without a computer. Basically a long string of random or pseudo-random characters forms the *pad*. Someone can use the pad to encrypt a plain-text message. The procedure is to combine each plain-text character in order with the corresponding character from the pad. The combination is typically performed using modulo N addition of the two characters. The simplest case is when  N = 2 which is equivalent to the  bitwise exclusive OR (XOR). The bitwise XOR is equivalent to modulo 2 addition. Because characters from the pad may only be used once, the pad must be at least as long as the plain-text message.  The one time use of a random string of characters from the pad is what gives the system its perfect security property. If two parties wish to exchange multiple messages, then the pad must be at least as long as the sum of the length of all the messages. The main disadvantage of an OTP is that the two parties must each obtain a copy of the same *pad*. This is not a problem for secure backup of a given entity's secrets because the OTP does not need to be shared with any other entity. This is also not necessarily a problem for blinding secrets with respect to third parties because un-blinding operations usually require sharing a secret between the co-parties (but not external third parties). 

Suppose for example, a OTP is used to blind the digest of a document such as an ACDC (a type of verifiable credential). Given that the adversary does not have access to the OTP then the blinding encryption has perfect secrecy which means that the only viable attack is via brute force. If the blinding pseudo random string has at least 128 bits of entropy then brute force attack is computationally infeasible. Consequently a OTP encrypted digest could be safely stored in a public database. 

## Properties of the XOR

As mentioned above, one way to define the bitwise XOR (exclusive OR) is as bitwise addition modulo 2. For simplicity, unless otherwise explicitly defined, we use the `+` symbol to represent the bitwise XOR. The XOR is shown in the following truth table.

|x|y|x + y|
|---|---|:-:|
|0|0|0|
|0|1|1|
|1|0|1|
|1|1|1|

XOR is commutative and associative with the following properties:
`x + x = 0`
`x + 1 = not x`
`x + 0 = x`

Let `X` and `Y` be  binary strings of `N` bits. Then to compute `X + Y` we perform a bitwise XOR on each pair of bits in each bit position of `X` and `Y`.

The most relevant property we want to note is that addition modulo 2 and subtraction modulo 2 produce the same result. Therefore XOR provides both addition and subtraction.

Thus `X + X = 0`.

Let `Z = X + Y ` , then `X = Z + Y` and `Y = Z + X`.

Suppose we have a set of `N` binary strings, indexed by `i` that is `[B0, B1, ... B(N-1)]`, we can "accumulate" them into a single value `A` by XORing them together as follows:

```A = B1 + B2 + B3 + ... + B(N-1)```

We can "remove" any `Bi` from A by "subtracting" it, i.e. XORing it again to `A`

```A' = A + Bi```

We can add it back by XORing it yet again to `A'`

```A = A' + Bi```

If we let the variable `A` be the current value of the accumulated `Bi`s then we can think of it as a compact way of storing the set of `Bi`s that are members of the set accumulated into the `A`.  We now have everything we need to construct a XORA, an XORed Accumulator [[9]]. 

This is exactly the procedure for secret splitting [[4]] mentioned above. The splits are each of the indexed strings `Bi`. The secret is the accumulated value `A`.  In other words `A` is a XORA.



## XORA as a Cryptographic Accumulator

In its simplest form, a Cryptographic Accumulator [[8]] is a one way hash function for membership. An accumulator enables users to proof that a given value is a member of a certain set without revealing all the individual values of all the other members of the set. A hash function maps data of arbitrary size to fixed sized values. A cryptographic digest is an example of a hash function. A set of digests that is aggregated or combined or accumulated into a single value that is same size as each of the individual digests is in the form of a cryptographic accumulator. If there is a way to prove that a given digest is a member of that set represented by that accumulator without revealing each and everyone of the other members then that aggregated combination (accumulation) could be used as a cryptographic accumulator. There are several types of cryptographic accumulators with additional features beyond those provided by a basic one way hash membership function. 

Given that we use XOR as the one way hash function we have a very simple membership proof.

Let

``` A = Sum(Bi) all i```

Let 

``` Rj = Sum(Bi) j≠i```

Then

``` A = Rj + Bj```

The limitation is that the pair `Rj` and `Bj` is not unique. It may be possible to find some other pair `Sj` and `Cj` where `Cj` is not an element of `{Bi}` such that 

```A = Sj + Cj```

Given that `A` is public, an adversary could therefore forge a proof unless we also require that the prover commit to a set of proofs derived from the set `{Bi}`. A proover may then confidentially and privately commit to, and disclose to a proovee (verifier) both `Rj` and `Bj`  without disclosing any other `Bi` or `Ri` using non-repudiable digital signatures on `Rj` and `Bj`. The proovee may then verify that `Bj` is a member of `A` using `Rj` but without knowing any other `Bi` `j≠i`. Because the proof also requires cryptographic commitments by the proover via digital signatures, a forger must also forge signatures not merely find any set of pairings that produce `A`. The only public information is `A`. The members of `A`, i.e. the `Bi` are blinded by each other with perfect security. Should one of the `Bi`, say `B0` have at least 128 bits of entropy then it serves as a OTP to encrypt all the other accumulated `Bi`. This makes a brute force attack on `A` to reveal the `Bi`  computationally infeasible.  The attacker must attack the signatures instead. 

The main limitation of this type of proof is that the provee must have some basis for trusting that the proover is consistent or more specifically non-duplicitous in its signed commitments. One basis for trust would be recourse against the proover when the proover is evidently duplicitous. A non-repudiable digital signature means that the proover cannot subsequently repudiate their commitment made with that signature and any mutually inconsistent signatures provide provable duplicity and may therefore may be used by the proovee to hold the proovee (verifier) accountable, thus providing a basis for trust. The proofs must be duplicity evident.


Given we have a system that provides provable duplicity detection (i.e. is duplicity evident), we can use the cryptographic accumulator signed inclusion proof mechanism described above to construct privacy preserving selective disclosure mechanisms for ACDCs such as hidden attributes or selectively disclosed attributes and/or a revocation registry of one-time-use ACDCs [[9]].


## Confidentiality and Privacy Preserving Disclosure Mechanisms for ACDCs

An authentic chained data container (ACDC) [[10]][[11]] is an IETF [[25]] internet draft focused specification being incubated at the ToIP (Trust over IP) foundation [[12]][[13]]. An ACDC is a variant of the W3C Verifiable Credential (VC) specification [[24]]. Some of the major distinquishing features of ACDCs are a well defined security model based on the IETF focused family of KERI related standards, normative support for chaining, use of JSON schema, multiple serialization formats, and support for Ricardian contracts. One of the new features of ACDC is support for hidden attributes that enable private disclosure under a non-disclosure agreement as part of a chain-link confidentiality protection mechanism.

### Hidden Attribute ACDC









## References

[1]. Information-Theoretic and Perfect Security.  

[1]: https://en.wikipedia.org/wiki/Information-theoretic_security  

[2]. One-Time-Pad.

[2]: https://en.wikipedia.org/wiki/One-time_pad

[3]. Vernom Cipher (OTP).

[3]: https://www.ciphermachinesandcryptology.com/en/onetimepad.htm

[4]. Secret Splitting.  

[4]: https://www.ciphermachinesandcryptology.com/en/secretsplitting.htm

[5]. Secret Sharing.  

[5]: https://en.wikipedia.org/wiki/Secret_sharing  

[6]. Cryptographically-secure pseudorandom number generator (CSPRNG).  

[6]: https://en.wikipedia.org/wiki/Cryptographically-secure_pseudorandom_number_generator  

[7]. Information Theory.  

[7]: https://en.wikipedia.org/wiki/Information_theory  

[8]. Cryptographic Accumulator.  

[8]: https://en.wikipedia.org/wiki/Accumulator_(cryptography)  

[9]. XORA (XORed Accumulator)

[9]: https://github.com/SmithSamuelM/Papers/blob/master/whitepapers/XORA.md

[10]. IETF ACDC (Authentic Chained Data Containers) Internet Draft.

[10]: https://github.com/trustoverip/tswg-acdc-specification

[11]. Authentic Chained Data Containers (ACDC White Paper.  

[11]: https://github.com/SmithSamuelM/Papers/blob/master/whitepapers/ACDC.web.pdf

[12]. Trust Over IP (ToIP) Foundation.  

[12]: https://trustoverip.org  

[13]. ACDC (Authentic Chained Data Container) Task Force.  

[13]: https://wiki.trustoverip.org/display/HOME/ACDC+%28Authentic+Chained+Data+Container%29+Task+Force  

[14]. Smith, S. M., "Key Event Receipt Infrastructure (KERI) Design"  

[14]: https://github.com/SmithSamuelM/Papers/blob/master/whitepapers/KERI_WP_2.x.web.pdf  

[15]. IETF KERI (Key Event Receipt Infrastructure) Internet Draft.  

[15]: https://github.com/WebOfTrust/ietf-keri  

[16]. IETF CESR (Composable Event Streaming Representation) Internet Draft.  

[16]: https://github.com/WebOfTrust/ietf-cesr  

[17]. IETF SAID (Self-Addressing IDentifier) Internet Draft.  

[17]: https://github.com/WebOfTrust/ietf-said  

[18]. IETF PTEL (Public Transaction Event Log) Internet Draft.  

[18]: https://github.com/WebOfTrust/ietf-ptel

[19]. IETF CESR-Proof Internet Draft.  

[19]: https://github.com/WebOfTrust/ietf-cesr-proof  

[20]. IPEX (Issuance and Presentation EXchange) Draft.  

[20]: https://github.com/WebOfTrust/keripy/blob/master/ref/Peer2PeerCredentials.md

[21]. IETF DID-KERI Internet Draft.  

[21]: https://github.com/WebOfTrust/ietf-did-keri  

[22]. IETF (Internet Engineering Task Force).

[22]: https://www.ietf.org  

[23]. GLEIF vLEI.

[23]: https://github.com/WebOfTrust/vLEI

[23]. GLEIF with KERI Architecture.  

[23]: https://github.com/SmithSamuelM/Papers/blob/master/presentations/GLEIF_with_KERI.web.pdf  

[24]. W3C Decentralized Identifiers (DIDs) v1.0.  

[24]: https://w3c-ccg.github.io/did-spec/

[25]. W3C Verifiable Credentials Data Model v1.1.  

[25]: https://www.w3.org/TR/vc-data-model/  





