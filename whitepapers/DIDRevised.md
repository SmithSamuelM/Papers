# DID Hardening

## Overview

This is detailed proposal for modifying the DID specification. [DID specification](https://w3c-ccg.github.io/did-spec/). Motivations for this proposal include enabling key management as a first order feature of DIDs as well as leveraging existing tooling for URL and UUID identifiers.

The current DID specification explicitly states that DIDs include both the unique non-colliding decentralized generation features of URNs (UUIDs) with the locatability features of URLs. What the current DID specification implies but does not explicitly state is that DIDs also include the self-certifying features of public keys. Indeed it is this feature of DIDs, that is, that they are also derived from the public key of the crytographic public/private key pair that make them most attractive as decentralized identifiers. Moreover their nature as keys means that key management should be a first order feature of the DID specification.

The current DID syntax specification is a minor modification of the existing URL syntax. [RFC3986](https://tools.ietf.org/html/rfc3986). As such it benefits from familiarity which is a boon to adoption. Too the extent that the DID spec is compatible with existing URL parsing tools with nothing more than trivial modification, then that would be an even further boon to adoption. Consequently,  this proposal uses a URL parsing tool to verify the degree of reusability of current URL tooling for DIDs. All else being equal when comparing two options for revising the DID spec, if one option can use existing parsing tools it should be preferred over another that requires building new parsing tools (unless trivial).

The parsing tool is the Python3  urllib.parse.urlsplit tool.

```python
>>> from urllib.parse import urlsplit
>>> urlsplit("did:sid:ABCDEFGHI=/abc/def#path/0")
SplitResult(scheme='did', netloc='', path='sid:ABCDEFGHI=/abc/def', query='', fragment='path/0')
```

The relevant portions of the ABNF syntax for URLs are as follows.

### URL ABNF

```abnf
URI           = scheme ":" hier-part [ "?" query ] [ "#" fragment ]

scheme      = ALPHA *( ALPHA / DIGIT / "+" / "-" / "." )

hier-part     = "//" authority path-abempty
                / path-absolute
                / path-rootless
                / path-empty
                
path          = path-abempty    ; begins with "/" or is empty
                    / path-absolute   ; begins with "/" but not "//"
                    / path-noscheme   ; begins with a non-colon segment
                    / path-rootless   ; begins with a segment
                    / path-empty      ; zero characters

path-abempty  = *( "/" segment )
path-absolute = "/" [ segment-nz *( "/" segment ) ]
path-noscheme = segment-nz-nc *( "/" segment )
path-rootless = segment-nz *( "/" segment )
path-empty    = 0<pchar>

segment       = *pchar
segment-nz    = 1*pchar
segment-nz-nc = 1*( unreserved / pct-encoded / sub-delims / "@" )
                    ; non-zero-length segment without any colon ":"

pchar         = unreserved / pct-encoded / sub-delims / ":" / "@"
                
query         = *( pchar / "/" / "?" )

fragment      = *( pchar / "/" / "?" )


unreserved  = ALPHA / DIGIT / "-" / "." / "_" / "~"

reserved    = gen-delims / sub-delims

gen-delims  = ":" / "/" / "?" / "#" / "[" / "]" / "@"

sub-delims    = "!" / "$" / "&" / "'" / "(" / ")"
                / "*" / "+" / "," / ";" / "="


```

### DID ABNF

The current DID syntax (ABNF) is as follows:

```abnf
did-reference      = did [ "/" did-path ] [ "#" did-fragment ]
 did                = "did:" method ":" specific-idstring
 method             = 1*methodchar
 methodchar         = %x61-7A / DIGIT
 specific-idstring  = idstring *( ":" idstring )
 idstring           = 1*idchar
 idchar             = ALPHA / DIGIT / "." / "-"

did-fragment      = *( pchar / "/" / "?" )
```

Note the DID ABNF above is not complete. The ABNF for the did-path is missing. Given the complexity of the hier-part of the URL spec, the did-path needs to be completely specified.

The current DID syntax is compatible with the URL syntax but the semantics are different in some important ways. One difference is that, if the did-path is missing, then the meaning of the did-fragment is special.

### DID Fragment

From the DID Spec (sec 5.5):

A generic DID fragment (the did-fragment rule in Section 3.1 The Generic DID Scheme) is identical to a URI fragment and MUST conform to the ABNF of the fragment ABNF rule in [RFC3986]. A DID fragment MUST be used only as a method-independent pointer into the DID Document to identify a unique key description or other DID Document component. To resolve this pointer, the complete DID reference including the DID fragment MUST be used as the value of the id key for the target JSON object.

A specific DID scheme MAY specify ABNF rules for DID fragments that are more restrictive than the generic rules in this section.

DID Fragment:

The portion of a DID reference that follows the first hash sign character ("#"). A DID fragment uses the same syntax as a URI fragment. See section 5.5. 
Note that a DID fragment MUST immediately follow a DID. If a DID reference includes a DID path followed by a fragment, that fragment is NOT a DID fragment.


The primary purpose of the did-path is to reference or locate resources associated with a DID such as a DID service endpoint. This provides a way to index a group of related information that is associated with a DID. When a did-path is present then the did-fragment is interpreted relative to the resource indicated by the did-path. Thus if the did-path points to a document then the did-fragment might be a given field in the document. 

When the did-path is missing then the did-fragment references fields in a special document, that is, the DID Document. The DID Document holds meta-data about the DID. Some of that meta-data has to do with the DID acting as a self-certifying identifier (public key), in that the holder of the associated private key can authenticate itself as the holder. Another meta-data function of the DID Document is to provide information about authorizations to change the DID Document and to sign the DID document in order to detect tampering.

What we have is facility via the DID document to provide some types of key related operations associated with DIDs but not all. Importantly there is ambiguity in that the DID Document is a catch all for several different operations and functions.  

There is a difference between operations that manage keys such as key regeneration, key rotation, key revocation, and key recovery and operations that use keys such as authentication, authorization, signing and encryption as well as operations on the DID document itself that manage its special role such as proving provenance of the contents of the DID Document relative to the holder of the private key associated with the DID.

The current DID specification does not provide clear unambiguous syntactic and semantic rules for these different classes of operations. Indeed one primary motivation for this proposal is to work toward providing those rules.


### Self Certifying Data

This proposal is also motivated and informed by a use case that was not anticipated by the current DID specification. The use case is data intensive applications of self-certifying data (SCD) or self-identifying data (SID), A self-certifying data item is a data structure that is identified by an included DID and its provenance can be verified by cryptographic operations using the associated public/private key pair. The data structure carries both application data and the DID meta-data needed for verifing and proving provenance without perfoming expensive external lookups. Thus a self-certifying data item encapsulates in place the essential portions of meta-data from a DID document and service endpoints. It is an self-contained structure that is compatible with distributed data flow or stream processing such as reputation. 
Moreover, the current DID specification assumes that DIDs are primarily public identifiers affiliated with known entities and that the affiliated DID Documents and Service Endpoints are also public. This ignores an important use case where the affiliation between DID data and DID entity is not known at the time of DID creation but is to be claimed later, as would be the case in some services that are processing pseudonomusly identified data.  In this case DIDs would be generated on the fly. Thus operations on the DID keys such as hierarchical deterministic key generation which is not normally exposed as public may be extremely beneficial when shared amongst the entities in the processing chain as a way to manage the entailed proliferation of keys that may be all claimed later as a hierarchial group. The DIDs and associated generation operations may be shared amongst a group of more-or-less trusted entities that are involved in the processing chain. Following best practices data provenance, such as, signed at rest, requires key management of the DIDs. This key management is shared but private to the entities. 


## Operations

Possibles classes of operations that might be worth considering as as follows:

1) Operations on the public/private key pair affiliated with a DID

2) Operations that use the public/private key pair affiliated with a DID

3) Operations on the meta-data affiliated with a DID

4) Operations on the data affiliated with a DID based self-certifying data item

5) Operations with data associated with a DID path

Class 1 may be the most important. It has to do with key management. Class 2 is only important to the extent that it is used to support the other classes. Classes 3 and 4 are very similar, the difference being wether or not the meta-data is self-contained. Class 5 is less important but relevant.

## Key Management

The four main key management operations are:

* Reproduction
* Rotation
* Revocation
* Recovery

What is problematic for DIDs is that the key management must be decentralized. The client or the client's proxy, not the server, is reposible for the the key management operations.  Given that for each public key there is an affiliated private key that must be kept save and secure. The generation of large numbers of new DIDs (aka public/private key pairs) imposes a significant burden on the client. Indeed, client side key management may be the primary barrier to widespread adoption of DIDs.

### Hierachical Deterministic Key Generation

Reproduction has to do with the generation of new keys. One way to reduce the burden of tracking large numbers of public/private key pairs is to use a hierarchical deterministic key generation algorithm (HD Keys). The algorithm needs a master or root key pair and a chain code for each derived key. Then only the master key pair needs to be kept track of and only the master private key needs to be kept securely secret. The other private keys can be reproduced on the fly given the key generation algorithm and the chain code. An extended public key would include the chain code in its representation so that the associated private key can be derived by the holder of the master private key anytime the extended public key is presented. The DID spec does not have syntax or semantics for representing HD keys.

Unlike the DID Fragment identifier which is referencing a field in the DID Document, and HD Key path (chain code) is an operation on the master key itself. While it would be possible to put the chain code (HD Key path) into the DID Document and then reference it via the DID Fragment this adds complexity and unnecessary indirection. It would be cleaner to just have a way of specifying the chain code as a parameter to an HD path algorithm specification.  What comes immediately to mind is to use a Query Parameter. This seems a natural fit. Indeed other key management tasks that involve operations on a key directly could be accomodated in the same way. This seems to be a simplifying design solution. Provides flexibility that is a natural semantic extension of existing URL syntax.

Consequently we propose that the DID ABNF be changed to include a did-query section.


#### Revised DID ABNF with DID-Query

The current DID syntax (ABNF) is as follows:

```abnf
did-reference       = did [ "/" did-path ] [ "?" did-query ] [ "#" did-fragment ]
 did                = "did:" method ":" specific-idstring
 method             = 1*methodchar
 methodchar         = %x61-7A / DIGIT
 specific-idstring  = idstring *( ":" idstring )
 idstring           = 1*idchar
 idchar             = ALPHA / DIGIT / "." / "-"
 
 did-fragment       = *( pchar / "/" / "?" )
 did-query          = *( pchar / "/" / "?" )
```

The semantics are as follows. If the *did-path* is missing then the *did-query* provides parameters to operations on the key/pair associated with the DID. The query parameter name specifies which operation to perform and the value the argument for that operation. The DID specification could then provide a canonical reference of standard operations and their associated *did-query* parameter names and value formats.
When the *did-path* is provided then the *did-query* does not have any special meaning but is application specific.

For example:

```bash
did:rep:ABCDEFG=?chain=m/0'/1'/1/2
```

Where *chain* is HD Key generation using the BIP-32 chain code provided as the value of the parameter with the DID as the master key pair. This parses correctly as

```python
>>> urlsplit("did:rep:ABCDEFG=?chain=m/0'/1'/1/2")
SplitResult(scheme='did', netloc='', path='rep:ABCDEFG=', query="chain=m/0'/1'/1/2", fragment='')
```

A did-query could be used in concert with a did-fragment to combine meta-data from the DID-document with an operation on that meta-data.  

This makes it straightforward to extend the DID syntax and semantics in a straightforward, familiar, and compatible way.

### Revised DID ABNF with Extended DID Reference and DID Query

The DID specification includes a special case, that is, when the did-path is missing the did-fragment is relative to the DID Document. This special case is very limiting. The URL specification has much more freedom in specifying the hier-part path component of the URL. Indeed, a URL has five different syntax types for specifying the hier-part. These are lost in the DID specification. The service endpoints in the DID Document are then used to restore some of that functionality but at the cost of additional lookups for each service endpoint. Because external lookups are expensive, it is desirable, that there be a standard way to cache or otherwise manage the information that might be obtained at the service endpoints. This is important for self-certifying data in data intensive applications.

We propose restoring some of the lost flexibility or equivalently adding extensibility to the DID specification by changing the syntax and semantics for the the DID reference. The proposed change will enable more flexibility in how the *did:fragment* is used to reference meta-data. This will enable to better management of external lookups.  

The change is as follows:
The DID reference will now support additional semi-colon separated components to indicate meta-data paths. This provides a way of indicating a path to meta-data that might be cached differentially. Currently additional colon separated components are part of the idstring. Because the idstring also allows for period characters. Repurposing the colon does not lose any generality. The period can be used instead to provide namespacing within an idstring. We re-purpose the colon for meta-data path name-spacing.

The proposed revised ABNF for DIDs is as follows:


```abnf
did-reference       = did [ "/" did-path ] [ "?" did-query ] [ "#" did-fragment ]
 did                = "did:" method ":" specific-idstring *( ":" meta-path)
 method             = 1*methodchar
 methodchar         = %x61-7A / DIGIT
 specific-idstring  = idstring 
 meta-path          = idstring
 idstring           = 1*idchar
 idchar             = ALPHA / DIGIT / "." / "-"
 
 did-fragment       = *( pchar / "/" / "?" )
 did-query          = *( pchar / "/" / "?" )
```

The semantics are as follows:

If no metadata path component are provided after the idstring the the did-fragment refers to the default DID Document wherever that is. A metadata path of *:dd *is the same as the default DID Document. If the a meta-data path is provided then it refers to a predefined meta-data document given by the path. Thus the meta-data in a DID document could be split up into different documents with different lookup/caching behavior.

Example:

```bash
"did:rep:ABNCDEF=:recovery#shards/0"
```

```python
urlsplit("did:rep:ABNCDEF=:recovery#shards/0")
SplitResult(scheme='did', netloc='', path='rep:ABNCDEF=:recovery', query='', fragment='shards/0')
```

## Conclusion

The two proposed changes to the DID spec provide a significantly improved degree of flexibility and extensibility while maintaining familiarity and compatibility with existing URL parsing tools. What remains is to define specific semantics for did-query parameters and did metadata paths. The improved extensibility means that these use-case specific specifications can be left as future enhancements. 


