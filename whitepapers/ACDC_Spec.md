---
tags: ACDC, XORA, KERI, Selective Disclosure
email: sam@samuelsmith.org
version: 0.1.2
---

[![hackmd-github-sync-badge](https://hackmd.io/Ml5eciWhT2K7mUEIAGO9Xw/badge)](https://hackmd.io/Ml5eciWhT2K7mUEIAGO9Xw)


# ACDC Specification

## Introduction

An authentic chained data container (ACDC) [[10]][[11]] is an IETF [[25]] internet draft focused specification being incubated at the ToIP (Trust over IP) foundation [[12]][[13]]. A major use case for the ACDC specification is to provide GLEIF vLEIs (verifiable Legal Entity Identifiers) [[23]]. GLEIF is the Global Legal Entity Identifier Foundation [[24]]. An ACDC is a variant of the W3C Verifiable Credential (VC) specification [[26]]. The VC specification depends on the W3C DID (Decentralized IDentifier) specification [[25]]. ACDCs are dependent on a suite of related IETF focused standards associated with the KERI (Key Event Receipt Infrastructure) [[14]][[15]] specification. These include CESR [[16]], SAID [[17]], PTEL [[18]], CESR-Proof [[19]], IPEX [[20]], and did:keri [[21]]. Some of the major distinguishing features of ACDCs include  normative support for chaining, use of composable JSON Schema [[38]][[39]], multiple serialization formats, compact formats, support for Ricardian contracts [[40]], and a well defined security model derived from KERI [[14]][[15]].


## ACDC Fields

An ACDC may be abstractly modeled as a nested key:value mapping or dictionary. A key in this context is a field label. To avoid confusion with cryptographic keys we use the term field to refer to mapping pair and the terms field label and field value, (label, value) for element of the pair instead of the more common (key, value). Each nesting is composed of a set of fields or (label, value) pairs. We call this nested set of fields a nested map and when represented by a block delimited serialization such as JSON [[37]], it may be called a block. ACDCs support multiple serialization formats but for the sake of simplicity we will only use JSON here [[37]]. The basic set of field labels is defined in the following table.


| Label |Title|Description|
|:-:|:--|:--|
|v| Version String| Regexable format: ACDCvvSSSShhhhhh_ that provides protocol type, version, serialization type, size and terminator. | 
|d| SAID | Self-referential fully qualified cryptographic digest of enclosing map. |
|i| Identifier (AID)| Semantics determined by context of enclosing map. | 
|u| UUID | Random Universally Unique IDentifier as fully qualified high entropy pseudo-random string, a salted nonce. |
|ri| Registry Identifier (AID) | Issuance and/or revocation, transfer, or retraction registry for ACDC. | 
|s| Schema| May be a block or SAID of schema block expressed as JSON schema. | 
|a| Attribute| May be a block or SAID of a block that provides the data being disclosed or presented as either an attestation or authorization. | 
|e| Edge| May be a block or SAID of block that provides chained provenance by connecting this ACDC node to other ACDC nodes via directed edges as a fragment of a distributed labeled property graph.| 
|n| Node| SAID of another ACDC as terminating point of a directed edge  that connects the encapsulating ACDC to the  specified ACDC node as a fragment of a distributed labeled property graph.| 
|r| Rule | May be a block or SAID of a block that provides contractual  restrictions, terms-of-use, consent, waivers, or a chain-link confidentiality agreement under which disclosure is made. | 

### Compact Labels

The primary fields labels are compact in that they use only one or two characters . ACDCs are meant to support constrained resource applications such as supply chain or IoT in general and therefore use compact labels. The over-the-wire verifiable signed serialization is compact but a more verbose one-to-one semantic overlay may be applied to the compact labels after verification. 

### SAID Fields

Several fields may have either a SAID for a value or a nested map of fields. These are the `s`, `a`, `e`, and `r` fields. Using a SAID enables a more compact representation of any ACDC. The SAIDs follows the SAID protocol [[17]]. Each SAID is a self-referential cryptographic digest of the contents of an associated serialized map or block. Each SAID provides a stable universal cryptographically verifiable reference to its enclosing block. This includes the whole ACDC given by the top level `d` field as well as the SAIDs of any nested sections. The exploded blocks may be attached or cached to optimize bandwidth and availability without decreasing security. 

SAIDs are essentially cryptographic digests (hashes). Given that both the digest and signature operations have sufficient cryptographic strength including collision resistance [[32]][[42]], then a verifiable cryptographic non-repudiable commitment made by a digital signature of that SAID is equivalent to a verifiable non-repudiable commitment made by a digital signature of the full serialization of the associated block from which the SAID was derived. This enables reasoning about ACDCs in a fully interoperable, verifiable, compact and secure manner. This also supports the well-known bow-tie model of Ricardian Contracts [[40]]


### Autonomic IDentifiers

Some fields, such as the `i` and `ri` fields, MUST each have an AID (Autonomic IDentifier) as its value. An AID is fully qualified Self-Certifying IDentifier (SCID) that follows the KERI protocol [[14]][[15]]. A SCID is derived from one or more (public, private) key pairs using asymmetric or public key cryptography to create verifiable digital signatures. Each AID has a set of one or more controllers who each control a private key. By virtue of their private key(s), the set of controllers may make statements on behalf of the associated AID that are backed by uniquely verifiable commitments via digital signatures on those statements. Any entity may then verify those signatures using the associated set of public keys. No shared or trusted relationship between the controllers and verifiers is required. The verifiable key state for AIDs is established with the KERI protocol [[14]][[15]]. The use of AIDS enables ACDCs to be used in a portable but securely attributable, fully decentralized manner in an ecosystem that spans trust domains. Because KERI is agnostic about the namespace for any particular AID, different namespace standards may be used to express KERI AIDs within AID fields in an ACDC. The examples below use the W3C DID namespace specification with the `did:keri` method [[21]]. But the examples would have the same validity from a KERI perspecitive with only the bare fully qualified AID identifier prefix without the encapsulating did:keri namespacing.

An `i` field MUST appear at the top level of an ACDC. The `i` field value is always the AID of the *Issuer* of the ACDC. The *Issuer* is the author or originator of the ACDC. This is the AID to which secure attribution may be made as the source of the ACDC.



## Composable JSON Schema for Selective Disclosure

Notable is the fact that there are no top level type fields in an ACDC. This because the schema, `s` field is the type field. ACDCs follow the design principle of separation of concerns between type information  about a container's payload and the actual payload information in the container. Type information is metadata not data. Composable and conditional JSON Schema allow a validator to answer complex questions about the type of even optional payload elements without mixing type fields into a payload data structure. 

One of the most important features of ACDCs is support for Chain-Link Confidentiality[[41]]. This provides a powerful mechanism for protecting against un-permissiond exploitation of the data disclosed via an ACDC. Essentially an exchange of information compatible with chain-link confidentiality starts with an offer by the discloser of confidention information to be disclosed. This offer includes sufficient partial disclosure about the information to be disclosed (metadata) including the terms of use of the disclosure such that the disclosee can agree to those terms. Once the disclosee has accepted the terms then full disclosure is made. We call the full disclosure that happens after contracrtual acceptance of the terms of use, permissioned disclosure.

Composable JSON Schema [[38]][[39]], enable use of the same base schema to provide validation of the partial disclosure of the offer prior to contract acceptance, and validation of full or detailed disclosure after contract acceptance. A cryptographic committment to the base schema securely specifies the allowable semantics. Decomposition of the base schema enables a validator to impose more specific semantics at later stages of the exchange process. Specifically, the `oneOf` sub-schema composition operator validates against both the compact SAID of the block and the full block. Decomposing the schema to remove the optional compact form enable a validator to ensure complaint full disclosure. To clarify, a validator can confirm schema compliance both pre and post detailed disclosure by using a composed base schema pre-disclosure and a decomposed more specific schema post-disclosure. These features provide a mechanism for secure schema validated contractually bound selective disclosure of confidential data via ACDCs. 


### Selective Disclosure Facility

A `u` field may optionally appear in any block at any level of an ACDC. Whenever a block in an ACDC includes a `u` field it makes that block potentially securely selectively disclosable notwithstanding disclosure of the associated schema of the block. The purpose of the `u` field is to provide sufficient entropy to the SAID of the associated block to make brute force attack on the SAID computationally infeasble. The `u` field may considered a salty nonce [[27]]. Without the entropy provided the `u` field, an adversary may be able to reconstruct the block contents merely from the SAID of the block and the schema of the block using a rainbow or dictionary attack on the set of field values allowed by the schema [[28]][[29]]. The effective entropy of the schema compliant field values may be much less than the strength of the SAID digest. Another way of saying this is that the cardinality of the power set of all combinations of allowed field values may be much less than the cryptographic strength of the SAID. Thus an adversary could successfully brute force discover the exact block by creating digests of all the elements of the power set which if small enough may be computationally feasible. The entropy in the `u` field ensures that the cardinality of the power set allowed by the schema is at least as big as the entropy of the SAID.

When a `u` field appears at the top level of an ACDC then the whole ACDC itself, not merely some block within the ACDC, may be disclosed in a privacy preserving (correlation minimizing) manner such as a one-time use ACDC. 

## ACDC Variants

There are several variants of ACDCs determined by the presence/absence of certain fields and the value of those fields. 


At the top level, the presence (absence) absence  of the `u` field produces two variants private (public) and when present an empty `u` enables a private meta-data only variant

### Public ACDC

Given that there is no `u` field at the top level of an ACDC, then knowledge of both the the SAID, `d`, field at the top level of an ACDC and the schema of the ACDC may enable discovery of the remaining contents of the ACDC via a rainbow table attack [[28]][[29]]. Therefore the `d` field at the top level, although, a cryptographic digest, does not securely blind the contents of the ACDC given knowledge of the schema.  Therefore any cryptographic commitments to that `d` field provide a fixed point of correlation potentially to the ACDC field values themselves in spite of non-disclosure of those field values. Thus an ACDC without a `u` field must be considered a public (non-confidential) ACDC.

### Private ACDC

Given that there is `u` field at the top level of an ACDC whose value has sufficient cryptographic entropy, then the SAID, `d`, field at the top level of an ACDC  may provide a secure cryptographic digest of the contents of the ACDC. An adversary when given both the schema of the ACDC and the  SAID, `d` field, is not able to discover the remaining contents of the ACDC in a computationally feasible manner such as a rainbow table attack[[28]][[29]]. Therefore the `u` field (UUID) may be used to securely blind the contents of the ACDC notwithstanding knowledge of the schema and `d` field (SAID).  Moreover a cryptographic commitment to that `d` field (SAID) does not provide a fixed point of correlation to the other ACDC field values themselves unless and until there has been disclosure of those field values. Thus an ACDC with a sufficiently high entropy `u` field may be considered a private or (confidential) ACDC. This allows a commitment to the the SAID of a private ACDC to be made prior to disclosure of the ACDC. It allows a set of multiple copies of essentially the same ACDC that differ only in their SAIDs to be used publicaly in different contexts but without provable public correlation of their contents including their use of a shared registry.

### Metadata ACDC

An empty `u` field appearing at the top level of an ACDC indicates that the ACDC is a metadata ACDC. The purpose of the ACDC is to provide a mechanism so that a presenter can make cryptographic commitments to the metadata of a yet to be disclosed private ACDC without providing any point of correlation with the SAID, `d` field of that yet to be disclosed ACDC. The SAID, `d` field, of the metadata ACDC is cryptographically derived from an ACDC with an empty `u` field so it will necessarily be different from an ACDC with a high entropy `u` field. But the presenter may make a non-repudiable cryptographic commitment to the metadata SAID to initiate a chain-link confidentiality exchange without leaking correlation to the actual ACDC to be disclosed [[41]]. A verifier may validate the metadata in the metadata ACDC before agreeing to any restrictions imposed by the future disclosure. The metadata includes the issuer, the schema, the provenance edges, and rules. The attributes field of a metadata ACDC may be empty so that its value is not correlatable across presentations. Should the verifier refuse to agree to the rules then the presenter has not leaked the SAID of the actual ACDC or the SAID of the attributes block that would have been presented. The verifier is able to verify the issuer, the schema, the provenaced edges, and rules prior to agreeing to the rules.  Similarly an Issuer of an ACDC may use a metadata ACDC to initiate a contractual waiver with an Issuee prior to issuance. Should the Issuee refuse the waiver then the Issuer has not leaked the SAID of the actual ACDC that would have been issued nor the SAID of its attributes block.

When a metadata ACDC is presented only the presenter's signatures are attached not the Issuer's signatures. This precludes the Issuer's signatures from being used as a point of correlation. The issuer's signatures are only disclosed to the verifier after the verifier has agreed to keep them confidential. The verifier is still protected because ultimately its validation will fail if the presenter does not eventually provide verifiable Issuer signatures. But should the verifier not agree to the terms of the disclosure expressed in the rules section the Issuer signatures are not leaked by the presenter.

## Unpermissioned Exploitation

The primary goal is to protect against unpermissioned exploitation of data. The primary goal is not privacy per se but privacy may be a mechanism to protect against unpermissioned exploitation of data.
There are two primary mechanisms we may use to protect against unpermissioned exploitation. These are
- Chain-link Confidentiality [[41]]
- Selective Disclosure


### Principle of Least Disclosure

> The system should disclose only the minimum amount of information about a given party needed to facilitate a transaction and no more. [[45]]


### Three Party Exploitation Model

First Party = *Discloser* of data.  
Second Party = *Disclosee* of data received from First Party (*Discloser*).  
Third Party = *Observer* of data disclosed by First Party (*Discloser*)  to Second Party (*Disclosee*).  

#### Second Party (Disclosee) Exploitation
- implicit permissioned correlation.
    - no contractual restrictions on use of disclosed data. 
- explicit permissioned correlation.
    - use as permitted by contract
- explicit unpermissioned correlation with other second parties or third parties.
    - malicious use in violation of contract

#### Third Party (Observer) Exploitation
- implicit permissioned correlation. 
    - no contractual restrictions on use of observed data. 
- explicit unpermissioned correlation via collusion with second parties.
    - malicious use in violation of second party contract

### Chain link confidentiality exchange
- *Discloser* provides non-repudiable *Offer* with verifiable metadata (sufficient partial disclosure) which include any terms or restrictions on use. 
- *Disclosee* verifies *Offer* against composed schema and metadata adherence to desired data.
- *Disclosee* provides non-repudiable *Accept* of terms subject compliant disclosure.
- *Discloser* provides non-repudiable *Disclosure* with sufficient compliant detail.
- *Disclosee* verifies *Disclosure* using decomposed schema and adherence to offered data.

*Disclosee* may now enagage in permissioned use and carries liability as deterrent against unpermissioned use.




## Compact ACDC
A compact ACDC includes only the SAIDs of each top level section.

### Compact Public ACDC
A fully compact public ACDC is shown below. 


``` json
{
  "v":  "ACDC10JSON00011c_",
  "d":  "EBdXt3gIXOf2BBWNHdSXCJnFJL5OuQPyM5K0neuniccM",
  "i":  "did:keri:EmkPreYpZfFk66jpf3uFv7vklXKhzBrAqjsKAn2EDIPM",
  "ri": "did:keri:EymRy7xMwsxUelUauaXtMxTfPAMPAI6FkekwlOjkggt",
  "s":  "E46jrVPTzlSkUPqGGeIZ8a8FWS7a6s4reAXRZOkogZ2A",
  "a":  "EgveY4-9XgOcLxUderzwLIr9Bf7V_NHwY1lkFrn9y2PY",
  "e":  "ERH3dCdoFOLe71iheqcywJcnjtJtQIYPvAu6DZIl3MOA",
  "r":  "Ee71iheqcywJcnjtJtQIYPvAu6DZIl3MORH3dCdoFOLB",
}

```

### Compact Private ACDC
A fully compact private ACDC is shown below. 


``` json
{
  "v":  "ACDC10JSON00011c_",
  "d":  "EBdXt3gIXOf2BBWNHdSXCJnFJL5OuQPyM5K0neuniccM",
  "u":  "0ANghkDaG7OY1wjaDAE0qHcg",
  "i":  "did:keri:EmkPreYpZfFk66jpf3uFv7vklXKhzBrAqjsKAn2EDIPM",
  "ri": "did:keri:EymRy7xMwsxUelUauaXtMxTfPAMPAI6FkekwlOjkggt",
  "s":  "E46jrVPTzlSkUPqGGeIZ8a8FWS7a6s4reAXRZOkogZ2A",
  "a":  "EgveY4-9XgOcLxUderzwLIr9Bf7V_NHwY1lkFrn9y2PY",
  "e":  "ERH3dCdoFOLe71iheqcywJcnjtJtQIYPvAu6DZIl3MOA",
  "r":  "Ee71iheqcywJcnjtJtQIYPvAu6DZIl3MORH3dCdoFOLB",
}

```


## Attribute Section

In the examples above the attribute section has been compacted into merely the SAID of that section. 

Suppose that the un-compacted value of the attribute section as denoted by the `a` field is as follows:

``` json
"a":
  {
    "d": "EgveY4-9XgOcLxUderzwLIr9Bf7V_NHwY1lkFrn9y2PY",
    "i": "did:keri:EpZfFk66jpf3uFv7vklXKhzBrAqjsKAn2EDIPmkPreYA",
    "score": 96,
    "name": "Jane Doe"
  }

```

The `d` field at the top level of the attribute block is the SAID of that block and is the same SAID used as the compacted value of the `a` field. 


The combinations of the presence (absence) of an `i` field in the attribute section produce two more variants of ACDCs.


### Targeted ACDC

When present, the value of the `i` field at the top level of an attribute block is the AID of the *Issuee* of the ACDC. This *Issuee* is provably controllable identifier that is the *Target*. This enables such targeted ACDCs to be used for many different purposes such as authorization or delegation directed at the *Target* AID, i.e. the *Issuee*. In other words this form of an ACDC, by virtue of the attribute block top level `i` field  provides a container for authentic data that may also be used as some form of credential that is verifiably bound to the *Issuee* as *Target* by the *Issuer* and that by virtue of provable control over the *Issuee* AID may also be verifiably presented by the controller of the *Issuee*.

The definition of the term *credential*: *is evidence of authority, status, rights, entitlement to privileges, or the like*. To elaborate, the presence of an `i` field at the top level of the attributes block enables the ACDC to be used as a verifiable credential given by the *Issuer* to the *Issuee*. 

One reason the *Issuee* is nested into the `a` block is to enable the *Issuee* to be private. The *Issuee* may also be called the *Holder* of the ACDC. In other verifiable credential parlance the *Issuee* is called the *Subject*. But here we use the more semantically precise albeit less common terms of *Issuer* and *Issuee*. The ACDC is issued from or by an *Issuer* and is issued to an *Issuee*. This terminology does not bias or color the role (function) an *Issuee* plays in the use of an ACDC. But what an *Issuee* does provide is a mechanism for control of the subsequent use of the ACDC once it has been issued. To elaborate, because the `i` field is an AID, there is a provable controller of that AID and therefore that controller may make non-repudiable commitments via digital signatures on behalf of that AID.  The subsequent use of the ACDC may be securely attributed to the Issuee.

Importantly the presence of an `i` field enables the associated *Issuee* to make authoritative verifiable presentations of the ACDC. A designated *Issuee*  also better enables initiation of presentation exchanges of the ACDC between that Issuee and a verifier.

In addition, because the *Issuee* is a specified counter-party the *Issuer* may engage in a contract with the *Issuee* as a pre-condition to issuance and impose liability waivers or other terms of use on that *Issuee* that the *Issuee* agrees too by virtue of a non-repudiable signature on the ACDC at the time of issuance. 

Likewise, the presence of an *Issuee*, `i` field, enables the *Issuer* to use the ACDC as a contractual vehicle for conveying an authorization to the *Issuee*.  This enables verifiable delegation chains of authority because an *Issuee* in one ACDC may be the *Issuer* of some other ACDC. Thereby an *Issuer* may delegate authority to an *Issuee* who may then become a verifiably authorized *Issuer* that then delegates that authority (or an attenuation of that authority) to some other *Issuee* and so forth.  



### Untargeted ACDC

Consider the case where the `i` field is absent at the top level of the attributes block as shown below:

``` json
"a":
  {
    "d": "EgveY4-9XgOcLxUderzwLIr9Bf7V_NHwY1lkFrn9y2PY",
    "temp": 45,
    "lat": "N40.3433", 
    "lon:: "W111.7208"
  }

```

This ACDC has an *Issuer* but no *Issuee*. There is no provably controllable *Target*. This may be though of as an undirected verifiable attestation or observation of the data in the attributes block by the *Issuer*. One could say that the attestation is addressed to "whom it may concern". An untargeted ACDC enables verifiable authorship by the Issuer of the data in the attributes block but there is no specified counter-party and no verifiable mechanism for delegation of authority.  Consequently the rules section may only provide contractual obligations with implied counter-parties

This form of an ACDC provides a container for authentic data only (not specified authorizations). But authentic data is still a very important use case. An untargeted ACDC enables verifiable authorship of data. An observer such as a sensor that controls an AID may make verifiable non-repudiable measurements and publish them as ACDCs. These may be chained together to provide provenance for or a chain-of-custody of any data.  These ACDCs could be used to provide a verifiable data supply chain for any compliance regulated application. This provides a way to protect participants in a supply chains from imposters. Such data supply chains are also useful as a verifiable digital twin of a physical supply chain.

A hybrid chain of one or more targeted ACDCs ending in a chain of one or more untargeted ACDCs enables a delegated authorized attestations at the tail of that chain. This may be very useful in many regulated supply chain applications such as verifiable authorized authentic data sheets for a given pharmaceutical.


### Public-Attributes ACDC

Consider the following uncompacted attributes block:

``` json
"a":
  {
    "d": "EgveY4-9XgOcLxUderzwLIr9Bf7V_NHwY1lkFrn9y2PY",
    "i": "did:keri:EpZfFk66jpf3uFv7vklXKhzBrAqjsKAn2EDIPmkPreYA",
    "score": 96,
    "name": "Jane Doe"
  }

```

Given the absence of a `u` field at the top level of the attributes block, then knowledge of both SAID, `d`, field at the top level of an attributes block and the schema of the attributes block may enable discovery of the remaining contents of the attributes block via a rainbow table attack [[28]][[29]]. Therefore the `d` field of the attributes block, although, a cryptographic digest, does not securely blind the contents of the attributes block given knowledge of the schema. It only provides compactness not privacy. Moreover any cryptographic commitments to that `d` field provide a fixed point of correlation potentially to the attribute block field values themselves in spite of non-disclosure of those field values via a compact ACDC. Thus an ACDC without a `u` field in its attributes block must be considered a public-attribute ACDC even when expressed in compact form.



### Private-Attributes ACDC

Consider the following form of an uncompacted attributes block:

``` json
"a":
{
  "d": "EgveY4-9XgOcLxUderzwLIr9Bf7V_NHwY1lkFrn9y2PY",
  "u": "0AwjaDAE0qHcgNghkDaG7OY1",
  "i": "did:keri:EpZfFk66jpf3uFv7vklXKhzBrAqjsKAn2EDIPmkPreYA",
  "score": 96,
  "name": "Jane Doe"
}
```

Given presence of a `u` field at the top level of the attributes block and whose value has sufficient cryptographic entropy, then the SAID, `d`, field at the top level of the attributes block  may provide a secure cryptographic digest of the contents of the attributes block. An adversary when given both the schema of the attributes block and its  SAID, `d` field, is not able to discover the remaining contents of the attributes block in a computationally feasible manner such as a rainbow table attack [[28]][[29]].  Therefore the `u` field (UUID) of the attributes block in a compact ACDC enables the `d` field to securely blind the contents of the attributes block notwithstanding knowledge of the schema and the `d` field (SAID) of that attributes block.  Moreover a cryptographic commitment to that `d` field does not provide a fixed point of correlation to the attribute field values themselves unless and until there has been disclosure of those field values. 

To elaborate, when an ACDC includes a sufficiently high entropy `u` field at the top level of its attributes block then the ACDC may be considered a private-attributes ACDC when expressed in compact form, that is, the attributes block is represented by its SAID or `d` field and the value of the `a` field is the value of the nested `d` field from the uncompacted attributes block. The compact form of the ACDC may be presented and committed too without leaking details of the attributes. Later disclosure of the uncompacted attributes block may be verified against the SAID of the attributes provided in the compact form.

Because the *Issuee* AID is nested in the attributes as its top level `i` field, a presentation exchange could be initiated on behalf of a different AID that has not yet been correlated to the *Issuee* AID and then only correlated to the Issuee AID after the verifier has agreed to the chain-link confidentiality provisions in the rules section of the private-attributes ACDC [[41]].


### Selective Disclosure ACDC

Consider the following uncompacted attribute block:

``` json
"a":
[
  {
    "d": "ErzwLIr9Bf7V_NHwY1lkFrn9y2PYgveY4-9XgOcLxUde",
    "u": "0AqHcgNghkDaG7OY1wjaDAE0",
    "i": "did:keri:EpZfFk66jpf3uFv7vklXKhzBrAqjsKAn2EDIPmkPreYA"
  },
  {
    "d": "ELIr9Bf7V_NHwY1lkgveY4-Frn9y2PY9XgOcLxUderzw",
    "u": "0AG7OY1wjaDAE0qHcgNghkDa",
    "score": 96
  },
  {
    "d": "E9XgOcLxUderzwLIr9Bf7V_NHwY1lkFrn9y2PYgveY4-",
    "u": "0AghkDaG7OY1wjaDAE0qHcgN",
    "name": "Jane Doe"
  },
  {
    "d": "ErzwLIr9Bf7V_NHwY1lkFrn9y2PYgveY4-9XgOcLxUde",
    "u": "0AOY1wjaDAE0qHcgNghkDaG7"
  }
]
```

All the attributes are provided as an array of blocks. Each attribute has its own dedicated block with its own SAID, `d`, field and `u` field in addition to  the attribute field itself. Notice the presence of an Issuer attribute block, thus making this a targeted selective disclosure ACDC. An untargeted selective disclosure ACDC would be missing the *Issuer* attribute block. The last attribute block is a dummy attribute whose only purpose is to blind the other attributes in the array. The dummy attribute block is never disclosed. This is so that even when all but one (non-dummy) attribute is disclosed in a given presentation or a set of presentations then that one remaining undisclosed attribute is still blinded. 

Assuming each attribute block has its own `u` field with sufficient cryptographic entropy, then the SAID, `d`, field of that specific attribute block  may provide a secure cryptographic digest of the contents of its attribute block. An adversary despite being given both the schema of the attribute block and its  SAID, `d` field, is not able to discover the remaining contents of the attribute block in a computationally feasible manner such as a rainbow table attack [[28]][[29]].  Therefore the `u` field (UUID) of each attribute block enables the associated `d` field to securely blind the block's contents notwithstanding knowledge of the block's schema and its `d` field (SAID).  Moreover a cryptographic commitment to that `d` field does not provide a fixed point of correlation to the attribute field values themselves unless and until there has been disclosure of those field values. 

The SAIDs, `d` fields, of each of the attribute blocks in the attributes array are combined into an XOR accumulator (XORA) by a bit-wise XOR (addition mod 2). The value of the `a` field in compact form is the value of this accumulator. This differs from the other compact forms where the value of the `a` field is a SAID. To clarify, in a selective disclosure ACDC, when in compact form, instead of SAID, the value of the `a` field is an accumlator (XORA) of the SAIDs of each of the individual attribute blocks from its attributes array.

Using zero based indexing for each element of the attributes arrays, let `ai` represent the SAID of attribute at index `i`. Then for the example above, `a0` denotes the SAID of the *Issuee* attribute, `a1` denotes the SAID of the *score* attribute,  `a2` denotes the SAID of the *name* attribute, and `a3` denotes the SAID of the dummy attribute. To restate, the value of the `a` field in compact form is a XORA. Let `A` denote this value which is given by the following expression:

``` A = a0 + a1 + a2 + a3  = sum(ai for all i)```

where `+` is bitwise XOR (bitwise addition mod 2) .

Recall the definition of an XORA proof of inclusion. We recast it here in terms of the SAIDs of the blocks.

Let

``` A = sum(ai) for all i``` where `A` is the accumulator and sum is the bitwise XOR (bitwise addition mod 2).

Let 

``` Rj = sum(ai) all i, i≠j``` where `Rj` is the remains of `aj` with respect to `A`.

Then

``` A = Rj + aj```

For each `j` the Issuer creates a signature  `sj` on `aj` and a signature `Sj` on `Rj`. A non-repudiable proof of inclusion of `aj` in `A` consists of the tuple 

```(aj, sj, Rj, Sj)```.

A verifier validates that `A = aj + Rj` and verifies the signatures of the Issuer `sj` on `aj` and `Sj` on `Rj`. The full proof of inclusion is only disclosed to the verifier after the verifier has agreed to the terms of the rules section. Therefore, in the event the verifier declines to accept the terms of disclosure,  then a presentation of the compact version of the ACDC does not provide any point of correlation to any of the attributes in the accumulator `A`. The attributes are hidden inside `A`.


To protect against later forgery via a birthday attack against the accumulator `A` given  a later compromise of the signing keys of the Issuer [[30]][[31]], the issuer also anchors an inclusion proof digest seal in the same event as the the ACDC issuance seal. Because this requires the forger to publish in the KEL of the issuer an inclusion proof digest seal with the forged inclusion proofs for the accumulator `A` in the same event as the issuance seal for the ACDC that contains `A`, then any later attempt at forgery using compromised keys is thereby detectable as duplicity. 

There are two potential forms for this inclusion proof digest seal. The simplest form but with more verbose verification is a digest seal of a concatentation of inclusion proof digests. More complex but will less verbose verification is a Merkle tree root seal of the set of inclusion proof digests.

The value of the simple form of the inclusion proof digest seal is the cryptographic digest of the string of concatenated strings of the digests of the concatenated elements of each inclusion proof tuple in order.  This digest is given by 

``` D = H(C(H(C(ai, si, Ri, Si)) for all i)) ``` 

where `H` is a prefix digest operator and  `C` is a prefix concatenation operator.

Let `hi` represent `H(ai || si || Ri || Si)`  where `||` is the infix concatenation operator. Then the expression above can be simplified as:

``` D = H(C( hi for all i)) ``` 


To clarify, given the example selective disclosure ACDC attributes array above and using `||` to represent the infix concatenation operation, the inclusion proof digest seal digest is calculated as follows:

```
D = H( h0 || h1 || h2 || h3)
```


Given sufficient collision resistance of the digest and signatures, such a digest is not subject to a birthday attack because it is the digest of an ordered concatenation not an XOR [[30]][[31]][[32]][[42]].

In addition to the inclusion proof tuple for an attribute, the presenter also provides proof to the verifier that the signatures in the inclusion proof tuple contributed to the inclusion proof digest seal. This may be done by providing the full list of digests of concatenated signatures of all the inclusion proof tuples. For the example above the list is given by:

``` [h0, h1, h2, h3] ```


From this list the verifier can compute the value of the inclusion proof digest seal  and verify that the disclosed inclusion proof tuple matches one of the `hi` from the list and therefore that it contributed to the inclusion proof seal digest. This makes any attempt at forgery duplicity evident.

Similarly the more complex form of the inclusion proof digest seal is a Merkle tree of the inclusion proof tuple digests, `hi`. The Merkle tree needs to have appropriate second-pre-image attack protection of interior branch nodes. The presenter then only needs to provide a subset of digests from the Merkle tree to prove that a given digest, `hj` contributed to the Merkle tree root digest. For ACDCs with a small number of attributes the added complexity of the Merkle tree approach may not be worth the savings in verbosity.

Given sufficient collision resistance of the digest and signatures, either inclusion proof digest seal format (list or Merkle tree) prevents a verifier or any adversary from discovering in a computationally feasible way the values of any undisclosed attribute SAID merely from the combination of `A`, the inclusion proofs of other non-dummy but disclosed attributes and the set of all the inclusion proof digests [[32]][[42]]. A verifier would also need the inclusion proof tuple of the non-disclosed attribute.

A private selective disclosure ACDC provides significant correlation minimization because a presenter may use a metadata ACDC prior to acceptance by the verifier of the terms of chain-link confidentiality [[41]]. Thus only malicious verifiers who violate chain-link confidentiality may correlate between presentations of a given private selective disclosure ACDC. But they are not able to discover any undisclosed attributes.

Furthermore as explained in the XORA section, the four tuple XORA non-repudiable inclusion proof provides support for zero-trust authentic data storage and computing architecures because the pair `(ai, si)` may be stored together on one device to provided zero trust verifiability of the authenticity of that devices data without exposing the full inclusion proof. The signature `Si` on `Ri` may be stored separately on a different device. This prevents a side channel attack on the Issuee's storage device that exposes only the `ai` and `si` from being able to forge alternate non-repudiable inclusion proofs or to discover provable non-repudiable inclusion of the undisclosed fields.

### Hierarchical Derivation at Issuance of Selective Disclosure ACDC

The amount of data transferred between the Issuer and Issuee or other recipient of issuance of a selective disclosure ACDC can be minimized by using a hierarchical deterministic derivation function to derive the value of the `u` fields from a shared secret salt [[27]]. The Issuer may use a Diffie-Hellman key exchange to share a secret salt with the Issuee [[43]][[44]]. The Issuer also provides a template of the ACDC but with empty `u` (UUID) and `d` (SAID) fields for each block with such fields. Each `u` field value is then derived from the shared salt with a path prefix that indexes a given a given block. Given the `u` field (UUID) value, the `d` field (SAID) may then be derived and both full and compact versions of the ACDC may then be generated. An example path could be "0" for the top level 'u' field (for private ACDCS) and "0/0" for the zeroth indexed attribute in the attributes array. Likewise "0/1" for the next attributed and so on. 

In addition to the shared salt and ACDC template, the Issuer also provides the list of inclusion proof tuples, a signature on the SAID of the compact ACDC and references to the anchoring seals. Everything else an Issuee needs to make a verifiable presentation can be calculated by the Issuee. 


## Selective Disclosure via Bulk Issuance of Private ACDCs

A private ACDC may be issued in bulk as a set. The only difference between each ACDC is the SAID. The point of bulk issuance is to minimize the data transfer at issuance and the storage requirements for a set of copies of essentially the same ACDC. Essentially each copy of a bulk issued ACDC shares a template that both the Issuer and Issuee use to generate a given ACDC in the set without requiring that the Issuer and Issuee exchange and store each member of the set independently. This minimizes the data transfer and storage requirements for the Issuer and Issuee.

The purpose of bulk issuance is to more efficiently enable the Issuee to use unique ACDC SAIDs to isolate and thereby minimize correlation across different usage contexts of essentially the same ACDC while allowing public commitments to the ACDC SAIDs.

For example an ACDC provenance chain is connected via references to the SAIDs given by the top level `d` fields, of the ACDCs in that chain.  A given ACDC thereby makes commitments to other ACDCs. Expressed another way, an ACDC may be a node in a directed graph of ACDCs. Each directed edge in that graph emanating from one ACDC includes a reference to the SAID of some other connected ACDC. These edges provide points of correlation to an ACDC via their SAID reference. Using private bulk issued ACDCs allows the Issuee to better control the correlatability of those presentations using different presentation strategies.  
For example, the Issuee could use one copy of a bulk issued private ACDC per presentation even to the same verifier. This strategy would consume the most copies. It is essentially a one-time-use ACDC strategy. Alternatively, the Issuee could use the same copy for all presentations to the same verifier and thereby only permit the verifier to correlate between presentations it received directly but not to other verifiers. This limits the consumption to one copy per verifier. In yet another alternative, the Issuee could use the one copy for all presentations in given context with a group of verifiers that thereby only permits correlation among that group. 

In this context we are talking about permissioned correlation. Any verifier that has received a full presentation of a private ACDC has access to all the fields disclosed by the presentation but the terms of the chain-link confidentiality agreement may forbid sharing those field values outside a given context. Thus an Issuee may use a combination of bulk issued ACDCs with chain-link confidentiality to control permissioned correlation of the contents of an ACDC while allowing the SAID of the ACDC to be more public. The SAID of a private ACDC does not expose the ACDC contents to an un-permissioned third party. Unique SAIDs belonging to bulk issued ACDCs prevent third parties from making provable correlation between ACDCs via their SAIDs in spite of those SAIDs being public. This does not stop malicious verifiers (as second parties) from colluding and correlating against the disclosed fields but it does limit provable correlation to the information disclosed to a given group of malicious colluding verifiers. To restate unique SAIDs per copy of a set of private bulk issued ACDC prevent un-permissioned third parties from making provable correlation in spite of those SAIDs being public unless they collude with malicious verifiers (second parties).


In some applications, chain-link-confidentiality is insufficient to deter un-permissioned correlation. Some verifiers may be malicious with sufficient malicious incentives to overcome whatever counter incentives the terms of the contractual chain-link confidentiality may impose. In these cases more aggressive technological anti-correlation mechanisms may be useful. To elaborate, in spite of the the fact that chain-link confidentiality terms of use may forbid such malicious correlation, making such correlation more difficult technically may provide better protection than chain-link confidentiality alone [[41]].

It is important to note that any group of colluding malicious verifiers may always make statistical correlation between presentations despite technical barriers to cryptographically provable correlation. In general there is no cryptographic mechanism that precludes statistical correlation among a set of colluding verifiers because they may make cryptographically unverifiable or unprovable assertions about information presented to them that may be proven as likely true using merely statistical correlation techniques.


### Basic Bulk Issuance

To elaborate, bulk issuance is provided via a set of copies of essentially the same ACDC which only differ in the top level `u` field (UUID) and `d` field (SAID) values. The Issuer and Issuee employ a hierarchical deterministic derivation function from a shared secret salt to generate the `u` fields (UUIDs) of each copy. This is analogous to that described in the section for selective disclosure ACDCs. To recapitulate, the Issuer shares a secret salt with the Issuee (such as with a Diffie-Hellman key exchange [[43]][[44]]). The Issuer also shares a template of the ACDC but with empty top level `u` (UUID) and `d` (SAID) field values. The salt is used with a deterministic index to generate unique `u` field (UUID) field values for each copy of the ACDC. Given the `u` field (UUID) value the associated `d` field (SAID) of the ACDC may be generated from the template. An example deterministic derivation path would be to use the copy index such as "0", "1", "2" etc.  

This approach can be extended to enable bulk issued selective disclosure ACDCs by using a hierarchical derivation path for the `u` field values. For example, the path "1/2" may be used to generate the UUID of attribute index 2 at ACDC index 1.

In addition to the shared salt and ACDC template, the Issuer also provides a list of signatures of SAIDs, one for each SAID of each copy of the associated compact ACDC.  The Issuee at the time of presentation can generate from the template and the salt the full ACDC along with the non-repudiable siganture. The Issuee does not need to store a copy of each bulk issued ACDC, merely the template, the salt, and the list of signatures. 

The Issuer must also anchor the issuance of the set of bulk issued ACDCs in its KEL via a digest seal. The digest seal makes a cryptographic commitment to each of the members of the bulk issued set of ACDC. This protects against later forgery of ACDCs in the event the Issuer's signing keys become compromised.  A later attempt at forgery requires a new event or new version of an event that includes a new anchoring seal that makes a cryptographic commitment to each of the new forged ACDCs. This new anchoring event is detectable as duplicitous, i.e. it provides evidence of duplicity as required by KERI.

A verbose approach would be to create a separate seal for each member of the bulk issued set where each seal's digest value is cryptographically derived from the SAID of the ACDC. This is the case when an ACDC uses a public TEL in a registry. 

A more compact approach is to use a digest seal whose value is derived not from one member of the bulk issued set but from the whole set. The complications of this approach is that it must be done in sucha as way as to not enable provable correlation by a third party of the SAIDS of the bulk issued set of ACDCs. A naive approach would be to simply used the bare SAIDs of each ACDC, but that would directly provably correlate those SAIDs. Recall that the purpose of bulk issuance is to allow the SAID of an ACDC in a bulk issued set to be used publically without dislosing in an unpermissioned provable way the SAIDs of the other members. 

One solution to use an XORA (accumulator) to generate the value for the digest seal. This is analogous to the selective disclosure mechanism used for the attribute section,  `a` field value in a selective disclosure ACDC. Recall that one advantage of the XORA approach is that the inclusion four tuple better supports zero-trust storage architectures of authentic data. 

The goal is to generate an accumulated value that enables an Issuee to selectively disclose an ACDC in a build issued set without leaking the other members of the set to unpermissioned second or third parties.

Analagously to the selective disclosure XORA, the bulk issued XORA derivation process is described below:

Using zero based indexing for each member of the bulk issued set of ACDCs, let `bi` represent the SAID the ACDC at index `i`. The set may be denoted `{bi} for all i`. The accumulator is generated by combining all the `bi` in `{bi}` via a bitwise XOR (bitwise addition mod 2). Let `B` denote accumulator value which is given by the following expression:

``` b = sum(bi for all i)```

where `sum` is bitwise XOR (bitwise addition mod 2) .

Recall the definition of an XORA proof of inclusion. We recast it here in terms of the SAIDs of the ACDCs.

Let

``` B = sum(bi) for all i``` where `B` is the accumulator and `sum` is the bitwise XOR (bitwise addition mod 2).

Let 

``` Rj = sum(bi) all i, i≠j``` where `Rj` is the remains of `bj` with respect to `B`.

Then

``` B = Rj + bj```

For each `j` the Issuer creates a signature  `sj` on `bj` and a signature `Sj` on `Rj`. A non-repudiable proof of inclusion of `bj` in `B` consists of the tuple 

```(bj, sj, Rj, Sj)```.

A verifier validates that `B = bj + Rj` and verifies the signatures of the Issuer `sj` on `bj` and `Sj` on `Rj`. The full proof of inclusion is only disclosed to the verifier after the verifier has accepted (agreed to) the terms of the rule section. Therefore, in the event the verifier declines to accept the terms of disclosure,  then a presentation of the compact version of the ACDC does not provide any point of correlation to any other SAID of any other ACDC from the builk set in the accumulator `B`. The SAIDs are hidden inside `B`. Indeed if the Issuee uses a metadata version of the ACDC then even its SAID is not disclosed until after acceptance of terms in the rule section.

To protect against later forgery via a birthday attack against the accumulator `D` given  a later compromise of the signing keys of the Issuer [[30]][[31]], the issuer also anchors an inclusion proof digest seal in the same event as the issuance seal for the bulk accumulator. Because this requires the forger to publish in the KEL of the issuer an inclusion proof digest seal with the forged inclusion proofs for the accumulator `D` in the same event as the issuance seal for the bulk issue accumulator `D`, then any later attempt at forgery using compromised keys is thereby detectable as duplicity. 

The inclusion seal blinds the SAID, `bi` of ACDC member at index `i` with the remains value, `Ri` and the signatures `sj`, and `Si` on the SAID `bi` and remains `Ri` via a hash of the concatenated members of the inclusion proof four tuple. Only permissioned verifiers (second parties) recieve all members of the inclusion proof tuple for given ACDC SAID. Therefore an un-permission third party who has knowledge of a given SAID, `bi` of an ACDC in the bulk issued set is unable to provable correlate that SAID to any other members with only the inclusion proof seal digest. Likwise any group of permissioned verifiers (second parties) can only provably correlate the SAIDs of those ACDCs that have been disclosed to that group. A third party correlator must collude with malcious second parties to provably correlate any SAIDs and the extent of provable correlation is limited to the subset of ACDCs SAIDs disclosed to the group of colluding malicious verifiers (second parties)

There are two potential forms for this inclusion proof digest seal. The simplest form but with more verbose verification is a digest seal of a concatentation of inclusion proof digests. More complex but will less verbose verification is a Merkle tree root seal of the set of inclusion proof digests.

The value of the simple form of the inclusion proof digest seal is the cryptographic digest of the string of concatenated strings of the digests of the concatenated elements of each inclusion proof tuple in order.  This digest is given by 

``` D = H(C(H(C(bi, si, Ri, Si)) for all i)) ``` 

where `H` is a prefix digest operator and  `C` is a prefix concatenation operator.

Let `hi` represent `H(bi || si || Ri || Si)`  where `||` is the infix concatenation operator. Then the expression above can be simplified as:

``` D = H(C( hi for all i)) ``` 

Given sufficient collision resistance of the digest and signatures, such a digest is not subject to a birthday attack because it is the digest of an ordered concatenation not an XOR [[30]][[31]][[32]][[42]].

In addition to the inclusion proof tuple for an attribute, the presenter also provides proof to the verifier that the signatures in the inclusion proof tuple contributed to the inclusion proof digest seal. This may be done by providing the full list of digests of concatenated signatures of all the inclusion proof tuples such as,

``` [h0, h1, h2, ...] ```


From this list the verifier can compute the value of the inclusion proof digest seal  and verify that the disclosed inclusion proof tuple matches one of the `hi` from the list and therefore that it contributed to the inclusion proof seal digest. This makes any attempt at forgery duplicity evident.

Similarly the more complex form of the inclusion proof digest seal is to build a Merkle tree of the inclusion proof tuple digest, `hi`. The Merkle tree needs to have appropriate second-pre-image attack protection of interior branch nodes. The presenter then only needs to provide a subset of digests from the Merkle tree to prove that a given digest, `hj` contributed to the Merkle tree root digest. For ACDCs with a small number of attributes the added complexity of the Merkle tree approach may not be worth the savings in verbosity.

Given sufficient collision resistance of the digest and signatures, either inclusion proof digest seal format (list or Merkle tree) prevents a verifier or any adversary from discovering in a computationally feasible way the values of any undisclosed SAIDs from the bulk set merely from the combination of `B`, the inclusion proofs of other disclosed SAIDS, and the set of all the inclusion proof digests [[32]][[42]]. A verifier would also need the inclusion proof tuple of the non-disclosed SAID.

To reiterate bulk issued private ACDC provides significant correlation minimization because a presenter may use a metadata ACDC prior to acceptance by the verifier of the terms of chain-link confidentiality [[41]]. Thus only malicious verifiers who violate chain-link confidentiality may correlate between presentations of any member of the set of bulk issued ACDCs. But they are not able to discover any undisclosed SAIDs.

Furthermore as explained in the XORA section, the four tuple XORA non-repudiable inclusion proof provides support for zero-trust authentic data storage and computing architecures because the pair `(bi, si)` may be stored together on one device to provided zero trust verifiability of the authenticity of that devices data without exposing the full inclusion proof. The signature `Sj` on `Rj` may be stored separately on a different device. This prevents a side channel attack on the Issuee's storage device that exposes only the `bi` and `si` from being able to forge alternate non-repudiable inclusion proofs or to discover provable non-repudiable inclusion of the undisclosed ACDC SAIDs.


There are use two cases of the accumulator value `B` to consider.
In the first case the set of bulk issued ACDCs does NOT use a public TEL (Transaction Event Log). In the second case the set of bulk issued ACDCs does use a public TEL.

In the first case a digest seal with the accumulator value `B` is directly anchored in the KEL of the Issuer. The inclusion proof digest seal, with `D`, is also anchored in the same event as the accumulator seal.  

In the second case the accumulator value `B` is used as the value identifier of the VC in the inception event of the public TEL in the registry given by the `ri` field. The SAID of the inception event is used to identifer the TEL which is anchored  with a seal in the KEL of the Issuer. The TEL SAID in this case is derived from the accumulator value `B`. The inclusion proof digest seal, with `D`, is also anchored in the same event as the TEL seal.

Recall that the usual purpose of a TEL is to provide a verifiable data registry that enables enables dynamic revocation of an ACDC via a state of the TEL. A verifier checks the state at the time of use to see if the associated ACDC has been revoked. The Issuer is in control of changing the state of the TEL. The registry identifier, `ri` field is used to indicate the public registry which usually has a TEL entry for each ACDC. Typically the identifier of each TEL entry is the SAID of the TEL's inception event which is a digest of the event contents which include the SAID of the ACDC. In the bulk issuance case however, the TEL's inception event contents include the XORA value `B` instead of the SAID of a given ACDC. This allows all members of the bulk issued set to share the same TEL without any third party being able to discover which TEL in an unpermissioned provable way. In order to prove which TEL a specific bulk issued ACDC is using the fully inclusion proof tuple must be disclosed. 

To summarize, the purpose of using a XORA from which the TEL identifier is derived is to enable a set of bulk issued ACDCs to share a single public TEL that provides dynamic revocation but without enabling unpermissioned correlation to any other members of the bulk set by virtue of the shared TEL. This enables the issuance/revocation/transfer state of all copies of set of bulk issued ACDCs to be provided by a single TEL which minimizes the storage and compute requirements on the TELs registry while providing selective disclosure to prevent unpermissioned correlation via the public TEL.  



### Bulk Issuance of Private ACDCs with Unique Issuee AIDs

One potential point of provable but unpermissioned correlation among any group of colluding malicious verifiers is when the same Issuee AID is used for presentation to all verifiers in that group. Recall that the contents of private ACDCs are not disclosed except to permissioned verifiers, thus a common Issuee AID would only be point of correlation for a group of colluding malicious verifiers.

One solution to this problem is for the Issuee to use a unique AID for the copy of a bulk issued ACDC presented to each verifier in a given context. This requires that each ACDC copy in bulk issued set use a unique Issuee AID. This would enable the Issuee in a given context to minimize provable correlation by malicious verifiers against any given Issuee AID. In this case, the bulk issuance process may be augmented to include the derivation of a unique AID for each copy of the ACDC by including in the inception event that defines the Issuee's self-addressing AID, a digest seal derived from the shared salt and copy index. This can be re-generated at time of presentation by the Issuee. Each unique Issuee AID would need its own KEL. But generation and publication of the associated KEL can be delayed until the bulk issued ACDC is actually used. This approach completely isolates a given Issuee AID in a given context with respect to the use of a bulk issued private ACDC to even better protect against un-permissioned correlation by malicious verifiers.


### Independent TEL Bulk Issued ACDCs

In some applications where chain-link confidentiality does not sufficiently deter malicious provable correlation by verifiers, an Issuee may benefit from using ACDC with independent TELs but that are still issued in bulk manner. 

In this case, the bulk issuance process must be augmented so that each uniquely identified copy of the ACDC gets its own TEL in the registry. Each verifier of a full presentation of a given copy of the ACDC only receives proof of one uniquely identified TEL and can NOT provably correlate the TEL state state of one presentation to any other presentation because the ACDC SAID, the TEL identifier, and the signature of the issuer on the SAID of a given copy will all be different for each copy. There is no point of provable correlation permissioned or otherwise.

The obvious drawbacks of this approach (independent unique TELs for each private ACDC) is that the size of the registry database increases as a multiple of the number of copies of each ACDC and every time an Issuer must change TEL state of given set of copies it must change the state of multiple TELs in the registry. This imposes both a storage and computation burden on the registry. The primary advantage of this approach, however, is that each copy of a private ACDC has a uniquely identified TEL. This minimizes un-permissioned third party exploitation via provable correlation of TEL identifiers even with colluding verifiers. They are limited to statistical correlation techniques. 

In this case the set of private ACDCs may or may not share the same Issuee AID because for all intents and purposes each copy appears to be a different ACDC even when issued to the same Issuee. Nonetheless, using unique Issuee AIDs may further reduce correlation by malicious verifiers beyond using independent TELs.

To summarize the main benefit of this approach, in spite of its storage and compute burden is that in some applications chain-link confidentialty does not sufficiently deter unpermission malicious collusion and so completely independent ACDCs must be used.


## Edge Section

In the compact ACDC examples above the edge section has been compacted into merely the SAID of that section. 

Suppose that the un-compacted value of the edge section denoted by the `e` field is as follows:

```
"e": 
    {
      "d": "EerzwLIr9Bf7V_NHwY1lkFrn9y2PgveY4-9XgOcLxUdY",
      "qvi":
      {
        "n": "EIl3MORH3dCdoFOLe71iheqcywJcnjtJtQIYPvAu6DZA"
      }
    }

```

The `d` field at the top level of the edge block is the SAID of that block and is the same SAID used as the compacted value of the `e` field that appears at the top level of the ACDC. Each edge in the edge section gets is own field. Each edge also has a local label. Note that each edge does not include a type field. The type of each edge is provided by the schema vis-a-vis the label of that edge. This is in accordance with the design principle of ACDCs that may be succintly expressed as "the schema is the type". This approach varies somewhat from common practice for labeled property graphs which often do not have schema. Because ACDCs have schema for other reasons, however, they leverage that schema to provide edge types with a cleaner separation of concerns.

Each edge block has one required node field labled `n`. The value of the node, `n` field is the SAID of the ACDC to which the edge connects. When an edge block has only one field, its `n` field then the edge block may be compacted into a single field value as follows. The schema for the edge section will indicate that the edge value, for the edge label, in this case, `qvi`, is a node SAID. This is shown below.

```
"e": 
    {
      "d": "EerzwLIr9Bf7V_NHwY1lkFrn9y2PgveY4-9XgOcLxUdY",
      "qvi": "EIl3MORH3dCdoFOLe71iheqcywJcnjtJtQIYPvAu6DZA"
    }

```

A labled property graph allows each edge to have its own set of properties. These might include modifiers that influence how the connected node is to be used such as a weight. Weighted directed edges the represent degrees of confidence or liklihood are commonly used for machine learning or reasoning under uncertainty. The following example adds a weight property to the edge block as indicated by the `w` field .

```
"e": 
    {
      "d": "EerzwLIr9Bf7V_NHwY1lkFrn9y2PgveY4-9XgOcLxUdY",
      "qvi":
      {
        "n": "EIl3MORH3dCdoFOLe71iheqcywJcnjtJtQIYPvAu6DZA",
        "w": "high"
        
      }
    }

```

Abstractly an ACDC with one or more edges may be a fragment of a distributed labeled property graph. However the local label does not enable unique global resolution of a given edge with properties other than the node, `n` field, property. To make an edge with additional  properties also globally uniquely resolvable we add a `d` field that is the SAID of the edge block. Because the SAID is a cryptographic digest it will universally and uniquely identify an edge with a given set of properties. This allows ACDCs to be used as secure fragments of a globally distributed labeled property graph. This combines the beneficial features of property graphs and global knowledge graphs in a secure manner that crosses trust domains. This is shown below.


```
"e": 
    {
      "d": "EerzwLIr9Bf7V_NHwY1lkFrn9y2PgveY4-9XgOcLxUdY",
      "qvi":
      {
        "d": "E9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NHwY1lkFrn",
        "n": "EIl3MORH3dCdoFOLe71iheqcywJcnjtJtQIYPvAu6DZA",
        "w": "high"
      }
    }

```

If we want that edge properties to also be blinded by its SAID, i.e. be private then we add a UUID, `u` field. As with the ACDC `u` field, if the field has sufficient entropy then the values of the properties are not discoverable in a computationally feasible manner merely given the SAID, `d` field and schema for the edge. These may be called private edges. This is shown below.

```
"e": 
    {
      "d": "EerzwLIr9Bf7V_NHwY1lkFrn9y2PgveY4-9XgOcLxUdY",
      "qvi":
      {
        "d": "E9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NHwY1lkFrn",
        "u":  "0AG7OY1wjaDAE0qHcgNghkDa",
        "n": "EIl3MORH3dCdoFOLe71iheqcywJcnjtJtQIYPvAu6DZA",
        "w": "high"
      }
    }

```

When a edge points to a private ACDC, an Issuer may choose to use a metadata version of that private ACDC when presenting the, `n`, field of and edge prior to acceptance of the terms of disclosure. The verifier can verify the metadata of the private node without the Issuer exposing that actual node contents via the actual node (ADCD) SAID or other attributes.

Private ACDCs (nodes) and private edges may be used in combination to prevent unpermissioned correlation of the distributed labeled property graph.

In general lookup of a the details of an ACDC reference as a node, `n` field value, in an edge begins with its provided SAID. Because the SAID is cryptographic digest with high collision resistance it provides a universally unique identifier to the referenced ACDC. Discovery of a service endpoint URL that provides database access to a copy of the ACDC may be bootstrapped via an OOBI (Out-Of-Band-Introduction) that links the service endpoint URL to the SAID of the ACDC. Alternatively the issuer may provide as an attachment at issuance a copy of the referenced ACDC. In either case after a successful issuance exchange the Issuee or holder of any ACDC will have either a copy or a means of obtaining a copy of any referenced ACDCs as nodes in the edge sections of all ACDCs so chained. That Issuee or holder will then have everything it needs to make a successful presentation to a verifier. This is the essence of percolated discovery.


## Rule Section

In the compact ACDC examples above the rule section has been compacted into merely the SAID of that section. Suppose that the un-compacted value of the rule section denoted by the `r` field is as follows:

```
"r": 
  {
    "d": EwY1lkFrn9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NA",
    "warrantyDisclaimer": "Issuer provides this credential on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied, including, without limitation, any warranties or conditions of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A PARTICULAR PURPOSE",
    "liabilityDisclaimer": " In no event and under no legal theory, whether in tort (including negligence), contract, or otherwise, unless required by applicable law (such as deliberate and grossly negligent acts) or agreed to in writing, shall the Issuer be liable for damages, including any direct, indirect, special, incidental, or consequential damages of any character arising as a result of this credential. "
}
```

The `d` field at the top level of the rule block is the SAID of that block and is the same SAID used as the compacted value of the `r` field that appears at the top level of the ACDC. Each clause in the rule section gets is own field. Each clause also has its own local label. Note that each clause does not include a type field. The type of each clause is provided by the schema vis-a-vis the label of that clause. This is in accordance with the design principle of ACDCs that may be succintly expressed as "the schema is the type". 

The value of each clause is either the explicit legal language of that clause or may be a block with additional properties. The purpose of the rule section is to provide a Ricardian Contract [[40]]. The important features of a Ricardian contract is that it be both human and machine readable and referenceable by a cryptotographic digest. A JSON encoded document is a practical example of both a human and machine readable document.  The SAID, `d`, field provides the digest.Legal contracts may be hierarchically structured into sections and subsections with named or numbered clauses in each section. The labels on the clauses may follow such a hierarchical structure using nested maps or blocks. When a clause is expressed as the block that also has legal language, the clause language the value of the `c` field. The clause may also have a clause SAID, in its `d` field. Clause specific SAIDs enable reference to individual clauses not merely the whole contract given by the rule section.

An example rule section with blocks for each clause is provided below:


```
"r": 
  {
    "d": "EwY1lkFrn9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NA",
    "warrantyDisclaimer": 
      {
        "d": "EXgOcLxUdYerzwLIr9Bf7V_NAwY1lkFrn9y2PgveY4-9",
        "c": "Issuer provides this credential on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied, including, without limitation, any warranties or conditions of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A PARTICULAR PURPOSE",
      },
    "liabilityDisclaimer": 
      {
        "d": "EY1lkFrn9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NAw",
        "c": " In no event and under no legal theory, whether in tort (including negligence), contract, or otherwise, unless required by applicable law (such as deliberate and grossly negligent acts) or agreed to in writing, shall the Issuer be liable for damages, including any direct, indirect, special, incidental, or consequential damages of any character arising as a result of this credential. "
      }
  }
```

This enables a compact form of a set of clauses where each clause value is the SAID of the corresponding clause. For example:

```
"r": 
  {
    "d": "EwY1lkFrn9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NA",
    "warrantyDisclaimer":  "EXgOcLxUdYerzwLIr9Bf7V_NAwY1lkFrn9y2PgveY4-9",
    "liabilityDisclaimer": "EY1lkFrn9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NAw",
  }
```


When in compact form either for the rule section as a whole or for each clause,
the lookup of the details of associated clause begins with its provided SAID. Because the SAID is cryptographic digest with high collision resistance it provides a universally unique identifier to the referenced clause details. Discovery of a service endpoint URL that provides database access to a copy of the rule section as contract or any of its clauses may be bootstrapped via an OOBI (Out-Of-Band-Introduction) that links the service endpoint URL to the SAID of the contract of clause. Alternatively the issuer may provide as an attachment at issuance a copy of the referenced contract of clause. In either case after a successful issuance exchange the Issuee or holder of any ACDC will have either a copy or a means of obtaining a copy of any referenced contracts or clauses of all ACDCs involved in the presentation. That Issuee or holder will then have everything it needs to make a successful presentation to a verifier. This is the essense of percolated discovery.


## Schema Section

TBD


## Informative Example of ACDC

### Compact Version

``` jsone
{
  "v":  "ACDC10JSON00011c_",
  "d":  "EBdXt3gIXOf2BBWNHdSXCJnFJL5OuQPyM5K0neuniccM",
  "u":  "0ANghkDaG7OY1wjaDAE0qHcg",
  "i":  "did:keri:EmkPreYpZfFk66jpf3uFv7vklXKhzBrAqjsKAn2EDIPM",
  "ri": "did:keri:EymRy7xMwsxUelUauaXtMxTfPAMPAI6FkekwlOjkggt",
  "s":  "E46jrVPTzlSkUPqGGeIZ8a8FWS7a6s4reAXRZOkogZ2A",
  "a":  "EgveY4-9XgOcLxUderzwLIr9Bf7V_NHwY1lkFrn9y2PY",
  "e":  "ERH3dCdoFOLe71iheqcywJcnjtJtQIYPvAu6DZIl3MOA",
  "r":  "Ee71iheqcywJcnjtJtQIYPvAu6DZIl3MORH3dCdoFOLB",
}

```

### Non-compact Version


``` json
{
  "v":  "ACDC10JSON00011c_",
  "d":  "EBdXt3gIXOf2BBWNHdSXCJnFJL5OuQPyM5K0neuniccM",
  "u":  "0ANghkDaG7OY1wjaDAE0qHcg",
  "i":  "did:keri:EmkPreYpZfFk66jpf3uFv7vklXKhzBrAqjsKAn2EDIPM",
  "ri": "did:keri:EymRy7xMwsxUelUauaXtMxTfPAMPAI6FkekwlOjkggt",
  "s":  "E46jrVPTzlSkUPqGGeIZ8a8FWS7a6s4reAXRZOkogZ2A",
  "a":  
  {
    "d": "EgveY4-9XgOcLxUderzwLIr9Bf7V_NHwY1lkFrn9y2PY",
    "u": "0AwjaDAE0qHcgNghkDaG7OY1",
    "i": "did:keri:EpZfFk66jpf3uFv7vklXKhzBrAqjsKAn2EDIPmkPreYA",
    "score": 96,
    "name": "Jane Doe"
  },
  "e": 
  {
    "d": "EerzwLIr9Bf7V_NHwY1lkFrn9y2PgveY4-9XgOcLxUdY",
    "qvi":
    {
      "d": "E9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NHwY1lkFrn",
      "n": "EIl3MORH3dCdoFOLe71iheqcywJcnjtJtQIYPvAu6DZA",
      "w": "high"
    }
  },
  "r": 
  {
    "d": "EwY1lkFrn9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NA",
    "warrantyDisclaimer": 
    {
      "d": "EXgOcLxUdYerzwLIr9Bf7V_NAwY1lkFrn9y2PgveY4-9",
      "c": "Issuer provides this credential on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied, including, without limitation, any warranties or conditions of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A PARTICULAR PURPOSE",
    },
    "liabilityDisclaimer": 
    {
      "d": "EY1lkFrn9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NAw",
      "c": " In no event and under no legal theory, whether in tort (including negligence), contract, or otherwise, unless required by applicable law (such as deliberate and grossly negligent acts) or agreed to in writing, shall the Issuer be liable for damages, including any direct, indirect, special, incidental, or consequential damages of any character arising as a result of this credential. "
    }
  }
}

```

### Non-compact Selctive Disclosure Version


``` json
{
  "v":  "ACDC10JSON00011c_",
  "d":  "EBdXt3gIXOf2BBWNHdSXCJnFJL5OuQPyM5K0neuniccM",
  "u":  "0ANghkDaG7OY1wjaDAE0qHcg",
  "i":  "did:keri:EmkPreYpZfFk66jpf3uFv7vklXKhzBrAqjsKAn2EDIPM",
  "ri": "did:keri:EymRy7xMwsxUelUauaXtMxTfPAMPAI6FkekwlOjkggt",
  "s":  "E46jrVPTzlSkUPqGGeIZ8a8FWS7a6s4reAXRZOkogZ2A",
  "a":
  [
    {
      "d": "ErzwLIr9Bf7V_NHwY1lkFrn9y2PYgveY4-9XgOcLxUde",
      "u": "0AqHcgNghkDaG7OY1wjaDAE0",
      "i": "did:keri:EpZfFk66jpf3uFv7vklXKhzBrAqjsKAn2EDIPmkPreYA"
    },
    {
      "d": "ELIr9Bf7V_NHwY1lkgveY4-Frn9y2PY9XgOcLxUderzw",
      "u": "0AG7OY1wjaDAE0qHcgNghkDa",
      "score": 96
    },
    {
      "d": "E9XgOcLxUderzwLIr9Bf7V_NHwY1lkFrn9y2PYgveY4-",
      "u": "0AghkDaG7OY1wjaDAE0qHcgN",
      "name": "Jane Doe"
    },
    {
      "d": "ErzwLIr9Bf7V_NHwY1lkFrn9y2PYgveY4-9XgOcLxUde",
      "u": "0AOY1wjaDAE0qHcgNghkDaG7"
    }
  ]
  "e": 
  {
    "d": "EerzwLIr9Bf7V_NHwY1lkFrn9y2PgveY4-9XgOcLxUdY",
    "qvi":
    {
      "d": "E9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NHwY1lkFrn",
      "n": "EIl3MORH3dCdoFOLe71iheqcywJcnjtJtQIYPvAu6DZA",
      "w": "high"
    }
  },
  "r": 
  {
    "d": "EwY1lkFrn9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NA",
    "warrantyDisclaimer": 
    {
      "d": "EXgOcLxUdYerzwLIr9Bf7V_NAwY1lkFrn9y2PgveY4-9",
      "c": "Issuer provides this credential on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied, including, without limitation, any warranties or conditions of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A PARTICULAR PURPOSE",
    },
    "liabilityDisclaimer": 
    {
      "d": "EY1lkFrn9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NAw",
      "c": " In no event and under no legal theory, whether in tort (including negligence), contract, or otherwise, unless required by applicable law (such as deliberate and grossly negligent acts) or agreed to in writing, shall the Issuer be liable for damages, including any direct, indirect, special, incidental, or consequential damages of any character arising as a result of this credential. "
    }
  }
}

```


## Appendix: XOR and OTP for Blinding Digests


### Information Theoretic Security and Perfect Security

The highest level of crypto-graphic security with respect to a cryptographic secret (seed, salt, or private key) is called  *information-theoretic security* [[1]]. A cryptosystem that has this level of security cannot be broken algorithmically even if the adversary has nearly unlimited computing power including quantum computing. It must be broken by brute force if at all. Brute force means that in order to guarantee success the adversary must search every combination of key or seed. A special case of *information-theoretic security* is called *perfect-security* [[1]].  *Perfect-security* means that the cipher text provides no information about the key. There are two well-known cryptosystems that exhibit *perfect security*. The first is a *one-time-pad* (OTP) or Vernum Cipher [[2]][[3]], the other is *secret splitting* [[4]], a type of secret sharing [[5]] that uses the same technique as a *one-time-pad*. 

### Cryptographic Strength 

For crypto-systems with *perfect-security*, the critical design parameter is the number of bits of entropy needed to resist any practical brute force attack. In other words, when a large random or pseudo-random number from a cryptographic strength pseudo-random number generator (CSPRNG) [[6]] expressed as string of characters is used as a seed or private key to a cryptosystem with *perfect-security*, the critical design parameter is determined by the amount of random entropy in that string needed to withstand a brute force attack. Any subsequent cryptographic operations must preserve that minimum level of cryptographic strength. In information theory [[7]] the entropy of a message or string of characters is measured in bits. Another way of saying this is that the degree of randomness of a string of characters can be measured by the number of bits of entropy in that string.  Assuming conventional non-quantum computers, the convention wisdom is that, for systems with information theoretic or perfect security, the seed/key needs to have on the order of 128 bits (16 bytes, 32 hex characters) of entropy to practically withstand any brute force attack. A cryptographic quality random or presudo-random number expressed as a string of characters will have essentially as many bits of entropy as the number of bits in the number. For other crypto-systems such as digital signatures that do not have perfect security the size of the seed/key may need to be much larger than 128 bits in order to maintain 128 bits of cryptographic strength.

An N-bit long base-2 random number has 2<sup>N</sup> different possible values. Given that with perfect security no other information is available to an attacker, the attacker may need to try every possible value before finding the correct one. Thus the number of attempts that the attacker would have to try may be as much as 2<sup>N-1</sup>.  Given available computing power, one can easily show that 128 is a large enough N to make brute force attack computationally infeasible.  

Let's suppose that the adversary has access to supercomputers. Current supercomputers can perform on the order of one quadrillion operations per second. Individual CPU cores can only perform about 4 billion operatons per second but a supercomputer will employ many cores in parallel. A quadrillion is approximately 2<sup>50</sup> = 1,125,899,906,842,624. Suppose somehow an adversary had control over one million (2<sup>20</sup> = 1,048,576) super computers which could be employed in parallel to mount a brute force attack. The adversary could then try 2<sup>50</sup> * 2<sup>20</sup> = 2<sup>70</sup> values per second (assuming very conservatively that each try only took one operation).
There are about 3600 * 24 * 365 = 313,536,000 = 2<sup>log<sub>2</sub>313536000</sup>=2<sup>24.91</sup> ~= 2<sup>25</sup> seconds in a year. Thus this set of a million super computers could try 2<sup>50+20+25</sup> = 2<sup>95</sup> values per year. For a 128-bit random number this means that the adversary would need on the order of 2<sup>128-95</sup> = 2<sup>33</sup> = 8,589,934,592 years to find the right value. This assumes that the value of breaking the cryptosystem is worth the expense of that much computing power. Consequently, a cryptosystem with perfect security and 128 bits of cryptographic strength is computationally infeasible to break via brute force attack.


### Blinding with a One-Time-Pad

As mentioned previously, the *one-time-pad* (OTP) [[2]][[3]] may exhibit perfect security. The OTP is a venerable cypher-system that has the advantage that it can be used manually without a computer. Basically a long string of random or pseudo-random characters forms the *pad*. Someone can use the pad to encrypt a plain-text message. The procedure is to combine each plain-text character in order with the corresponding character from the pad. The combination is typically performed using modulo N addition of the two characters. The simplest case is when  N = 2 which is equivalent to the  bit-wise exclusive OR (XOR). The bit-wise XOR is equivalent to modulo 2 addition. Because characters from the pad may only be used once, the pad must be at least as long as the plain-text message.  The one time use of a random string of characters from the pad is what gives the system its perfect security property. If two parties wish to exchange multiple messages, then the pad must be at least as long as the sum of the length of all the messages. The main disadvantage of an OTP is that the two parties must each obtain a copy of the same *pad*. This is not a problem for secure backup of a given entity's secrets because the OTP does not need to be shared with any other entity. This is also not necessarily a problem for blinding secrets with respect to third parties because un-blinding operations usually require sharing a secret between the co-parties (but not external third parties). 

Suppose for example, a OTP is used to blind the digest of a document such as an ACDC (a type of verifiable credential). Given that the adversary does not have access to the OTP then the blinding encryption has perfect secrecy which means that the only viable attack is via brute force. If the blinding pseudo random string has at least 128 bits of entropy then brute force attack is computationally infeasible. Consequently a OTP encrypted digest could be safely stored in a public database. 


## Appendix:  XOR Cryptographic Accumulator (XORA)

### Properties of the XOR

One way to define the bit-wise XOR (exclusive OR) is as bit-wise addition modulo 2. For simplicity, unless otherwise explicitly defined, we use the `+` symbol to represent the bit-wise XOR. The XOR is shown in the following truth table.

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

Let `X` and `Y` be  binary strings of `N` bits. Then to compute `X + Y` we perform a bit-wise XOR on each pair of bits in each bit position of `X` and `Y`.

The most relevant property we want to note is that addition modulo 2 and subtraction modulo 2 produce the same result. Therefore XOR provides both addition and subtraction.

Thus `X + X = 0`.

Let `Z = X + Y ` , then `X = Z + Y` and `Y = Z + X`.

Suppose we have a set of `N` binary strings, indexed by `i` that is `[b0, b1, ... b(N-1)]`, we can "accumulate" them into a single value `A` by XORing them together as follows:

```A = b1 + b2 + b3 + ... + b(N-1)```

We can "remove" any `bi` from A by "subtracting" it, i.e. XORing it again to `A` as follows:

```A' = A + bi```

We can add it back by XORing it yet again to `A'`

```A = A' + bi```

If we let the variable `A` be the current value of the accumulated `bi`s then we can think of it as a compact way of storing the set of `{bi}`s by accumlating them into `A`.  We now have everything we need to construct a XORA, an XORed Accumulator [[9]]. 

This is exactly the procedure for secret splitting [[4]] mentioned above. The splits are each of the indexed strings `bi`. The secret is the accumulated value `A`.  In other words `A` is a XORA.


### XORA Proof of Inclusion

In its simplest form, a Cryptographic Accumulator [[8]] is a one way hash function for membership. An accumulator enables users to proof that a given value is a member of a certain set without revealing all the individual values of all the other members of the set. A hash function maps data of arbitrary size to fixed sized values. A cryptographic digest is an example of a hash function. A set of digests that is aggregated or combined or accumulated into a single value that is same size as each of the individual digests is in the form of a cryptographic accumulator. If there is a way to prove that a given digest is a member of that set represented by that accumulator without revealing each and everyone of the other members then that aggregated combination (accumulation) could be used as a cryptographic accumulator. There are several types of cryptographic accumulators with additional features beyond those provided by a basic one way hash membership function. 

Given that we use an XOR of hashes as the one way membership function we have a very simple membership proof.

Let

``` A = sum(bi) all i``` where `A` is the value of the accumulator and `sum` is bitwise addition mod 2 or bitwise XOR.

Let 

``` Rj = sum(bi) i≠j``` where `Rj` is the remains of `A` with respect to `bj`

Then

``` A = Rj + bj```

A concern is that the pair `Rj` and `bj` is not unique. It may be possible to find some other pair `Xj` and `yj` where `yj` is not an element of the set `{bi}` such that 

```A = Xj + yj```

Given that `A` is public, an adversary could therefore forge a proof using `Xj` and `yj` unless we also require that the prover commit to a set of proofs derived from the set `{bi}`. A prover may then confidentially and privately commit to, and disclose to a provee (verifier) both `bj` and its remains `Rj`  using non-repudiable digital signatures on  `bj` and `Rj` without disclosing any other `bi` or `Ri`. 

Let `sj` be the signature of `bj` and `Sj` be the signature of `Rj` then the tuple `(bj, sj, Rj, Sj)` forms a non-repudiable proof of inclusion of `bj` in `A`.

The provee may then verify that `bj` is a member of `A` using `Rj` but without knowing any other `Bi` `i≠j`. Because the proof also requires cryptographic commitments by the prover via the digital signatures `sj` and `Sj`, a forger must also forge signatures on a pairing and not merely find some set of pairings that produce `A`. 

The only public information is `A`. The members of `A`, i.e. the `{bi}` are blinded by each other with perfect security. Should one of the `bi`, say `b0` have at least 128 bits of entropy then it would serve as a OTP to encrypt all the other accumulated `bi`. This makes a brute force attack on `A` to reveal any of the `{bi}`  computationally infeasible.  The attacker must attack the (public, private) key pairs used to generate and verify the signatures instead. 

An added benefit of using a XORA with the four tuple inclusion proof `(bi, si, Ri, Si)` is that it better supports zero trust computing architectures. One principle of zero trust computing is that all authentic data be verifiably authentic by being signed at rest. This protects against corruption of stored authentic data because the storage mechanism must always verify the signatures on the stored data before use. In this case from an authentic data perspective because  `bi` is a digest of some given data, the signature `si`  on `bi` is cryptographically equivalent to a signature on the given data that  `bi` digests. Thus only `bi` and `si` need be stored together for zero trust authenticity verification on the stored data. This requirement does not expose the full inclusion proof in a given storage device. The signature `Si` on `Ri`  may be stored separately. This protects against a side channel attack on the storage device as a means of defeating the forgery protection.

The main limitation of this type of inclusion proof is that the provee must have some basis for trusting that the prover is consistent or more specifically non-duplicitous in its signed commitments. One basis for trust would be recourse against the prover when the prover is evidently duplicitous. A non-repudiable digital signature means that the prover cannot subsequently repudiate their commitment made with that signature and any mutually inconsistent signatures provide provable duplicity and may therefore may be used by the provee (verifier) to hold the prover accountable, thus providing a basis for trust. Another way of saying this is that the signed proofs must be duplicity evident.

Given we have a system such as KERI that provides provable duplicity detection (i.e. is duplicity evident), we can use the cryptographic accumulator (XORA) signed inclusion proof mechanism described above to construct privacy preserving selective disclosure mechanisms for ACDCs such as hidden attributes or selectively disclosed attributes and/or a revocation registry of one-time-use ACDCs [[9]].

An important application of XORA and related techniques are to provide confidentiality and privacy preserving selective disclosure mechanims for ACDCs. 



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

[24]. GLEIF with KERI Architecture.  

[24]: https://github.com/SmithSamuelM/Papers/blob/master/presentations/GLEIF_with_KERI.web.pdf  

[25]. W3C Decentralized Identifiers (DIDs) v1.0.  

[25]: https://w3c-ccg.github.io/did-spec/

[26]. W3C Verifiable Credentials Data Model v1.1.  

[26]: https://www.w3.org/TR/vc-data-model/  

[27]. Salts, Nonces, and Initial Values.  

[27]: https://medium.com/@fridakahsas/salt-nonces-and-ivs-whats-the-difference-d7a44724a447  

[28]. Rainbow Table.  

[28]: https://en.wikipedia.org/wiki/Rainbow_table  

[29]. Dictionary Attacks, Rainbow Table Attacks and how Password Salting defends against them.  

[29]: https://www.commonlounge.com/discussion/2ee3f431a19e4deabe4aa30b43710aa7 

[30]. Birthday Attack.  

[30]: https://en.wikipedia.org/wiki/Birthday_attack  

[31]. Birthday Attacks, Collisions, And Password Strength.  

[31]: https://auth0.com/blog/birthday-attacks-collisions-and-password-strength/  
[32]. Cost analysis of hash collisions: Will quantum computers make SHARCS obsolete?  

[32]: https://cr.yp.to/hash/collisioncost-20090823.pdf  

[33]. The Provable Security of Ed25519: Theory and Practice.  

[33]: https://eprint.iacr.org/2020/823  

[34]. J. Brendel, C. Cremers, D. Jackson and M. Zhao, "The Provable Security of Ed25519: Theory and Practice," 2021 IEEE Symposium on Security and Privacy (SP), 2021, pp. 1659-1676, doi: 10.1109/SP40001.2021.00042.  

[34]: https://ieeexplore.ieee.org/document/9519456?denied=

[35].Taming the many EdDSAs.  

[35]: https://eprint.iacr.org/2020/1244.pdf  

[37]. JavaScript Object Notation (JSON).  

[37]: https://www.json.org/json-en.html  

[38]. JSON Schema.  

[38]: https://json-schema.org  

[39]. Schema Composition in JSON Schema.  

[39]: https://json-schema.org/understanding-json-schema/reference/combining.html  

[40]. Ricardian Contract.  

[40]: https://en.wikipedia.org/wiki/Ricardian_contract  

[41]. Chain-Link Confidentiality.  

[41]: https://papers.ssrn.com/sol3/papers.cfm?abstract_id=2045818

[42]. Hash Collision Resistance.  

[42]: https://en.wikipedia.org/wiki/Collision_resistance  

[43]. Diffie-Hellman Key Exchange.  

[43]: https://www.infoworld.com/article/3647751/understand-diffie-hellman-key-exchange.html  

[44]: Key Exchange.  

[44]: https://libsodium.gitbook.io/doc/key_exchange  

[45]. Identity System Essentials.  

[45]: https://github.com/SmithSamuelM/Papers/blob/master/whitepapers/Identity-System-Essentials.pdf  
