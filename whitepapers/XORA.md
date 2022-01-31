---
tags: XORA, KERI, ACDC, Accumulator
email: sam@samuelsmith.org
version: 1.00
---

# XORA (Exclusive Or Accumulator)

[![hackmd-github-sync-badge](https://hackmd.io/ZjNWbybtRom4wGfkwyTyYg/badge)](https://hackmd.io/ZjNWbybtRom4wGfkwyTyYg)



## XOR and OTP for Blinding Digests


### Information Theoretic Security and Perfect Security

The highest level of crypto-graphic security with respect to a cryptographic secret (seed, salt, or private key) is called  [information-theoretic security](https://en.wikipedia.org/wiki/Information-theoretic_security). A cryptosystem that has this level of security cannot be broken algorithmically even if the adversary has nearly unlimited computing power including quantum computing. It must be broken by brute force if at all. Brute force means that in order to guarantee success the adversary must search every combination of key or seed. A special case of information-theoretic security is called [perfect security](https://en.wikipedia.org/wiki/Information-theoretic_security). Perfect security means that the cipher text provides no information about the key. There are two well-known cryptosystems that can exhibit perfect security. One is [*secret sharing or splitting*](https://en.wikipedia.org/wiki/Secret_sharing) (see also [ss](http://users.telenet.be/d.rijmenants/en/secretsplitting.htm)). The other is a [*one-time pad*](https://en.wikipedia.org/wiki/One-time_pad) (see also [otp](http://users.telenet.be/d.rijmenants/en/onetimepad.htm). 

### Sufficient Cryptographic Strength to Withstand a Brute-force Attack

For cryptosystems with perfect security, the fundamental parameter is the number of bits of entropy needed to resist any practical brute force attack. In other words, when a large random number is used as a seed/key to a cryptosystem that has perfect security, the question to be answered is how large does the random number need to be to withstand a brute force attack? In Shannon information theory the entropy of a message is measured in bits. The randomness of a number or message can measured by the number of bits of entropy in the number. A cryptographic quality random number will have as many bits of entropy as the number of bits in the number. Assuming conventional non-quantum computers, the convention wisdom is that, for systems with information theoretic or perfect security, the seed/key needs to have on the order of 128 bits (16 bytes) to practically withstand any brute force attack. For other cryptosystems that do not have perfect security the size of the seed/key may need to be much larger.

An N-bit long base-2 random number has 2<sup>N</sup> different possible values. Given that with perfect security no other information is available to an attacker, the attacker may need to try every possible value before finding the correct one. Thus the number of attempts that the attacker would have to test may be as much as 2<sup>N-1</sup>.  Given available computing power, one can estimate if 128 is a large enought N to make brute force attack impractical.  

Let's suppose that the adversary has access to supercomputers. Current supercomputers can perform on the order of one quadrillion operations per second. Individual CPU cores can only perform about 4 billion operatons per second but a supercomputer will employ many cores in parallel. A quadrillion is approximately 2<sup>50</sup> = 1,125,899,906,842,624. Suppose somehow an adversary had a million (2<sup>20</sup> = 1,048,576) super computers to employ in parallel. The adversary could then try 2<sup>50</sup> * 2<sup>20</sup> = 2<sup>70</sup> values per second (assuming that each try only took one operation).
There are about 3600 * 24 * 365 = 313,536,000 = 2<sup>log<sub>2</sub>313536000</sup>=2<sup>24.91</sup> ~= 2<sup>25</sup> seconds in a year. Thus this set of a million super computers could try 2<sup>50+20+25</sup> = 2<sup>95</sup> values per year. For a 128-bit random number this means that the adversary would need on the order of 2<sup>128-95</sup> = 2<sup>33</sup> = 8,589,934,592 years to find the right value. This assumes that the value of breaking the cryptosystem is worth the expense of that much computing power. Consequently, a cryptosystem with perfect security and 128 bits of cryptographic strength is practically impossible to break.


### One-time Pad

As mentioned previously, the [*one-time pad* (OTP)](https://en.wikipedia.org/wiki/One-time_pad) (see also [OTP](http://users.telenet.be/d.rijmenants/en/onetimepad.htm)) may exhibit perfect security. The OTP is a venerable cyphersystem that has the advantage that it can be used manually without a computer. Basically a long string of random characters forms the *pad*. Someone can use the pad to encrypt a plain-text message. The procedure is to combine each plain-text character in order with the corresponding character from the pad. The combination is typically performed using modulo N addition of the two characters. The simplest case is when  N = 2 which is equivalent to the  bitwise XOR (e.g. XOR is modulo 2 addition). Because characters from the pad may only be used once, the pad must be at least as long as the plain-text message.  The one time use of a random string of characters from the pad is what gives the system its perfect security property. If two parties wish to exchange multiple messages, then the pad must be at least as long as the sum of the length of all the messages. The main disadvantage of a one-time pad is that the two parties must each obtain a copy of the same *pad*. This is not necessarly a problem for blinding secrets with respect to third parties because unblinding operations usually involve sharing a secret.

Suppose for example, a OTP is used to blind the a digest of a document (such as an ACDC or VC). Given that the adversary does not have access to the OTP then the blinding encryption has perfect secrecy which means that the only viable attack is via brute force. If the blinding pseudo random string has at least 128 bits of entropy then brute force attack is practicaly infeasible. Consequently the OTP encrypted digest history could be safely stored in a public database. 

## Properties of the XOR

One way to define the XOR (exclusive OR) is as a bitwise addition modulo 2. For simplicity, unless otherwise explicity defined, we use the `+` to represent the XOR. The XOR is shown in the following truth table.

|x|y|x + y|
|---|---|:-:|
|0|0|0|
|0|1|1|
|1|0|1|
|1|1|1|

XOR is commutative and associative.
We also have:
`x + x = 0`
`x + 1 = not x`
`x + 0 = x`

Let `X` and `Y` be  binary strings of `N` bits. Then to compute `X + Y` we perform a bitwise XOR on each pair of bits in each bit position of `X` and `Y`.

The most relevant property we want to note is that addition modulo 2 and subtraction modulo 2 produce the same result. Therefore XOR is both addition and subtraction.

Thus `X + X = 0`
Let `Z = X + Y ` , then `X = Z + Y` and `Y = Z + X`.

Suppose we have a set of `N` binary strings, indexed by `i` that is `[B0, B1, ... B(N-1)]`, we can "accumulate" them into a single value `A` by xoring them together as follows:

```A = B1 + B2 + B3 + ... + B(N-1)```

We can "remove" any `Bi` from A by "subtracting" it, i.e. xoring it again to `A`

```A' = A + Bi```

We can add it back by xoring it again to `A'`

```A = A' + Bi```

If we let the variable `A` be the current value of the accumulated `Bi`s then we can think of it as a compact way of storing the set of `Bi`s that are members of the set accumulated into the `A`

We now have everything we need to construct an XORA (XOR Accumulator)



## XORA (XORed Cryptographic Accumulator)

In its simplest form a [Cryptographic Accumulator](https://en.wikipedia.org/wiki/Accumulator_(cryptography)) is a one way hash function for membership. An accumulator enables users to proof that a given value is a member of a certain set without revealing the individual values of the other members of the set. A hash function maps data of arbitrary size to fixed sized values. A cryptographic digest is an example of a hash function. A set of digests that is combined to a single value that is same size as any individual digest would be of the form of a cryptographic accumulator. If there is a way to prove that a given digest is a member of that set represented by that accumulator without revealing each and everyone of the other members then that combination could be used as a cryptographic accumulator. There are several types of cryptographic accumulators with additional features beyond the basic one way hash membership function. 

Given that we use XOR as the one way hash function we have a very simple membership proof.

Let

``` A = Sum(Bi) all i```

Let 

``` Rj = Sum(Bi) j≠i```

Then

``` A = Rj + Bj```

So if proover privately commits to both `Rj` and `Bj` and `A` is public then proovee can verify that `Bj` is a member of `A` without proover disclosing the other members of `A` to proovee.


We can use this to construct a revocation registry of one time use verifiable credentials.
