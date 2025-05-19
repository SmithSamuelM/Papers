# TSP CESR Implementation Details

The TSP is a tunneling routing protocol that is meant to be transparent to any other message protocol that is tunneled as a payload of the TSP. Viewed from the perspective of the tunneled protocol, the TSP is all overhead, so the TSP wants to be very efficient. 

The most efficient approach is to encode
TSP native fields in its wrappers and native payloads as CESR primitives and groups. To clarify, the TSP native payloads for control messages like hop and relationship formation are fixed field with exclusively CESR native field values. 

In contrast, all tunneled payload messages (i.e. non-native TSP payloads) are encoded as sniffable CESR streams. A sniffable CESR stream supports not only native CESR primitives and groups but interleaved JSON, CBOR, and MGPK serializations. This provides a composable compact universal serialization format for tunneled messages.

## VID Serialization in Open and Closed Mode

Because VIDs appear in multiple places in TSP wrappers, using a separate type field everywhere to indicate the type of VID would be verbose. Instead a given TSP implementation may operate on one of two
modes. These are `open` mode and `closed` mode. 

## VID fields in Open Mode

In `open` mode, all VIDs must be either URNs or DIDs encoded as variable-length CESR UTF-8 byte strings (bytes). The CESR variable-length UTF-8 encoded byte strings (bytes) use one of the following 6 CESR codes in Base64 Text domain:  

```
    Bytes_L0:             str = '4B'  # Byte String lead size 0
    Bytes_L1:             str = '5B'  # Byte String lead size 1
    Bytes_L2:             str = '6B'  # Byte String lead size 2
    Bytes_Big_L0:         str = '7AAB'  # Byte String big lead size 0
    Bytes_Big_L1:         str = '8AAB'  # Byte String big lead size 1
    Bytes_Big_L2:         str = '9AAB'  # Byte String big lead size 2
```
Each code prefix is followed by length characters encoded in Base64 but abstractly represented as a `#` character. For example, `4B##` has two length characters that indicate the length of the associated bytestring in quadlets in Base64 (Text mode) or triplets in Base2 (Binary mode). 
There are two families of codes, the small codes and the big codes. The difference is the
length of the encoded byte string in base64 quadlets or base2 triplets (24 bits of information). The small codes have two size characters that thereby support VID bytestrings up to `64^2-1 = 4095` triplets/quadlets.  The big codes have four size characters that thereby support VID bytestrings up to `64^4-1 = 16,777,215` triplets/quadlets. 

Because all CESR encoded primitives must align on 24-bit boundaries, variable-length byte strings are pre-padded with leading pad bytes of zero. This lead padding ensures that the padded byte string is
24-bit aligned prior to conversion to Base64. The padded bytestring is then converted to Base64, and the appropriate bytestring code is attached.  There is a separate code for each padding length. The 24-bit alignment is ensured with either 0,1, or 2 lead pad bytes. 

Let `rs` be the size in bytes of the VID bytes; moreover, let `ps` be the lead pad byte size. The number of lead pad bytes, `ps`, is computed as follows (Python syntax):  

```python
ps = (3 - (rs % 3)) % 3
```
When `ps` is 0, then either of the code prefixes `4B` or `7AAB` is used. When `ps` is 1, then either of the code prefixes `5B` or `8AAB` is used. When `ps` is 2, then either of the code prefixes `6B` or `9AAB` is used. 

When an open-mode VID field is not encoded with one of these codes, then the VID field is erroneous and the packet may be silently dropped.

### Example VID encoding in open-mode

Let the VID be a `did:webs` DID as follows:

`did:webs:example.com:EAco5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M`

The length of the corresponding byte string is 65, i.e., `rs = 65`. This as a lead pad size of 1, i.e. `ps = ``
This means that code prefix `4B` is indicated. With lead pad the total raw string size is 66. This is
exactly 22 triplets in length `22 = 66/3`. The two length characters for the code prefix are `AW` which is 22 in Base64.  The byte string with lead pad byte is converted to Base64 which gives:

`AGRpZDp3ZWJzOmV4YW1wbGUuY29tOkVBY281ZFU1V2pEcnhEQks0YjRIckY4Ml9yWWI2TVg2eHNlZ2pxNG4wWTdN`
The full CESR code prefix with length characters is then prefixed to the given fully qualified CESR encoded Base64 (Text domain) representation as follows:

`5BAWAGRpZDp3ZWJzOmV4YW1wbGUuY29tOkVBY281ZFU1V2pEcnhEQks0YjRIckY4Ml9yWWI2TVg2eHNlZ2pxNG4wWTdN`

The Base2 (Binary domain) CESR representation is obtained by decoding the Base64 above as follows:

`\xe4\x10\x16\x00did:webs:example.com:EAco5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M`

The substring `\xe4\x10\x16\x00` prefex above represents 4 non-ASCII bytes in escaped notation. The first three byes are the Base2 representation of the CESR code `4BAW` the fourth one is the single lead pad byte. 

One advantage of the CESR lead pad approach is apparent from the example.  It is easy to read the original DID bytestring at the tail end of the Base2 encoded CESR primitive. 

Encoding and decoding of a variable-length byte primitive is provided by the Texter class in the `keri.core.coring` module of the `keripy` reference implementation library.


## VID Fields in Closed Mode

The CESR encoding of the VID field in closed mode is whatever that particular closed mode decides to use by pre-agreement. These must be encoded as CESR primitives.

When the closed mode assumes that all VIDs are KERI native AIDs then each VID would appear as a standard KERI AID which is a CESR encoded Digest or Public Key.  No did:keri or did:webs namespace is needed.

### Example VID Encoding in Close Mode KERI

An AID in KERI is encoded as the appropriate CESR cryptographic primitive. Usually, this is the CESR-encoded Blake3 hash of the inception event for that AID. An AID of this type would appear as:

`EAco5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M`



## TSP Structure

A TSP wrapper can be modeled with three parts:  Head, Body, Tail.  

### Head
The Head includes all the information that indicates that it is a TSP wrapper. This includes a TSP ESSR wrapper group code with wrapper size.  The ESSR TSP protocol and version field, the source VID field, and the destination VID field. The head is always plain text. All TSP wrappers have the same head structure. This is by design so that the head contains no correlatable metadata besides the source and destination VIDs. 

#### TSP ESSR Wrapper

All wrappers start with a CESR count code. This makes it sniffable in a stream.

All wrappers start with a CESR count code. This makes it sniffable in a stream. 
The count code must be one of the two count codes for ESSR type packets. These are `-E##` for small ESSR packets or `--E#####` for big ESSR packets. The count value counts the wrapped quadlets/triplets up to but not including the attached attachment group with signatures. To clarify, the count value includes the wrapped payload, including any recursively nested ESSR wrappers with their attachment groups, but not the top-level attachment group.


The next field is the protocol type and version field.
The protocol type uses four characters and must be `TSP—` for a normative TSP protocol. Changing the protocol type to anything but `TSP-` enables staging and testing and experimentation with non-normative variants of the TSP protocol that use the same ESSR wrapper structure but may have different payload types. It also provides future proofing for the development of future features to the TSP protocol.

The version uses three Base64 characters. The first one provide the major version. The second two provide the minor version. For example, `AAB` represents a major version of `0` and a minor version of `01`. In dotted notation this would be `0.01`.  This provides for a total of 64 major versions and 4096 minor versions for each major version. 

The CESR primitive code for such a 7 character primitive is `Y`.
An example TSP protocol type/version field value is as follows:

`YTSP-AAB`

Examples of the head section of a TSP wrapper are as follows:

#### Example TSP Head in Open Mode

| TSP ESSR Wrapper | Protocol+Version  | Src VID |  Dst VID  |
|:--------:|:-------:|:------------|:------------|
| `-E##` | `YTSP-AAB` |`5BAWAG...klmn` | `5BAWAG...p3ZW` | 

#### Example TSP Head in Closed Mode

| TSP ESSR Wrapper | Protocol+Version  | Src VID |  Dst VID  |
|:--------:|:-------:|:------------|:------------|
| `-E##` | `YTSP-AAB` | `EAAABBB...` |  `EAAADDD...` |


### Body

The body may be encrypted. The plain text body must be one of the following TSP payload types.

#### TSP Payload Types

Native TSP payloads are also called TSP control message payloads. These have the following payload types:

|  TSP Payload Type | CESR Code | Protocol | Description |
|:------:|:------:|:--------:|:------------|
| `HOP` | `XHOP` |  TSP |    hop list tunneled payload  |
| `RFI` | `XRFI` | TSP | relationship formation invite payload |
| `RFA` | `XRFA` | TSP | relationship formation accept payload |
| `RFD` | `XRFD` | TSP | relationship formation decline payload  |
| `SCS` | `XSCS` | TSP | sniffable CESR stream as payload |
| `PAD` | `XPAD` | TSP |  padding payload to minimize metadata correlation based on departure and arrival times

The second column is the actual CESR encoding of the payload type field value. The CESR code for 3-character primitives is `X`. 

The body payload type field enables unambiguous parsing of the plain test payload body. Each payload body uses a fixed set of fields as determined by the payload code. This is more compact than using
a field-map structure with field labels. 

The actual field structure for each native TSP body payload will be defined below.

#### Padding Control Message Payloads

An observer of a TSP wrapper packet may use the packet size as correlatable metadata. To protect against this type of correlation, every TSP control message payload includes a trailing pad field. The purpose of the pad field is to pad the total packet size to some desired minimum length, regardless of the non-padded payload size. The value of the pad field is CESR encoded as variable-length byte string primitive. The raw pad bytes should be a random set of characters to minimize correlability. Pad fields use the same encoding as open-mode VIDs. 

The pad value primitive is one of the following variable-length byte string codes as follows:
```
Bytes_L0:             str = '4B'  # Byte String lead size 0
Bytes_L1:             str = '5B'  # Byte String lead size 1
Bytes_L2:             str = '6B'  # Byte String lead size 2
Bytes_Big_L0:         str = '7AAB'  # Byte String big lead size 0
Bytes_Big_L1:         str = '8AAB'  # Byte String big lead size 1
Bytes_Big_L2:         str = '9AAB'  # Byte String big lead size 2
```

For example, the eight-character Base64 string `abcdefgh` would be encoded in CESR as `4BACabcdefghi`. 

To reiterate, because the TSP control message body payloads are fixed field with no labels, the padding field must always be present. When no padding is desired, the value of the pad field is just the CESR encoded variable-length byte string of zero length and zero lead size, that is, `4BAA`.

#### Examples:
Empty Padding Content:
`4BAA`

Non-Empty Padding Content:
`4BACabcdefghi`


### Ciphertext Body

When the body payload is encrypted as ciphertext using an HPKE encryption algorithm. The raw encryption is then CESR encoded using one of the supported CESR variable-length encrypted primitive codes. For each HPKE encryption algorithm, a set of six codes are defined. Three small codes and three big codes. Each small(big) code uses a different code for a lead pad size of (0,1,2) bytes, respectively. 

A ciphertext body appears as one field in the ESSR wrapper.

The CESR codes the currently supported HPKE encryption algorithms are as follows:
```
X25519_Cipher_L0:     str = '4C'  # X25519 sealed box cipher bytes of sniffable stream plaintext lead size 0
X25519_Cipher_L1:     str = '5C'  # X25519 sealed box cipher bytes of sniffable stream plaintext lead size 1
X25519_Cipher_L2:     str = '6C'  # X25519 sealed box cipher bytes of sniffable stream plaintext lead size 2
X25519_Cipher_Big_L0: str = '7AAC'  # X25519 sealed box cipher bytes of sniffable stream plaintext big lead size 0
X25519_Cipher_Big_L1: str = '8AAC'  # X25519 sealed box cipher bytes of sniffable stream plaintext big lead size 1
X25519_Cipher_Big_L2: str = '9AAC'  # X25519 sealed box cipher bytes of sniffable stream plaintext big lead size 2
HPKEBase_Cipher_L0:     str = '4F'  # HPKE Base cipher bytes of sniffable stream plaintext lead size 0
HPKEBase_Cipher_L1:     str = '5F'  # HPKE Base cipher bytes of sniffable stream plaintext lead size 1
HPKEBase_Cipher_L2:     str = '6F'  # HPKE Base cipher bytes of sniffable stream plaintext lead size 2
HPKEBase_Cipher_Big_L0: str = '7AAF'  # HPKE Base cipher bytes of sniffable stream plaintext big lead size 0
HPKEBase_Cipher_Big_L1: str = '8AAF'  # HPKE Base cipher bytes of sniffable stream plaintext big lead size 1
HPKEBase_Cipher_Big_L2: str = '9AAF'  # HPKE Base cipher bytes of sniffable stream plaintext big lead size 2
HPKEAuth_Cipher_L0:     str = '4G'  # HPKE Auth cipher bytes of sniffable stream plaintext lead size 0
HPKEAuth_Cipher_L1:     str = '5G'  # HPKE Auth cipher bytes of sniffable stream plaintext lead size 1
HPKEAuth_Cipher_L2:     str = '6G'  # HPKE Auth cipher bytes of sniffable stream plaintext lead size 2
HPKEAuth_Cipher_Big_L0: str = '7AAG'  # HPKE Auth cipher bytes of sniffable stream plaintext big lead size 0
HPKEAuth_Cipher_Big_L1: str = '8AAG'  # HPKE Auth cipher bytes of sniffable stream plaintext big lead size 1 
HPKEAuth_Cipher_Big_L2: str = '9AAG'  # HPKE Auth cipher bytes of sniffable stream plaintext big lead size 2

```
The lead pad bytes are prepended to the raw encrypted data. This is then Base64 encoded, and the
CESR code with its length character(s) is then prepended. To clarify, the CESR code is not itself encrypted.

CESR codes for other sniffable stream HPKE encryption formats have yet to be defined

### Plaintext Body

The plaintext representation of the payload body appears as a single CESR group that starts with one of the two dedicated SPAC payload group codes. For small payloads, the code is  `-Z##`. For big payloads, the code is `--Z#####`. 

The embedded fields in the payload group always start with the payload type field, which is then followed by the source VID field. The source VID field is required to support the ESSR format when the payload group is encrypted.

See above for a table of the payload types.

#### Example Payload Front End

| TSP Payload Group |   Payload Type   |  Src VID |
|:--------:|:--------:|:-------|
| `-Z##` | `XPAD` | `5BAWAG...klmn` |

The count portion of the payload count codes `-Z##` or `--Z#####`, is computed as the number of following quadlets/triplets in the payload starting with the payload type field.


## Payloads by Type
### Hop Payload

The most complex payload type is the `HOP` payload. This is because a `HOP` payload includes a
nested ESSR message.

In the hop payload includes in order: the payload group code, the payload type field, the source VID field, the hop list group with zero or more hop VIDs, the pad field, and an embedded ESSR message as indicated by an ESSR group code. The hop list group code is `-J##`. 
If the hop list is empty, then the empty list group, `-JAA`, is provided.
If the embedded message is empty, then the empty ESSR group code, `-EAA`, is provided.

The count portion of the payload count codes `-Z##` or `--Z#####`, is computed as the number of following quadlets/triplets in the payload starting with the payload type field and includes the full size of the embedded ESSR message with its attached signature group.


#### Example Hop Payload Open Mode

The following is an example of HOP payload with two hops and an embedded ESSR message with an encrypted payload.

| TSP Payload Group |   Payload Type   |  Src VID | Hop List Group | Hop VID 0 | Hop VID 1 | Pad | TSP ESSR Wrapper | Protocol+Version  | Src VID |  Dst VID   |  Ciphertext Payload  | Attachment Group | Idx Sig Group | Signature |
|:--------:|:--------:|:-------|:------:|:------------|:----------|:--------|:-----:|:-----:|:------------|:-----:|:--------|:--------:|:-------:|:-----------|
| `-Z##` | `XHOP` | `5BAWAG...klmn` | `-J##` | `5BAWAG...zxyw`  | `5BAWAG...efgh`  | `4B##` | `-E##` | `YTSP-ABA` | `5BAWAG...rstu` | `5BAWAG...jklm` | `4C##CefH...`  | `-C##` | `-K##` | `AAEbw3...` |

#### Example Hop Payload Closed Mode

The following is an example of HOP payload with two hops and an embedded ESSR messagewith an encrypted payload.

| TSP Payload Group |   Payload Type   |  Src VID | Hop List Group |  Hop VID  |   Hop VID   | Pad | TSP ESSR Wrapper | Version  | Src VID | Dst VID   |  Ciphertext Payload  | Attachment Group | Idx Sig Group | Signature |
|:--------:|:--------:|:-------|:------:|:---------:|:--------|:---------|:-----:|:-----:|:-------|:-------|:------------|:--------:|:-------:|:-----------|
| `-Z##` | `XHOP` | `EChij...` | `-J##` |  `EDxyz...` |  `ECkel....` | `4B##` | `-E##` | `YTSP-ABA` |  `EBabc...` | `EAzmk...` | `4C##CefH...`  | `-C##` | `-K##` | `AAEbw3...` |


### Generic Tunneled Payloads as Sniffable CESR Streams.

When not used for TSP native control messages, the embedded payload of a TSP tunnel is entirely application-specific. Sniffable CESR streams can accommodate virtually any data format. The `SCS` for sniffable CESR Stream is the generic payload type meant to encapsulate application-specific payloads to be delivered by TSP. These would include any trust task payloads. In CESR, the `-A##` and `--A#####` group codes are meant for generic pipeline able groups of other group or primitive codes. This enables parseable delimitation of a perfectly generic payload.
The First three fields of the `SCS` payload are as defined above for all payload types. The next field is the pad field (see above). The last field is the embedded payload field as an encapsulated CESR stream. The stream is encapsulated as a generic CESR group with small size code `-A##` or large size code `--A#####`.

The count portion of the payload count codes `-Z##` or `--Z#####`, is computed as the number of following quadlets/triplets in the payload starting with the payload type field and includes the full size of the embedded CESR stream which includes the contents of the `-A##` or `--A#####` group.

#### Generic payload as sniffable CESR stream in  Open Mode
| TSP Payload Group |   Payload Type   | Source VID |  Pad | CESR Stream |
|:--------:|:--------:|:-------|:-------|:------------|
| `-Z##` | `XSCS` | `5BAWAG...klmn` | `4B##` | `-A##....` |


#### Generic payload as sniffable CESR stream in  Closed Mode
| TSP Payload Group |   Payload Type   | Source VID | Pad | CESR Stream |
|:--------:|:--------:|:-------|:-------|:------------|
| `-Z##` | `XSCS` | `EAcsr...` | `4B##` | `-A##....` | 



### Pad Payload

Packet size as well as the time-of-departure (TOD) and the time-of-arrival (TOD) at the final destination can be used as correlatable metadata by a surveillor to correlate source and final destination of routed packets. While the pad field in the TSP payloads enables a given source to pad all packets to the same length to minimize correlation on packet size it does not protect against TOD and TOA metadata correlation. 

One way to minimize TOD/TOA correlation is to whiten the stream of packets by injecting pad packets so that there is a uniform distribution of packets over time. When a source does not have any material data to send, it may send a steady stream of pad packets instead, so that a correlator always sees a uniform time distribution of packets. The `PAD` packet payload starts with the standard three fields for payloads and adds a final field that is a pad field. The pad field is described above.

The pad field in the `XPAD` payload type has the same semantics as the pad field in the other payload types. The Source VID is the source VID as per the ESSR format, i.e. encrypt source.

The count portion of the payload count codes `-Z##` or `--Z#####`, is computed as the number of following quadlets/triplets in the payload starting with the payload type field and includes the pad field.

#### Pad Payload in Open Mode
| TSP Payload Group |   Payload Type   | Source VID | Pad |
|:--------:|:--------:|:-------|:------------|
| `-Z##` | `XPAD` |  `5BAWAG...klmn` |  `4B##` |


#### Pad Payload in Closed Mode
| TSP Payload Group |   Payload Type   | Source VID | Pad |
|:--------:|:--------:|:-------|:------------|
| `-Z##` | `XPAD` | `EAfg_src_VID` |  `4B##` |



### Relationship Formation (Sub) Protocol

The relationship formation sub-protocol defines three body payload types. These are `RFI`, `RFA` and `RFD`.  All include a pad field. 

The TSP ESSR wrapper in which a relationship formation payload is embedded is normative for the relationship formation sub-protocol. This is because the source and destination VIDs of the wrapper 
are used in relationship formation.

The TSP wrapper uses an existing relationship for the source and destination VIDS. Let these be  `X0` and `Y0` respectively. 

Should `X0` wish to form a new relationship with `Y0` with a new VID that X0 controls, say `X1`. Then `X0` sends a TSP ESSR Wrapped packet to `Y0` with an `RFI` payload. The source VID in the `RFI` payload is `X0`.

The `RFI` payload serves as an invitation from `X0` to `Y0` to form a new relationship using `X1` provided as the new `iVID` in the `RFI` payload. `X1` is controlled by `X0`. This invitation must be signed by the key(s) for `X`1 so that `Y0` knows that the invite came from the controller of `X1` via `X0`. This signature is included in the `RFI` payload inside the TSP ESSR Wrapper. The source VID in the `RFI` payload is `X0`.

If `Y0` wishes to accept the invite, then `Y0` responds with an `RFA` payload wrapped in a TSP ESSR wrapper from `Y0` to `X0`.  This payload includes the `iSAID` IID from the `RFI` payload. 
The `RFA` payload serves as an acceptance by `Y0` of the invite to form a new relationship between `X1` and `Y1`, where `Y1` is the new `aVID` in the RFA payload. `Y1` is controlled by `Y0`. This acceptance must be signed by `Y1` so that `X0` knows that acceptance came from the controller of `Y1` via `Y0`. This signature is included in the `RFI` payload inside the TSP ESSR Wrapper. The source VID in the `RFA` payload is `Y0`.

If `Y0` chooses not to accept the invite, then `Y0` sends a TSP ESSR-wrapped packet from `Y0` to `X0`  with payload `RFD`. This payload includes the `iSAID` IID from the `RFI` payload. The source VID in the `RFA` payload is `Y0`.

The `iSAID` in the `RFI` payload is computed over the non-pad, non-signature portions of the `RFI` payload.
The `aSAID` in the `RFA` payload is computed over the non-pad, non-signature portions of the `RFA` payload

####  Replay Attack

A reply attack mechanism is not required as long as relationship formations are treated as idempotent actions. The pair of cryptographically derived VIDs in a relationship form a universally unique pairing. Therefore, the formation of the pairing can be treated as idempotent in that a replay attack is just treated as a redundant or duplicate formation attempt that does not change the state of the relationship once formed.  This means that a stale decline (`RFD`) is ignored for any previously accepted relationships.

#### Salty Nonce

The salty nonce field in the `RFI` and `RFA` payloads protects against a rainbow table attack that could correlate the embedded new VID to the SAID of the payload. This assumes that the unencrypted SAID of the payload or the Signature of the payload is/are leaked in some way, and the encrypted payload is also leaked. When this happens, the SaltyNonce field ensures that the encrypted payload includes enough entropy to prevent correlating the newly leaked VID to the leaked SAID or Signature.

#### Count Group Code Computation

The count portion of the payload count codes `-Z##` or `--Z#####`, is computed as the number of following quadlets/triplets in the payload starting with the payload type field and ending with the pad field (inclusive).


#### Computed Fields in the RFI Payload

The signature field is computed on the concatenation of the following fields in order: Payload Type, Source AID, RFI SAID, Salty Nonce, New Rel iAID. This concatenation is signed, and that signature becomes the value of the signature in the Idx Sig Group. The pad field is not included in the signature computation.

The RFI SAID field is calculated using the SAID protocol (which substitutes dummy characters in the SAID field itself) on the concatenation of the following fields in order: Payload Type, Source AID, RFI SAID, Salty Nonce, New Rel iAID. The pad field is not included in the SAID computation.

#### Computed Fields in the RFA Payload

The count portion of the payload count codes `-Z##` or `--Z#####`, is computed as the number of following quadlets/triplets in the payload starting with the payload type field and ending with the pad field (inclusive).

The signature field is computed on the concatenation of the following fields in order: Payload Type, Source AID, RFA SAID, Salty Nonce, RFI SAID, New Rel aAID. This concatenation is signed, and that signature becomes the value of the signature in the Idx Sig Group. The pad field is not included in the signature computation.

The RFA SAID field is calculated using the SAID protocol (which substitutes dummy characters in the SAID field itself) on the concatenation of the following fields in order: Payload Type, Source AID, RFA SAID, Salty Nonce, RFI SAID, New Rel aAID. The pad field is not included in the SAID computation.



#### Relationship Formation Invitation (RFI) Payload in Open Mode
| TSP Payload Group |   Payload Type   | Source VID | RFI SAID (RF IID)  | Salty Nonce| New Rel iVID  |  Idx Sig Group | Signature iVID | Pad |
|:--------:|:--------:|:-------|:-------|:-------|:-----------------|:---------|:---------|:-----------|
| `-Z##` | `XRFI` | `5BAWAG...klmn`  | `EBa...`  | `Abcd...` | `5BAWAG...wxyz`  | `-K##` | `AAEaz4...` |  `4B##`|

#### Relationship Formation Acceptance (RFA) Payload in Open Mode
| TSP Payload Group |   Payload Type   |  Source VID | RFA SAID  |  Salty Nonce | RFI SAID (RF IID)  | New Rel aVID   | Idx Sig Group | Signature aVID | Pad |
|:--------:|:--------:|:-------|:-------|:------|:-------|:-----------------|:---------|:---------|:------------|
| `-Z##` | `XRFA` |  `5BAWAG...rstu`  |  `EAz...`  | `Aevg...` |  `EBa...`  | `5BAWAG...mnop`   | `-K##` | `AAEbw3...` |  `4B##` |

#### Relationship Formation Decline (RFD) Payload in Open Mode
| TSP Payload Group |   Payload Type   | Source VID | RF IID  | Pad |
|:--------:|:--------:|:-------|:-------|:------------|
| `-Z##` | `XRFD` | `5BAWAG...rstu`  | `EBa...`  | `4B##` |

#### Relationship Formation Invitation (RFI) Payload in Closed Mode
| TSP Payload Group |   Payload Type   | Source VID | RFI SAID (RF IID)  | Salty Nonce | New Rel iVID   | Idx Sig Group | Signature iVID | Pad|
|:--------:|:--------:|:-------:|:-------|:-------|:----------------|:---------|:---------|:----------|
| `-Z##` | `XRFI` | `EAmnb...` | `EBa...` | `Azbef...` | `EArsa...` | `-K##` | `AAEbw3...` |  `4B##` |


#### Relationship Formation Acceptance (RFA) Payload in Closed Mode
| TSP Payload Group |   Payload Type   | Source VID | RFA SAID  | Salty Nonce |  RFI SAID (RF IID)  | New Rel aVID   | Idx Sig Group | Signature aVID | Pad|
|:--------:|:--------:|:-------|:-------|:-------|:-------|:-------------------|:---------|:---------|:------------|
| `-Z##` | `XRFA` | `EBcde...` | `ECh....` | `Aklmj...` | `EBa...`  | `EDab_new_aVID`  | `-K##` | `AAEbw3...` |  `4B##` |


### Relationship Formation Decline (RFD) Payload in Closed Mode
| TSP Payload Group |   Payload Type   | Source VID | RF IID  | Pad |
|:--------:|:--------:|:-------|:-------|:------------|
| `-Z##` | `XRFD` | `EBcde...` | `EBa...`  |  `4B##` |


### Tail

The Tail part of each ESSR wrapper consists the attached signature(s) for the source VID. The attachment is CESR encoded as an attachment group with an embedded indexed signature group with embedded indexed signatures. Other groups may be included in the attachment group but are not normative and may be ignored when processing the attachment group.



#### Example Tail

| Attachment Group | Idx Sig Group | Signature |
|:-----------:|:-----------:|:------------|
| `-C##` | `-K##` | `AACZ0j...` |



## Examples of TSP ESSR Wrappers with Hop Payloads


### TSP Wrapper with encrypted payload in Open Mode
With TSP Protocol+Version

| TSP ESSR Wrapper | Protocl+Version  |  Src VID  |  Dst VID  |  Ciphertext Payload  | Attachment Group | Idx Sig Group | Signature |
|:---------------:|:-------:|:-------|:------|:----------|:----------:|:---------:|:----------|
| `-E##` | `YTSP-AAB` | `5BAWAG...rstu` |  `5BAWAG...xyzw` | `4C##BacD...` | `-C##` | `-K##` | `AACZ0j...` |


#### Hop Payload with tunneled ESSR in Open Mode
| TSP Payload Group |   Payload Type   |  Src VID | Hop List Group |  Hop VID  |   Hop VID   | Pad | TSP ESSR Wrapper | Version  | Src VID | Dst VID   |  Ciphertext Payload  | Attachment Group | Idx Sig Group | Signature |
|:--------:|:--------:|:-------|:------:|:---------:|:--------|:---------|:-----:|:-----:|:-------|:-------|:------------|:--------:|:-------:|:-----------|
| `-Z##` | `XHOP` | `5BAWAG...rstu` | `-J##` |  `5BAWAG...abcd` |  `5BAWAG...efgh` | `4B##` | `-E##` | `YTSP-ABA` |  `5BAWAG...ijkl` | `5BAWAG...mnop` | `4C##CefH...`  | `-C##` | `-K##` | `AAEbw3...` |

#### ESSR Wrapper with encrypted payload in closed mode

| TSP ESSR Wrapper | Protocl+Version  |  Src VID  |  Dst VID  |  Ciphertext Payload  | Attachment Group | Idx Sig Group | Signature |
|:---------------:|:-------:|:-------|:------|:----------|:----------:|:---------:|:----------|
| `-E##` | `YTSP-AAB` | `EAbce...` |  `EDefg...`  | `4C##BacD...` | `-C##` | `-K##` | `AACZ0j...` |

#### Hop Payload with tunneled ESSR in closed Mode
| TSP Payload Group |   Payload Type   |  Src VID | Hop List Group |  Hop VID  |   Hop VID   | Pad | TSP ESSR Wrapper | Version  | Src VID | Dst VID   |  Ciphertext Payload  | Attachment Group | Idx Sig Group | Signature |
|:--------:|:--------:|:-------|:------:|:---------:|:--------|:---------|:-----:|:-----:|:-------|:-------|:------------|:--------:|:-------:|:-----------|
| `-Z##` | `XHOP` | `EAbce...`  | `-J##` |  `EAzei...` |  `ECkel....` | `4B##` | `-E##` | `YTSP-ABA` |  `EBcde...` | `EBkms..` | `4C##CefH...`  | `-C##` | `-K##` | `AAEbw3...` |


## Determining if the payload field value is a ciphertext CESR primitive or a plaintext CESR group

In any CESR stream, the rule is that all primitives and encapsulated groups that appear within an encapsulating group must appear in the same CESR domain as the group code for the encapsulating group. The two domains are   (TEXT=Base64 or BINARY=Base2). A stream may have interleaved groups that are in different domains at the top level but not nested within any given group.

The group codes are sniffable (the first tritet (3) of bits is unique relative to anything else that may appear at the top level (these other things include JSON, CBOR, MGPK, and Annotated streams). So, a parser can determine the start of a group and which domain the group appears in (TEXT or BINARY) merely by sniffing (peeking) at the first three bits of a stream at the top level.  Subsequently, when parsing the encapsulated contents of the group, the parser may assume unambiguously that all its contents appear in the same domain (TEXT or BINARY) as the encapsulating group (count) code. Each field of the contents is either a nested group or a primitive. 

All group codes start with the character `-` in the TEXT domain (Base64) and the bits `111110` in the BINARY domain (Base2). The parser only has to look at the first character when in the TEXT domain or the first six bits when in the Binary domain in order to determine if the payload field is a plain text sniffable group or an encrypted primitive. The parser can then dispatch the correct sub-parser for each case.

### When the TSP ESSR Wrapper that contains that payload field is in the TEXT Domain
The parser can read the first character of the payload field. If it's  `-`, then the payload is a plain text payload group that may be recursively parsed. If it's not  `-`, then the payload must be an encrypted sniffable CESR stream and must use one of the HPKE encryption codes shown above. This is because none of the codes above in the text domain start with `-`.

### When the TSP ESSR Wrapper that contains that payload field is in the Binary Domain
The parser can read the first two tritets (6 bits) of the payload field, and if it's `111110`, then the payload is a plain text payload group in the BINARY domain, and this may be recursively parsed. If it's not `111110`, then the payload must be an encrypted sniffable CESR stream primitive, but in the BINARY domain and must use one of the HPKE encryption codes above, but in the Binary Domain. This is because none of the codes shown above, when converted to the binary domain, start with `111110`.  


## KERI Extension to Payload Message Type Table

Additionally, a useful set of message types for those using the TSP with KERI is to tunnel KERI messages as TSP ESSR payloads. A convenient way to do this would be to have a TSP payload message type for each KERI message type. Because KERI message types are exactly three base 64 characters, we can seamlessly use the same exact message types for both. In addition, KERI message types are all lowercase. If we adopt the convention that native TSP message types be uppercase then we make it easy to avoid collisions.
For KERI messages using these TSP codes the message is the message body plus any attachments. So a parser needs to be capable of parsing the attachments as well.

This same convention can be used for the new ACDC version two message types (see ACDC Protocol Top Level) below.

Below is a proposed TSP Payload Message type table.

|  Code | Protocol | Description |
|:------:|:--------:|:------------|
| XHOP |  TSP |    hop list tunneled payload  |
| XRFI | TSP | relationship formation invite payload |
| XRFA | TSP | relationship formation accept payload |
| XRFD | TSP | relationship formation decline payload  |
| XSCS | TSP | sniffable CESR stream as payload |
| Xxip | KERI | exchange inception message as payload |
| Xexn | KERI | exchange message as payload |
| Xqry | KERI | query message as payload |
| Xrpy | KERI | reply message as payload |
| Xpro | KERI | prod message as payload |
| Xbar | KERI | bare message as payload |
| Xicp | KERI | inception message as payload |
| Xrot | KERI | rotation message as payload |
| Xixn | KERI | interaction message as payload |
| Xdip | KERI | delegated inception message as payload |
| Xdrt | KERI | delegated rotation message as payload |
| Xrip | ACDC | registry inception message as payload |
| Xupd | ACDC | registry state update message as payload |
| Xacd | ACDC | ACDC message as payload |
| Xsch | ACDC | ACDC schema section message as payload |
| Xatt | ACDC | ACDC attribute section message as payload |
| Xagg | ACDC | ACDC attribute aggregate section message as payload |
| Xedg | ACDC | ACDC edge section message as payload |
| Xrul | ACDC | ACDC rule section message as payload |

### Examples
#### KERI `exn` Payload
| TSP Payload Group |   Message Type   |  Src VID |  Pad | exn message|
|:--------:|:--------:|:-------|:------:|:---------|
| `-Z##` | `Xexn` |`EAABCD...` |  `4B##`| exn message |

