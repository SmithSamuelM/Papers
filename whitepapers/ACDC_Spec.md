---
tags: ACDC, XORA, KERI, Selective Disclosure
email: sam@samuelsmith.org
version: 0.1.3
---

[![hackmd-github-sync-badge](https://hackmd.io/Ml5eciWhT2K7mUEIAGO9Xw/badge)](https://hackmd.io/Ml5eciWhT2K7mUEIAGO9Xw)


# ACDC Specification

## Introduction

An authentic chained data container (ACDC) [[10]][[11]] is an IETF [[25]] internet draft focused specification being incubated at the ToIP (Trust over IP) foundation [[12]][[13]]. A major use case for the ACDC specification is to provide GLEIF vLEIs (verifiable Legal Entity Identifiers) [[23]]. GLEIF is the Global Legal Entity Identifier Foundation [[24]]. An ACDC is a variant of the W3C Verifiable Credential (VC) specification [[26]]. The VC specification depends on the W3C DID (Decentralized IDentifier) specification [[25]]. ACDCs are dependent on a suite of related IETF focused standards associated with the KERI (Key Event Receipt Infrastructure) [[14]][[15]] specification. These include CESR [[16]], SAID [[17]], PTEL [[18]], CESR-Proof [[19]], IPEX [[20]], and did:keri [[21]]. Some of the major distinguishing features of ACDCs include  normative support for chaining, use of composable JSON Schema [[38]][[39]], multiple serialization formats, compact formats, support for Ricardian contracts [[40]], and a well defined security model derived from KERI [[14]][[15]]. 

The ACDC specification leverages two cryptographic operations namely digests and digital signatures. These operations when used in an ACDC MUST have a cryptographic strength or entropy of approximately 128 bits. (See the appendix for a discussion of cryptographic strength or entropy)


## ACDC Fields

An ACDC may be abstractly modeled as a nested key:value mapping. The term *key* in this context means a field label. To avoid confusion with cryptographic use of the term *key* we use instead the term *field* to refer to mapping pair and the terms *field label* and *field value* for each member of the pair. These pairs can be represented a two tuples e.g `(label, value)`. We qualify this terminology when necessary by using the  the term *field map* to reference to such a mapping. Field maps may be nested where a given field value is itself a reference to another *field map*.  We call this nested set of fields a nested  field map or simpley nest map for short. A field may be represented by a block delimited serialization such as JSON [[37]]. Each field map is represented by an object block with block delimeters `{}`. Given this equivalence we may also use the term block or nested block as synonymous with a field map or nested field map. In many programming languages a field map is implemented as a dictionary or hash table for performant asynchrounous lookup of field values given the field label. Reproducible serialization of field maps requires a canonical ordering of those fields. Once such canonical ordering is called insertion or field creation order. A list of (field, value) pairs provides an ordered representation of any field map. Most programming languages now support ordered dictionaries or hash tables that provide iteration over a list of ordered field (label, value) pairs where the ordering is insertion or field creation order. This enables reproducible round trip serialization/deserializtion of field maps.  ACDCs depend on insertion ordered field maps for canonical serialization. ACDCs support multiple serialization formats but for the sake of simplicity we will only use JSON here [[37]]. The basic set of normative field labels in ACDC field maps is defined in the following table.


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

The primary fields labels are compact in that they use only one or two characters. ACDCs are meant to support resource constrained applications such as supply chain or IoT (Internet of Things) applications. Compact labels better support resource constrained applications in general. With compact labels the over-the-wire verifiable signed serialization consumes a minimum amount of bandwidth. Nevertheless without loss of generality, a one-to-one normative semantic overlay using more verbose expressive field labels may be applied to the normative compact labels after verification of the over-the-wire serialization. This approach better supports secure bandwidth and storage constraints on transmission while not precluding any later semantic post-processing. This is a well known design pattern for resource constrained applications.


### Autonomic IDentifiers

Some fields, such as the `i` and `ri` fields, MUST each have an AID (Autonomic IDentifier) as its value. An AID is fully qualified Self-Certifying IDentifier (SCID) that follows the KERI protocol [[14]][[15]]. A SCID is derived from one or more (public, private) key pairs using asymmetric or public key cryptography to create verifiable digital signatures. Each AID has a set of one or more controllers who each control a private key. By virtue of their private key(s), the set of controllers may make statements on behalf of the associated AID that are backed by uniquely verifiable commitments via digital signatures on those statements. Any entity may then verify those signatures using the associated set of public keys. No shared or trusted relationship between the controllers and verifiers is required. The verifiable key state for AIDs is established with the KERI protocol [[14]][[15]]. The use of AIDS enables ACDCs to be used in a portable but securely attributable, fully decentralized manner in an ecosystem that spans trust domains. Because KERI is agnostic about the name space for any particular AID, different name space standards may be used to express KERI AIDs within AID fields in an ACDC. The examples below use the W3C DID name space specification with the `did:keri` method [[21]]. But the examples would have the same validity from a KERI perspective (dict) of fields. 



### SAID Fields

Several fields may have for their value either a field map  or a SAID. A SAID follows the SAID protocol [[17]]. Essentially a SAID is self-referential self-addressing (content addressable) identifier. A SAID is a special type of cryptographic digest of its encapsulating field map (block). Using a SAID as a field value enables a more compact representation of the associated map from which the SAID is derived. Any nested field map that includes a SAID field may be compacted into its SAID. In an ACDC Several fields may have for their value either a SAID or a map of fields (dictionary). Each SAID provides a stable universal cryptographically verifiable and agile reference to its encapsulating block. 

Recall that given sufficient cryptographic strength including collision resistance [[32]][[42]] a cryptographic commitment (such as a digital signature or cryptographic digest) on a given digest is equivalent to a commitment to the block from which the given digest was derived.  Specifically a digital signature on a SAID makes a verifiable cryptographic non-repudiable commitment to the SAID that is equivalent to the commitment made by a digital signature on the full serialization of the associated block from which the SAID was derived. This enables reasoning viabout ACDCs in whole or in part via their SAIDS in a fully interoperable, verifiable, compact and secure manner. This also supports the well-known bow-tie model of Ricardian Contracts [[40]]. This includes the whole ACDC given by the top level `d` field as well as the SAIDs of any of the nested sections. Specifically, at the top level the value of the `s`, `a`, `e`, and `r` fields may be a SAID instead of a field map. The exploded blocks for each SAID may be attached or cached to optimize bandwidth and availability without decreasing security. 


## Schema is Type

Notable is the fact that there are no top level type fields in an ACDC. This because the schema, `s` field is the type field. ACDCs follow the design principle of separation of concerns between a data container's actual payload information and the type information of that container's payload. In this sense, type information is metadata not data. The schema standard is JSON Schema [[38]]. Composable JSON Schema and conditional JSON Schema allow a validator to answer complex questions about the type of even optional payload elements without mixing type fields into a payload data structure [[39]]. For security reasons the full schema of an ACDC must be completely self-contained and statically fixed for that ACDC. By this we mean that no dynamic schema references or dynamic schema generation mechanisms are allowed. A serialization of a static schema  provides the full schema in detail. A compact representation of the detailed static schema may be provided by its SAID. The associated detailed static schema (uncompacted) is cryptographically bound to its SAID. The detailed static schema may be cached or attached. But composed, cached, and attached are not to be confused with dynamic schema. Specifically, JSON Schema provides complex schema identifiers references that may include non-relative URI references [[46]]. In general non-relative URI references MUST NOT be used because they are not cryptographically end-verifiable. This does not preclude the use of remotely cached SAIDified schema resources because those resources are end-verifiable to their embedded SAID references whereas a bare non-relative URI schema resource is not end-verifiable to its URI reference. A remote SAIDified schema resource may have availability problems but when available the resource is verifiable. Availability is a separate concern. ACDCs are designed to be end-verifiable when available.  To clarify ACDCs MUST use basic JSON Schema with only relative references to standalone self-contained static schema. That self-contained schema may  attached to the ACDC itself or provided via a highly available cache or data store. ACDCs MUST NOT use complex JSON Schema references which allow schema to be obtained dynamically from online JSON Schema Libraries. The latter approach may be difficult or impossible to secure because a cryptographic commitment to the base schema only commits to the non-relative URI reference and not the actual schema resource while the former approach is securely end-verifiable (zero-trust) because a cryptographic commitment to a SAID is equivalent to a commitment to the detailed associated schema itself.



### Selective Disclosure Enabled by High Entropy Digest Blinding

A `u` field may optionally appear in any block at any level of an ACDC. Whenever a block in an ACDC includes a `u` field it makes that block potentially securely selectively discloseable notwithstanding disclosure of the associated schema of the block. Likewise when a `u` field appears at the top level of an ACDC then the whole ACDC itself, not merely some block within the ACDC, may be disclosed in a privacy preserving (correlation minimizing) manner. 

The purpose of the `u` field in any block is to provide sufficient entropy to the SAID of the associated block to make brute force attack on the SAID computationally infeasible. The `u` field may considered a salty nonce [[27]]. Without the entropy provided the `u` field, an adversary may be able to reconstruct the block contents merely from the SAID of the block and the schema of the block using a rainbow or dictionary attack on the set of field values allowed by the schema [[28]][[29]]. The effective entropy of the schema compliant field values may be much less than the strength of the SAID digest. Another way of saying this is that the cardinality of the power set of all combinations of allowed field values may be much less than the cryptographic strength of the SAID. Thus an adversary could successfully brute force discover the exact block by creating digests of all the elements of the power set which if small enough may be computationally feasible. The entropy in the `u` field ensures that the cardinality of the power set allowed by the schema is at least as big as the entropy of the SAID.


## Chain-Link Confidentiality Protection with Composable JSON Schema

One of the most important features of ACDCs is support for Chain-Link Confidentiality[[41]]. This provides a powerful mechanism for protecting against un-permissioned exploitation of the data disclosed via an ACDC. Essentially an exchange of information compatible with chain-link confidentiality starts with an offer by the discloser to disclose confidential information to a potential disclosee. This offer includes sufficient metadate the information to be disclosed such that the disclosee can agree to those terms. Specifically the metadata includes both the schema of the information to be disclosed and the terms of use of that data once disclosed. Once the disclosee has accepted the terms then full disclosure is made. A full disclosure that happens after contractual acceptance of the terms of use we call *permissioned* disclosure. The pre-acceptance disclosure of metadata is a form of partial disclosure.

Composable JSON Schema [[38]][[39]], enable use of the same base schema for both the validation of the partial disclosure of the offer metadata prior to contract acceptance, and validation of full or detailed disclosure after contract acceptance. A cryptographic committment to the base schema securely specifies the allowable semantics for both partial and full disclosure schema validation. Decomposition of the base schema enables a validator to impose more specific semantics at later stages of the exchange process. Specifically, the `oneOf` sub-schema composition operator validates against either the compact SAID of the block or the full block. Decomposing the schema to remove the optional compact form enable a validator to ensure complaint full disclosure. To clarify, a validator can confirm schema compliance both pre and post detailed disclosure by using a composed base schema pre-disclosure and a decomposed more specific schema post-disclosure. These features provide a mechanism for secure schema validated contractually bound selective disclosure of confidential data via ACDCs. 


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


### Selective Disclosure Attribute ACDC

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
  }
]
```

The set of attributes are provided as an array of blocks. Each attribute in the set has its own dedicated block. Each block has its own SAID, `d`, field and `u` field in addition to its attribute field or fields. When an attribute block has more than one attribute field then the set of fields in that block are not independently selectively discloseable but MUST be disclosed together as a set. Note the presence of an Issuer attribute block in the example above, thus making that example a targeted selective disclosure ACDC. An untargeted selective disclosure ACDC would be missing the *Issuer* attribute block as follows:


``` json
"a":
[
  {
    "d": "ELIr9Bf7V_NHwY1lkgveY4-Frn9y2PY9XgOcLxUderzw",
    "u": "0AG7OY1wjaDAE0qHcgNghkDa",
    "score": 96
  },
  {
    "d": "E9XgOcLxUderzwLIr9Bf7V_NHwY1lkFrn9y2PYgveY4-",
    "u": "0AghkDaG7OY1wjaDAE0qHcgN",
    "name": "Jane Doe"
  }
]
```

Given that each attribute block's UUID, `u`, field has sufficient cryptographic entropy, then each attribute block's SAID, `d`, field provides a secure cryptographic digest of the contents of its attribute block that effectively blinds the attribute value from discovery given only its SAID and schema. To clarify, adversary despite being given both the schema of the attribute block and its  SAID, `d`, field, is not able to discover the remaining contents of the attribute block in a computationally feasible manner such as a rainbow table attack [[28]][[29]].  Therefore the `u` field (UUID) of each attribute block enables the associated `d` field to securely blind the block's contents notwithstanding knowledge of the block's schema and its `d` field (SAID).  Moreover a cryptographic commitment to that `d` field does not provide a fixed point of correlation to the attribute field values themselves unless and until there has been disclosure of those field values. 


Given a total of *N* elements in the attributes array, let *a<sub>i</sub>* represent the SAID, *d*, field of the attribute at zero based index *i*. More precisely the set of attributes is expressed as 

\{*a<sub>i</sub> | i ∈ \{0, ..., N-1\}\}*. 

The ordered set of *a<sub>i</sub>*  may be also expressd as a list, that is, 

*[a<sub>0</sub>, a<sub>1</sub>, ...., a<sub>N-1</sub>]*,

or equivalently 

*[a<sub>i</sub> | i ∈ \{0, ..., N-1\}]*

#### Inclusion Proof via Aggregated List

The *a<sub>i</sub>* in the list are aggregated into a single digest denoted *A* by computing the digest of their ordered concatenation. This is expressed as follows:

*A = H(C(a<sub>i</sub> for all i ∈ \{0, ..., N-1\}))* where *H* is the digest (hash) operator and *C* is the concatentation operator.

To be explicit, using the targeted example above, let *a<sub>0</sub>* denote the SAID of the *Issuee* attribute, *a<sub>1</sub>* denote the SAID of the *score* attribute, and *a<sub>2</sub>* denote the SAID of the *name* attribute then the aggregated digest *A* is computed as follows:

*A = H(C(a<sub>0</sub>, a<sub>1</sub>, a<sub>2</sub>))*. 

Equivalently using *+* as the infix concatenation operator, we have, 

*A = H(a<sub>0</sub> + a<sub>1</sub> + a<sub>2</sub>)*

Given sufficient collision resistance of the digest operator, the digest of an ordered concatenation is not subject to a birthday attack on its concatenated elements [[30]][[31]][[32]][[42]].

In compact form, the value of  a selective disclosure top level attribute section, *a*, field is set to the aggregated value *A*. This aggregate *A* makes a blinded cryptographic commitment to the ordered elements in the list

*[a<sub>0</sub>, a<sub>1</sub>, ...., a<sub>N-1</sub>]*. 

Moreover because each *a<sub>i</sub>* element also makes a blinded commitment to its block's attribute value(s), disclosure of any given *a<sub>i</sub>* element does not expose or disclose any discoverable information detail about either its own or  other block's attribute value(s). Therefore one may safely disclose the full list of *a<sub>i</sub>* elements without exposing the blinded block attribute values.

Proof of inclusion in the list consists of checking the list for a matching value. A computationally efficient way to do this is to create a hash table or B-tree of the list and then check for inclusion via lookup in the hash table or B-tree.

Given the aggregate value *A* appearing as the compact value of the top level *a* field (attributed section), a discloser may make a basic provable non-repudiable selective disclosure of a given attribute block, at index *j* by providing to the disclosee four items of information. These are:

- The actual detailed disclosed attribute block itself (at index *j*) with all its fields.
- The list of all attribute block digests, *[a<sub>0</sub>, a<sub>1</sub>, ...., a<sub>N-1</sub>]* that includes *a<sub>j</sub>*.
- The ACDC in compact form with attribute section field *a* having field value *A*.
- The signature(s), *s*, of the Issuee on the ACDC's top level SAID, *d*, field.

The actual detailed disclosed attribute block is only disclosed after the disclosee has agreed to the terms of the rules section. Therefore, in the event the potential disclosee declines to accept the terms of disclosure, then a presentation of the compact version of the ACDC and/or the list of attribute digests, *[a<sub>0</sub>, a<sub>1</sub>, ...., a<sub>N-1</sub>]*. does not provide any point of correlation to any of the attributes. The attributes of block *j* are hidden by *a<sub>j</sub>* and the list of attribute digests *[a<sub>0</sub>, a<sub>1</sub>, ...., a<sub>N-1</sub>]* is hidden by the aggregate *A*.

The disclosee may then verifiy the disclosure by:
- computing *a<sub>j</sub>* on the disclosed attributed block details.
- confirming that the computed *a<sub>j</sub>* appears in the provided list *[a<sub>0</sub>, a<sub>1</sub>, ...., a<sub>N-1</sub>]*.
- computing *A* from the provided list *[a<sub>0</sub>, a<sub>1</sub>, ...., a<sub>N-1</sub>]*.
- confirming that the computed *A* matches the provided *A* in the provided ACDC.
- computing the top level SAID, *d*, field of the provided ACDC.
- confirming presence of issuance seal digest in the Issuer's KEL 
- confirming that the issuance seal digest in the KEL of the Issuer is bound to the ACDC SAID either directly or indirectly through a TEL registry entry.
- verifying the provided signature(s) of the Issuee on the provided top level SAID

The last 3 steps that culminate with verifying the signature(s) requires determining the key state of the Issuer at the time of issuance, this may require additional information and additional verification steps as per the KERI, PTEL, and CESR-Proof protocols.

To protect against later forgery given a later compromise of the signing keys of the Issuer , the issuer Must have anchored an issuance proof digest seal to the ACDC in its KEL. This seal binds the signing key state to the issuance. There are two cases. In the first case an issuance/revocation registry is used. In the second case an issuance/revocation registry is not used. 

When the ACDC is registered using an issuance/revocation TEL (Transaction Event Log) then the the issuance proof digest seal is the SAID of the issuance (inception) event in the ACDC's TEL entry. The issuance event in the TEL includes the SAID of the ACDC. This indirectly binds the ACDC to the issuance proof seal in the Issuer's KEL through the TEL. 

When the ACDC is not registered using an issuance/revocation TEL then the issuance proof is a digest seal of the SAID of the ACDC itself. 

In either case this issuance proof seal makes a verifiable binding between the issuance of the ACDC and the key state of the Issuer at the time of issuance. Because the SAID of the ACDC is also bound to the attribute section SAID, *a* field, then the attribute details of each attribute block are also bound to the key state.

The requirement of an anchored issuance proof seal means that forger Must first successfully publish in the KEL of the issuer an inclusion proof digest seal bound to a forged ACDC. This makes any forgery attempt detectable. Moreover the only way to successfully publish such a seal is in a KEL that has not changed its key state. This makes the forgery attempt detectable as duplicity in any KEL that has changed its key state and recoverble via rotation in any KEL that has not yet changed its key state. To clarify, the issuance proof seal makes any later attempt at forgery using compromised keys detectable. 

#### Inclusion Proof via Merkle Tree

The inclusion proof via aggregated list may be somewhat verbose when there are a large number of attribute blocks in the selectively discloseable attribute section. A more efficient approach is to create a Merkle tree of the attribute block digests and set the value in compat form of the top level attribute section, *a*, field to the Merkle tree root digest.

The Merkle tree needs to have appropriate second-pre-image attack protection of interior branch nodes. The discloser then only needs to provide a subset of digests from the Merkle tree to prove that a given digest, *a<sub>j</sub>* contributed to the Merkle tree root digest. For ACDCs with a small number of attributes the added complexity of the Merkle tree approach may not be worth the savings in verbosity.

#### Hierarchical Derivation at Issuance of Selective Disclosure ACDCs

The amount of data transferred between the Issuer and Issuee or other recipient at issuance of a selective disclosure ACDC may be minimized by using a hierarchical deterministic derivation function to derive the value of the `u` fields from a shared secret salt [[27]]. The Issuer may use a Diffie-Hellman key exchange to share a secret salt with the Issuee [[43]][[44]]. The Issuer also provides a template of the ACDC but with empty `u` (UUID) and `d` (SAID) fields for each block with such fields. Each `u` field value is then derived from the shared salt with a path prefix that indexes a given a given block. Given the `u` field (UUID) value, the `d` field (SAID) may then be derived. Likewise both full and compact versions of the ACDC may then be generated. An example derivation path could be the string "0" for the top level `u` field (for private ACDCS) and the string "0/0" for the zeroth indexed attribute in the attributes array. Likewise the string "0/1" for the next attribute and so on. 

In addition to the shared salt and ACDC template, the Issuer also provides its signature(s) on is own generated compact version ACDC. The Issuer may also provide references to the anchoring issuance proof seals. Everything else an Issuee needs to make a verifiable presentation can be computed at the time of presentation by the Issuee. 

#### Selective Disclosure Attribute ACDC Summary

To summarize, seletive disclosure any given attribute block in the attributes array is provided via a a single aggregated binded commitment to a list of blinded commitments, one to each attribute block. Membership in the aggregated commitment of any given attribute may be proven without leaking the actual attribute value(s). This provides for provable selective disclosure of the attribute values. This approach does not require any more complex cryptography than digests and digital signatures. This satisfies the design ethos of minimally sufficient means. The primary drawback of this approach is verbosity. It trades ease of implementation for size. 

Given sufficient collision resistance of the digest and signatures, either inclusion proof format (list or Merkle tree) prevents a disclosee or any adversary from discovering in a computationally feasible way the values of any undisclosed attribute details from the combination of `A`, and/or the list of attribute block digests, *[a<sub>0</sub>, a<sub>1</sub>, ...., a<sub>N-1</sub>]* [[32]][[42]]. A disclosee would also need the actual attribute block details.

A private selective disclosure ACDC provides significant correlation minimization because a presenter may use a metadata ACDC prior to acceptance by the disclosee of the terms of the chain-link confidentiality expressed in the rule section [[41]]. Thus only malicious disclosees who violate chain-link confidentiality may correlate between presentations of a given private selective disclosure ACDC. But they are not able to discover any undisclosed attributes.

ACDCs, as a standard, benefit from a minimally sufficient approach to selective disclosure that is simple enough to be universally implementable and adoptable. This does not preclude support for other more sophisticated approaches as optional standards. But the minimally sufficient approach should be universal so that at least one selective disclosure mechanism be made available in all ACDC implementations. To clarify, not all instances of an ACDC must employ the minimal selective disclosure mechanism as described above but all ACDC implementations must support any instance of an ACDC that employs the minimal selective disclosure mechanism as described above.


## Bulk Issued Private ACDCs

The purpose of bulk issuance is to more efficiently enable the Issuee to use unique ACDC SAIDs to isolate and thereby minimize correlation across different usage contexts of essentially the same ACDC while allowing public commitments to the ACDC SAIDs. A private ACDC may be issued in bulk as a set. In its basic form, the only difference between each ACDC is the top level SAID, *d*, and UUID, *u* field values. To elaborate bulk issuance enables the use of un-correlatable copies while minimizing the associated data transfer and the storage requirements. Essentially each copy of a bulk issued ACDC set shares a template that both the Issuer and Issuee use to generate a given ACDC in that set without requiring that the Issuer and Issuee exchange and store unique copies of member of the set independently. This minimizes the data transfer and storage requirements for both the Issuer and Issuee.

An ACDC provenance chain is connected via references to the SAIDs given by the top level `d` fields, of the ACDCs in that chain.  A given ACDC thereby makes commitments to other ACDCs. Expressed another way, an ACDC may be a node in a directed graph of ACDCs. Each directed edge in that graph emanating from one ACDC includes a reference to the SAID of some other connected ACDC. These edges provide points of correlation to an ACDC via their SAID reference. Private bulk issued ACDCs enable the Issuee to better control the correlatability of presentations using different presentation strategies.  

For example, the Issuee could use one copy of a bulk issued private ACDC per presentation even to the same verifier. This strategy would consume the most copies. It is essentially a one-time-use ACDC strategy. Alternatively, the Issuee could use the same copy for all presentations to the same verifier and thereby only permit the verifier to correlate between presentations it received directly but not between other verifiers. This limits the consumption to one copy per verifier. In yet another alternative, the Issuee could use one copy for all presentations in given context with a group of verifiers thereby only permitting correlation among that group. 

In this context we are talking about permissioned correlation. Any verifier that has received a full presentation of a private ACDC has access to all the fields disclosed by the presentation but the terms of the chain-link confidentiality agreement may forbid sharing those field values outside a given context. Thus an Issuee may use a combination of bulk issued ACDCs with chain-link confidentiality to control permissioned correlation of the contents of an ACDC while allowing the SAID of the ACDC to be more public. The SAID of a private ACDC does not expose the ACDC contents to an un-permissioned third party. Unique SAIDs belonging to bulk issued ACDCs prevent third parties from making provable correlation between ACDCs via their SAIDs in spite of those SAIDs being public. This does not stop malicious verifiers (as second parties) from colluding and correlating against the disclosed fields but it does limit provable correlation to the information disclosed to a given group of malicious colluding verifiers. To restate unique SAIDs per copy of a set of private bulk issued ACDC prevent un-permissioned third parties from making provable correlation in spite of those SAIDs being public unless they collude with malicious verifiers (second parties).


In some applications, chain-link-confidentiality is insufficient to deter un-permissioned correlation. Some verifiers may be malicious with sufficient malicious incentives to overcome whatever counter incentives the terms of the contractual chain-link confidentiality may impose. In these cases more aggressive technological anti-correlation mechanisms such as bulk issued ACDCs may be useful. To elaborate, in spite of the the fact that chain-link confidentiality terms of use may forbid such malicious correlation, making such correlation more difficult technically may provide better protection than chain-link confidentiality alone [[41]].

It is important to note that any group of colluding malicious verifiers may always make statistical correlation between presentations despite technical barriers to cryptographically provable correlation. In general there is no cryptographic mechanism that precludes statistical correlation among a set of colluding verifiers because they may make cryptographically unverifiable or unprovable assertions about information presented to them that may be proven as likely true using merely statistical correlation techniques.


### Basic Bulk Issuance

The amount of data transferred between the Issuer and Issuee or other recipient at issuance of a set of bulk issued ACDCs may be minimized by using a hierarchical deterministic derivation function to derive the value of the *u* fields from a shared secret salt [[27]]. The Issuer may use a Diffie-Hellman key exchange to share a secret salt with the Issuee [[43]][[44]]. The Issuer also provides a template of the private ACDC but with empty *u* (UUID) and *d* (SAID) fields for the top level and each nested block with such fields. Each *u* field value is then derived from the shared salt with a deterministic path prefix that indexes both its membership in the bulk issued set and its location in the ACDC. Given the *u* field (UUID) value, the associated *d* field (SAID) may then be derived. Likewise both full and compact versions of the ACDC may then be generated. This is analogous to that described in the section for selective disclosure ACDCs but extended to a set of private ACDCs. An example deterministic derivation path would be to use a string of the copy index such as "0", "1", "2" etc as the initial element in each path. 


In general if *k* is the index of an ordered set of bulk issued private ACDCs of size *M*, the derivation path could start with the string *"k"* where *k* is replaced with the decimal or hexidecimal textual representation of the numeric index *k*. Furthermore, a bulk issued private ACDC with a private attribute section could use *"k"* for its top level UUID and *"k/0"* for the attribute section UUID. This hierarchical path could be extended to any nested private attribute blocks. This approach can be further extended to enable bulk issued selective disclosure ACDCs by using a similar hierarchical derivation path for the UUID field value in each of the selectively discloseable blocks in the array of attributes. For example, the path *"k/j"* may be used to generate the UUID of attribute index *j* at ACDC index *k*.

In addition to the shared salt and ACDC template, the Issuer also provides a list of signatures of SAIDs, one for each SAID of each copy of the associated compact ACDC.  The Issuee at the time of presentation can generate from the template and the salt the full ACDC along with the non-repudiable signature. The Issuee does not need to store a copy of each bulk issued ACDC, merely the template, the salt, and the list of signatures. 

The Issuer must also anchor in its KEL an issuance proof digest seal of the set of bulk issued ACDCs. The issuance proof digest seal makes a cryptographic commitment to the set of top level SAIDS belonging to the bulk issued ACDCs. This protects against later forgery of ACDCs in the event the Issuer's signing keys become compromised.  A later attempt at forgery requires a new event or new version of an event that includes a new anchoring issuance proof digest seal that makes a cryptographic commitment the set of new forged ACDC SAIDS. This new anchoring event of the forgery is therefore detectable.

Similarly to the selective disclosure attribute ACDC, the issuance proof digest is aggregated from all members in the bulk issued set of ACDCs. The complication of this approach is that it must be done in such a way as to not enable provable correlation by a third party of the actual SAIDS of the bulk issued set of ACDCs. Therefore the actual SAIDs must not be part of the aggregation but blinded commitments to those SAIDs instead. With blinded commitments, knowledge of any or all members of such a set does not disclose the membership of any SAID unless and until it is unblinded. Recall that the purpose of bulk issuance is to allow the SAID of an ACDC in a bulk issued set to be used publicly without disclosing in an un-permissioned provable way the SAIDs of the other members. 

The basic approach is to compute the aggregation as the digest of the concatenation of a set of blinded digests of bulk issued ACDC SAIDS. Each ACDC SAID is first blinded via concatentation to a UUID (salty nonce) and then the digest of that concatentation is concatenated with the other blinded SAID digests. Finally a digest of that concatenation provides the aggregate. 

Suppose there are *M* ACDCs in a bulk issued set. Using zero based indexing for each member of the bulk issued set of ACDCs, such that index *k* satisfies *k ∈ \{0, ..., M-1\}, let *d<sub>k</sub>* denote the top level SAID of an ACDC in an ordered set of bulk issued ACDCs. Let *v<sub>k</sub>* denote the UUID (salty nonce) used to blind that said. This is not the top level UUID, *u*, field of the ACDC itself but an entirely different UUID used to blind the ACDC's SAID for the purpose of aggregation. The derivation path for *v<sub>k</sub>* from the shared secret salt is *"k."* where *k* is the index of the ACDC. 

Let *c<sub>k</sub> = v<sub>k</sub> + d<sub>k</sub>* where *+* is the infix concatenation operator denote the blinding concatenation.  
Then the blinded digest *b<sub>k</sub>* is given by,  
*b<sub>k</sub> = H(c<sub>k</sub>) = H(v<sub>k</sub> + d<sub>k</sub>)*   
where *H* is the digest operator. The aggregation of blinded digests, *B*, and is given by,  
*B = H(C(b<sub>k</sub> for k ∈ \{0, ..., M-1\}))*   
where *C* is the concatenation operator and  *H* is the digest operator. This aggregation, *B* provides the issuance proof digest. 

This aggregate *B* makes a blinded cryptographic commitment to the ordered elements in the list
*[b<sub>0</sub>, b<sub>1</sub>, ...., b<sub>M-1</sub>]*. A commitment to *B* is a commitment to all the *b<sub>k</sub>* and hence all the d<sub>k</sub>.

Given sufficient collision resistance of the digest operator, the digest of an ordered concatenation is not subject to a birthday attack on its concatenated elements [[30]][[31]][[32]][[42]].

Disclosure of any given *b<sub>k</sub>* element does not expose or disclose any discoverable information detail about either the SAID of its associated ACDC or any other ACDC's SAID. Therefore one may safely disclose the full list of *b<sub>k</sub>* elements without exposing the blinded bulk issued SAID values, d<sub>k</sub>.

Proof of inclusion in the list consists of checking the list for a matching value. A computationally efficient way to do this is to create a hash table or B-tree of the list and then check for inclusion via lookup in the hash table or B-tree.

A proof of inclusion in a bulk set requires disclosure of *v<sub>k</sub>* which is only disclosed after the disclosee has accepted (agreed to) the terms of the rule section. Therefore, in the event the verifier declines to accept the terms of disclosure, then a presentation of the compact version of the ACDC does not provide any point of correlation to any other SAID of any other ACDC from the bulk set in the aggregate *B*. In addition, because the other SAIDs are hidden by each *b<sub>k</sub>* inside `B` even a presentation of *[b<sub>0</sub>, b<sub>1</sub>, ...., b<sub>M-1</sub>]* does not provide any point of correlation to the actual bulk issued ACDC without *v<sub>k</sub>*. Indeed if the Discloser uses a metadata version of the ACDC in its offer then even its SAID is not disclosed until after acceptance of terms in the rule section.

A discloser may make a basic provable non-repudiable selective disclosure of a given bulk issued ACDC, at index *k* by providing to the disclosee four items of information (proof of inclusion). These are:

- The ACDC in compact form (at index *k*) with all its fields and having *d<sub>k</sub>* as the value of its top level SAID, *d*, field.
- The blinding UUID, *v<sub>k</sub>* from which *b<sub>k</sub> = H(v<sub>k</sub> + d<sub>k</sub>)* may be computed.
- The list of all blinded SAIDs, *[b<sub>0</sub>, b<sub>1</sub>, ...., b<sub>M-1</sub>]* that includes *b<sub>k</sub>*.
- The signature(s), *s<sub>k</sub>*, of the Issuee on the ACDC's top level SAID, *d<sub>k</sub>*, field.

The disclosee may then verifiy the disclosure by:
- computing *d<sub>j</sub>* on the disclosed compact ACDC.
- computing *b<sub>k</sub> = H(v<sub>k</sub> + d<sub>k</sub>)*
- confirming that the computed *b<sub>k</sub>* appears in the provided list *[b<sub>0</sub>, b<sub>1</sub>, ...., b<sub>M-1</sub>]*.
- computing *b* from the provided list *[b<sub>0</sub>, b<sub>1</sub>, ...., b<sub>M-1</sub>]*..
- confirming the presence of an issuance seal digest in the Issuer's KEL that makes a commitment to *B* either directly or indirectly through a TEL registry entry.
- verifying the provided signature(s), *s<sub>k</sub>*, of the Issuee on the provided top level SAID, *d<sub>k</sub>*, field.

The last 3 steps that culminate with verifying the signature(s) requires determining the key state of the Issuer at the time of issuance, this may require additional information and additional verification steps as per the KERI, PTEL, and CESR-Proof protocols.

Recall that the goal is to generate an aggregated value that enables an Issuee to selectively disclose one ACDC in a bulk issued set without leaking the other members of the set to un-permissioned parties (second or third).

To protect against later forgery given a later compromise of the signing keys of the Issuer , the issuer Must have anchored an issuance proof digest seal to the ACDC in its KEL. This seal binds the signing key state to the issuance. There are two cases. In the first case an issuance/revocation registry is used. In the second case an issuance/revocation registry is not used. 

When the ACDC is registered using an issuance/revocation TEL (Transaction Event Log) then the the issuance proof digest seal is the SAID of the issuance (inception) event in the ACDC's TEL entry. The issuance event in the TEL includes *B* as its identifier value. This indirectly binds the aggregater *B* to the issuance proof seal in the Issuer's KEL through the TEL. Recall that the usual purpose of a TEL is to provide a verifiable data registry that enables enables dynamic revocation of an ACDC via a state of the TEL. A verifier checks the state at the time of use to see if the associated ACDC has been revoked. The Issuer is in control of changing the state of the TEL. The registry identifier, *ri*, field is used to indicate the public registry which usually has a TEL entry for each ACDC. Typically the identifier of each TEL entry is the SAID of the TEL's inception event which is a digest of the event contents which include the SAID of the ACDC. In the bulk issuance case however, the TEL's inception event contents include the aggregated value *B* instead of the SAID of a given ACDC. This allows all members of the bulk issued set to share the same TEL without any third party being able to discover which TEL in an un-permissioned provable way. In order to prove which TEL a specific bulk issued ACDC is using the full inclusion proof must be disclosed. 

When the ACDC is not registered using an issuance/revocation TEL then the issuance proof is a digest seal of *B* itself. 

In either case this issuance proof seal makes a verifiable binding between the issuance of all the ACDCs in the bulk issued set and the key state of the Issuer at the time of issuance. 

The requirement of an anchored issuance proof seal means that forger Must first successfully publish in the KEL of the issuer an inclusion proof digest seal bound to a set of forged bulk issued ACDCs. This makes any forgery attempt detectable. Moreover the only way to successfully publish such a seal is in a KEL that has not changed its key state. This makes the forgery attempt detectable as duplicity in any KEL that has changed its key state and recoverble via rotation in any KEL that has not yet changed its key state. To clarify, the issuance proof seal makes any later attempt at forgery using compromised keys detectable. 


#### Inclusion Proof via Merkle Tree

The inclusion proof via aggregated list may be somewhat verbose when there are a very large number of builk issued ACDCs in a given set. A more efficient approach is to create a Merkle tree of the blinded SAID digests, *b<sub>k</sub>* and set the aggregate *B* to be the Merkle tree root.

The Merkle tree needs to have appropriate second-pre-image attack protection of interior branch nodes. The discloser then only needs to provide a subset of digests from the Merkle tree to prove that a given digest, *b<sub>k</sub>* contributed to the Merkle tree root digest. For a small numbered bulk issued set of ACDCs the added complexity of the Merkle tree approach may not be worth the savings in verbosity.



#### Bulk Issuance of Private ACDCs with Unique Issuee AIDs

One potential point of provable but unpermissioned correlation among any group of colluding malicious verifiers may arise when the same Issuee AID is used for presentation to all verifiers in that group. Recall that the contents of private ACDCs are not disclosed except to permissioned verifiers, thus a common Issuee AID would only be point of correlation for a group of colluding malicious verifiers.

One solution to this problem is for the Issuee to use a unique AID for the copy of a bulk issued ACDC presented to each verifier in a given context. This requires that each ACDC copy in bulk issued set use a unique Issuee AID. This would enable the Issuee in a given context to minimize provable correlation by malicious verifiers against any given Issuee AID. In this case, the bulk issuance process may be augmented to include the derivation of a unique AID for each copy of the ACDC by including in the inception event that defines the Issuee's self-addressing AID, a digest seal derived from the shared salt and copy index. This can be re-generated at time of presentation by the Issuee. Each unique Issuee AID would need its own KEL. But generation and publication of the associated KEL can be delayed until the bulk issued ACDC is actually used. This approach completely isolates a given Issuee AID in a given context with respect to the use of a bulk issued private ACDC to even better protect against un-permissioned correlation by malicious verifiers. 

The derivation path for the digest seal is *"k/0."* where *k* is the index of the ACDC. To clarify *"k/0.""* specifies the path to generate the UUID to be included in the inception event that generates the Issuee AID for the ACDC at index *k*.


### Independent TEL Bulk Issued ACDCs


Recall that the purpose of using the aggregate *B* for a bulk issued set from which the TEL identifier is derived is to enable a set of bulk issued ACDCs to share a single public TEL that provides dynamic revocation but without enabling un-permissioned correlation to any other members of the bulk set by virtue of the shared TEL. This enables the issuance/revocation/transfer state of all copies of set of bulk issued ACDCs to be provided by a single TEL which minimizes the storage and compute requirements on the TELs registry while providing selective disclosure to prevent un-permissioned correlation via the public TEL.  

In some applications, however, where chain-link confidentiality does not sufficiently deter malicious provable correlation by verifiers, an Issuee may benefit from using ACDC with independent TELs but that are still issued in bulk manner. 

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

Each edge sub-block has one required node field labeled `n`. The value of the node, `n` field is the SAID of the ACDC to which the edge connects. 

A labeled property graph allows each edge to have its own set of properties in its own sub-block. These might include modifiers that influence how the connected node is to be used such as a weight. Weighted directed edges the represent degrees of confidence or liklihood are commonly used for machine learning or reasoning under uncertainty. The following example adds a weight property to the edge sub-block as indicated by the `w` field .

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

Abstractly an ACDC with one or more edges may be a fragment of a distributed labeled property graph. However the local label does not enable unique global resolution of a given edge with properties other than the node, `n` field, property. To make an edge with additional  properties also globally uniquely resolvable we add a `d` field that is the SAID of the edge's sub-block. Because the SAID is a cryptographic digest it will universally and uniquely identify an edge with a given set of properties. This allows ACDCs to be used as secure fragments of a globally distributed labeled property graph. This combines the beneficial features of property graphs and global knowledge graphs in a secure manner that crosses trust domains. This is shown below.


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

Given that an individual edge sub-block  has a  `d` field (SAID) then a compact representation of the sub-block is to replace it with its SAID. This may be useful for complex edges with many properties. This is shown as follows:


```
"e": 
    {
      "d": "EerzwLIr9Bf7V_NHwY1lkFrn9y2PgveY4-9XgOcLxUdY",
      "qvi": "E9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NHwY1lkFrn",
    }

```


When an edge sub-block has only one field, its `n` field then the edge block may use an alternate simplified compact form where the field value is the `n` field. The schema for that particular edge label, in this case, `qvi`,  will indicate that the edge value is a node SAID and not the edge sub-block SAID as would be the case for the normal compact form shown above. This alternate compact form is shown below.

```
"e": 
    {
      "d": "EerzwLIr9Bf7V_NHwY1lkFrn9y2PgveY4-9XgOcLxUdY",
      "qvi": "EIl3MORH3dCdoFOLe71iheqcywJcnjtJtQIYPvAu6DZA"
    }

```



When a edge points to a private ACDC, an Issuer may choose to use a metadata version of that private ACDC when presenting the, `n`, field of and edge prior to acceptance of the terms of disclosure. The verifier can verify the metadata of the private node without the Issuer exposing that actual node contents via the actual node (ADCD) SAID or other attributes.

Private ACDCs (nodes) and private edges may be used in combination to prevent unpermissioned correlation of the distributed labeled property graph.

In general lookup of a the details of an ACDC reference as a node, `n` field value, in an edge begins with its provided SAID or the SAID of its associated edge sub-block. Because a SAID is cryptographic digest with high collision resistance it provides a universally unique identifier to the referenced ACDC. Discovery of a service endpoint URL that provides database access to a copy of the ACDC may be bootstrapped via an OOBI (Out-Of-Band-Introduction) that links the service endpoint URL to the SAID of the ACDC. Alternatively the issuer may provide as an attachment at issuance a copy of the referenced ACDC. In either case after a successful issuance exchange the Issuee or holder of any ACDC will have either a copy or a means of obtaining a copy of any referenced ACDCs as nodes in the edge sections of all ACDCs so chained. That Issuee or holder will then have everything it needs to make a successful presentation to a verifier. This is the essence of percolated discovery.


## Rule Section

In the compact ACDC examples above the rule section has been compacted into merely the SAID of that section. Suppose that the un-compacted value of the rule section denoted by the `r` field is as follows:

```
"r": 
  {
    "d": "EwY1lkFrn9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NA",
    "warrantyDisclaimer": 
      {
        "l": "Issuer provides this credential on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied, including, without limitation, any warranties or conditions of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A PARTICULAR PURPOSE",
      },
    "liabilityDisclaimer": 
      {
        "l": " In no event and under no legal theory, whether in tort (including negligence), contract, or otherwise, unless required by applicable law (such as deliberate and grossly negligent acts) or agreed to in writing, shall the Issuer be liable for damages, including any direct, indirect, special, incidental, or consequential damages of any character arising as a result of this credential. "
      }
  }
```

The `d` field at the top level of the rule block is the SAID of that block and is the same SAID used as the compacted value of the `r` field that appears at the top level of the ACDC. Each clause in the rule section gets is own field. Each clause also has its own local label.

The `l` field in each block provides the associated legal language.  

Note there are not type fields. The type of a contract and the type of each clause is provided by the schema vis-a-vis the label of that clause. This is in accordance with the design principle of ACDCs that may be succintly expressed as "the schema is the type". 

The purpose of the rule section is to provide a Ricardian Contract [[40]]. The important features of a Ricardian contract is that it be both human and machine readable and referenceable by a cryptotographic digest. A JSON encoded document is a practical example of both a human and machine readable document.  The SAID, `d`, field at the top level of the rule section provides the digest. This provision supports the bow tie model of Ricardian Conracts. Legal contracts may be hierarchically structured into sections and subsections with named or numbered clauses in each section. The labels on the clauses may follow such a hierarchical structure using nested maps or blocks. 



The clause may also have a clause SAID, in its `d` field. Clause specific SAIDs enable reference to individual clauses not merely the whole contract as given by the SAID of the rule section.



An example rule section with SAIDS  for each clause is provided below:


```
"r": 
  {
    "d": "EwY1lkFrn9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NA",
    "warrantyDisclaimer": 
      {
        "d": "EXgOcLxUdYerzwLIr9Bf7V_NAwY1lkFrn9y2PgveY4-9",
        "l": "Issuer provides this credential on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied, including, without limitation, any warranties or conditions of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A PARTICULAR PURPOSE",
      },
    "liabilityDisclaimer": 
      {
        "d": "EY1lkFrn9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NAw",
        "l": " In no event and under no legal theory, whether in tort (including negligence), contract, or otherwise, unless required by applicable law (such as deliberate and grossly negligent acts) or agreed to in writing, shall the Issuer be liable for damages, including any direct, indirect, special, incidental, or consequential damages of any character arising as a result of this credential. "
      }
  }
```

The used of SAIDS in each clause enables a compact form of a set of clauses where each clause value is the SAID of the corresponding clause. For example:

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

An alternate simplified compact form uses the value of the `l` field as the value of the of the clause field label. The schema for a specific clause label will indicate that field value, for a given clause label is the legal language itself and not the clause block SAID as is the normal compact form shown above. This alternate compact form is shown below.

```
"r": 
  {
    "d": EwY1lkFrn9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NA",
    "warrantyDisclaimer": "Issuer provides this credential on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied, including, without limitation, any warranties or conditions of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A PARTICULAR PURPOSE",
    "liabilityDisclaimer": " In no event and under no legal theory, whether in tort (including negligence), contract, or otherwise, unless required by applicable law (such as deliberate and grossly negligent acts) or agreed to in writing, shall the Issuer be liable for damages, including any direct, indirect, special, incidental, or consequential damages of any character arising as a result of this credential. "
}
```


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




## Appendix: Cryptographic Strength and Security

### Cryptographic Strength 

For crypto-systems with *perfect-security*, the critical design parameter is the number of bits of entropy needed to resist any practical brute force attack. In other words, when a large random or pseudo-random number from a cryptographic strength pseudo-random number generator (CSPRNG) [[6]] expressed as string of characters is used as a seed or private key to a cryptosystem with *perfect-security*, the critical design parameter is determined by the amount of random entropy in that string needed to withstand a brute force attack. Any subsequent cryptographic operations must preserve that minimum level of cryptographic strength. In information theory [[7]] the entropy of a message or string of characters is measured in bits. Another way of saying this is that the degree of randomness of a string of characters can be measured by the number of bits of entropy in that string.  Assuming conventional non-quantum computers, the convention wisdom is that, for systems with information theoretic or perfect security, the seed/key needs to have on the order of 128 bits (16 bytes, 32 hex characters) of entropy to practically withstand any brute force attack. A cryptographic quality random or presudo-random number expressed as a string of characters will have essentially as many bits of entropy as the number of bits in the number. For other crypto-systems such as digital signatures that do not have perfect security the size of the seed/key may need to be much larger than 128 bits in order to maintain 128 bits of cryptographic strength.

An N-bit long base-2 random number has 2<sup>N</sup> different possible values. Given that with perfect security no other information is available to an attacker, the attacker may need to try every possible value before finding the correct one. Thus the number of attempts that the attacker would have to try may be as much as 2<sup>N-1</sup>.  Given available computing power, one can easily show that 128 is a large enough N to make brute force attack computationally infeasible.  

Let's suppose that the adversary has access to supercomputers. Current supercomputers can perform on the order of one quadrillion operations per second. Individual CPU cores can only perform about 4 billion operatons per second but a supercomputer will employ many cores in parallel. A quadrillion is approximately 2<sup>50</sup> = 1,125,899,906,842,624. Suppose somehow an adversary had control over one million (2<sup>20</sup> = 1,048,576) super computers which could be employed in parallel to mount a brute force attack. The adversary could then try 2<sup>50</sup> * 2<sup>20</sup> = 2<sup>70</sup> values per second (assuming very conservatively that each try only took one operation).
There are about 3600 * 24 * 365 = 313,536,000 = 2<sup>log<sub>2</sub>313536000</sup>=2<sup>24.91</sup> ~= 2<sup>25</sup> seconds in a year. Thus this set of a million super computers could try 2<sup>50+20+25</sup> = 2<sup>95</sup> values per year. For a 128-bit random number this means that the adversary would need on the order of 2<sup>128-95</sup> = 2<sup>33</sup> = 8,589,934,592 years to find the right value. This assumes that the value of breaking the cryptosystem is worth the expense of that much computing power. Consequently, a cryptosystem with perfect security and 128 bits of cryptographic strength is computationally infeasible to break via brute force attack.

### Information Theoretic Security and Perfect Security

The highest level of crypto-graphic security with respect to a cryptographic secret (seed, salt, or private key) is called  *information-theoretic security* [[1]]. A cryptosystem that has this level of security cannot be broken algorithmically even if the adversary has nearly unlimited computing power including quantum computing. It must be broken by brute force if at all. Brute force means that in order to guarantee success the adversary must search every combination of key or seed. A special case of *information-theoretic security* is called *perfect-security* [[1]].  *Perfect-security* means that the cipher text provides no information about the key. There are two well-known cryptosystems that exhibit *perfect security*. The first is a *one-time-pad* (OTP) or Vernum Cipher [[2]][[3]], the other is *secret splitting* [[4]], a type of secret sharing [[5]] that uses the same technique as a *one-time-pad*. 



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

[46]. Complex JSON Schema Identifiers.  

[46]: https://json-schema.org/understanding-json-schema/structuring.html#base-uri  