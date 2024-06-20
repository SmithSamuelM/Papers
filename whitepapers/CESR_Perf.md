# CESR Performance
 
[Samuel M. Smith Ph.D.](mailto:sam@samuelsmith.org)

Version 0.0.1 2024/04/14 Original 2024/04/14

## Introduction

full qualified Self-qualifying Self-typing Self-framing



Self-Sovereign Identifiers open reputation and identity system basics
Key management with pre-rotation [[1]][[2]].
KERI (Key Event Receipt Infrastructure) [[3]].


https://connect2id.com/products/nimbus-jose-jwt/examples/jwk-generation

```json
{
  "kty" : "EC",
  "crv" : "P-256",
  "x"   : "SVqB4JcUD6lsfvqMr-OKUNUphdNn64Eay60978ZlL74",
  "y"   : "lf0u0pMj4lGAzZix5u4Cm5CMQIgMNpkwy163wtKYVKI",
  "d"   : "0g5vAEKzugrXaRbgKG0Tj2qJ5lMP4Bezds1_sTybkfk"
}
```

Cose key
https://pycose.readthedocs.io/en/latest/pycose/keys/ec2.html

```python
key_attribute_dict = {
    'KTY': 'EC2',
     'CURVE': 'P_256',
    'ALG': 'ES256',
    'D': unhexlify(b'57c92077664146e876760c9520d054aa93c3afb04e306705db6090308507b4d3')}

>>> cose_key = CoseKey.from_dict(key_attribute_dict)

>>> #encode/serialize key
>>> serialized_key = cose_key.encode()
>>> serialized_key
b'\xa6\x01\x02\x03& \x01!X \xba\xc5\xb1\x1c\xad\x8f\x99\xf9\xc7+\x05\xcfK\x9e&\xd2D\xdc\x18\x9ftR(%Z!\x9a\x86\xd6\xa0\x9e\xff"X  \x13\x8b\xf8-\xc1\xb6\xd5b\xbe\x0f\xa5J\xb7\x80J:d\xb6\xd7,\xcf\xedko\xb6\xed(\xbb\xfc\x11~#X W\xc9 wfAF\xe8vv\x0c\x95 \xd0T\xaa\x93\xc3\xaf\xb0N0g\x05\xdb`\x900\x85\x07\xb4\xd3'


```

## Annotation

```python
def denot(ams):
    """De-annotate CESR stream

    Returns:
        denot (bytes):  deannotation of input annotated CESR stream

    Parameters:
        ams (str):  CESR annotated message stream text
    """
    oms = bytearray()
    lines = ams.splitlines()
    for line in lines:
        line = line.strip()
        front, sep, back = line.partition('#') # finde comment if any
        front = front.strip()  # non-commented portion strip white space
        if front:
            oms.extend(front.encode())

    return bytes(oms)
```


```bash

incoming = 
b'-FAtYKERICAAXicpEO6lMLcTbUhdpbQVXCh78MShuT_69th6tiZhEbAfPCj4DG9XhvcVryHjoIGcj5nK4sAE3oslQHWi4fBJre3NGwTQMAAAMAAB-LALDG9XhvcVryHjoIGcj5nK4sAE3oslQHWi4fBJre3NGwTQMAAA-LAAMAAA-LAA-LAA-LAA'

annotated = 
-FAt # Key Event Counter FixedMessageBodyGroup count=45 quadlets
  YKERICAA # 'v' version  Verser Tag7 proto=KERI vrsn=2.00
  Xicp # 't' message type Ilker Tag3 Ilk=icp
  EO6lMLcTbUhdpbQVXCh78MShuT_69th6tiZhEbAfPCj4 # 'd' SAID Diger Blake3_256 
  DG9XhvcVryHjoIGcj5nK4sAE3oslQHWi4fBJre3NGwTQ # 'i' AID Prefixer Ed25519 
  MAAA # 's' Number Short sn=0
  MAAB # 'kt' Tholder signing threshold=1
  -LAL # 'k' Signing Key List Counter NonTransReceiptCouples count=11 quadlets
    DG9XhvcVryHjoIGcj5nK4sAE3oslQHWi4fBJre3NGwTQ # key Verfer Ed25519
  MAAA # 'nt' Tholder rotation threshold=0
  -LAA # 'n' Rotation Key Digest List Counter NonTransReceiptCouples count=0 quadlets
  MAAA # 'bt' Tholder Backer (witness) threshold=0
  -LAA # 'b' Backer (witness)List Counter NonTransReceiptCouples count=0 quadlets
  -LAA # 'c' Config Trait List Counter NonTransReceiptCouples count=0 quadlets
  -LAA # 'a' Seal List Counter NonTransReceiptCouples count=0 quadlets


incoming = 
b'-FDCYKERICAAXicpEMEvSn0o6Iv2-3gInTDMMDTV0qQEfooM-yTzkj6Kynn6EMEvSn0o6Iv2-3gInTDMMDTV0qQEfooM-yTzkj6Kynn6MAAAMAAC-LAhDG9XhvcVryHjoIGcj5nK4sAE3oslQHWi4fBJre3NGwTQDK58m521o6nwgcluK8Mu2ULvScXM9kB1bSORrxNSS9cnDMOmBoddcrRHShSajb4d60S6RK34gXZ2WYbr3AiPY1M0MAAC-LAhEB9O4V-zUteZJJFubu1h0xMtzt0wuGpLMVj1sKVsElA_EMrowWRk6u1imR32ZNHnTPUtc7uSAvrchIPN3I8S6vUGEEbufBpvagqe9kijKISOoQPYFEOpy22CZJGJqQZpZEyPMAAD-LAhBG9XhvcVryHjoIGcj5nK4sAE3oslQHWi4fBJre3NGwTQBK58m521o6nwgcluK8Mu2ULvScXM9kB1bSORrxNSS9cnBMOmBoddcrRHShSajb4d60S6RK34gXZ2WYbr3AiPY1M0-LABXDND-LA8-RAuDG9XhvcVryHjoIGcj5nK4sAE3oslQHWi4fBJre3NGwTQMAAAEB9O4V-zUteZJJFubu1h0xMtzt0wuGpLMVj1sKVsElA_DK58m521o6nwgcluK8Mu2ULvScXM9kB1bSORrxNSS9cnMAABEMrowWRk6u1imR32ZNHnTPUtc7uSAvrchIPN3I8S6vUG-QAMMAAPEEbufBpvagqe9kijKISOoQPYFEOpy22CZJGJqQZpZEyP'

annotated = 
-FDC # Key Event Counter FixedMessageBodyGroup count=194 quadlets
  YKERICAA # 'v' version  Verser Tag7 proto=KERI vrsn=2.00
  Xicp # 't' message type Ilker Tag3 Ilk=icp
  EMEvSn0o6Iv2-3gInTDMMDTV0qQEfooM-yTzkj6Kynn6 # 'd' SAID Diger Blake3_256 
  EMEvSn0o6Iv2-3gInTDMMDTV0qQEfooM-yTzkj6Kynn6 # 'i' AID Prefixer Blake3_256 
  MAAA # 's' Number Short sn=0
  MAAC # 'kt' Tholder signing threshold=2
  -LAh # 'k' Signing Key List Counter NonTransReceiptCouples count=33 quadlets
    DG9XhvcVryHjoIGcj5nK4sAE3oslQHWi4fBJre3NGwTQ # key Verfer Ed25519
    DK58m521o6nwgcluK8Mu2ULvScXM9kB1bSORrxNSS9cn # key Verfer Ed25519
    DMOmBoddcrRHShSajb4d60S6RK34gXZ2WYbr3AiPY1M0 # key Verfer Ed25519
  MAAC # 'nt' Tholder rotation threshold=2
  -LAh # 'n' Rotation Key Digest List Counter NonTransReceiptCouples count=33 quadlets
    EB9O4V-zUteZJJFubu1h0xMtzt0wuGpLMVj1sKVsElA_ # key digest Diger Blake3_256
    EMrowWRk6u1imR32ZNHnTPUtc7uSAvrchIPN3I8S6vUG # key digest Diger Blake3_256
    EEbufBpvagqe9kijKISOoQPYFEOpy22CZJGJqQZpZEyP # key digest Diger Blake3_256
  MAAD # 'bt' Tholder Backer (witness) threshold=3
  -LAh # 'b' Backer (witness)List Counter NonTransReceiptCouples count=33 quadlets
    BG9XhvcVryHjoIGcj5nK4sAE3oslQHWi4fBJre3NGwTQ # AID Prefixer Ed25519N
    BK58m521o6nwgcluK8Mu2ULvScXM9kB1bSORrxNSS9cn # AID Prefixer Ed25519N
    BMOmBoddcrRHShSajb4d60S6RK34gXZ2WYbr3AiPY1M0 # AID Prefixer Ed25519N
  -LAB # 'c' Config Trait List Counter NonTransReceiptCouples count=1 quadlets
    XDND # trait Traitor Tag3 trait=DND
  -LA8 # 'a' Seal List Counter NonTransReceiptCouples count=60 quadlets
    -RAu # Seal Counter SealSourceTriples count=46 quadlets
      DG9XhvcVryHjoIGcj5nK4sAE3oslQHWi4fBJre3NGwTQMAAAEB9O4V-zUteZJJFubu1h0xMtzt0wuGpLMVj1sKVsElA_# seal Sealer SealEvent
        #  'i' = DG9XhvcVryHjoIGcj5nK4sAE3oslQHWi4fBJre3NGwTQ
        #  's' = 0
        #  'd' = EB9O4V-zUteZJJFubu1h0xMtzt0wuGpLMVj1sKVsElA_
      DK58m521o6nwgcluK8Mu2ULvScXM9kB1bSORrxNSS9cnMAABEMrowWRk6u1imR32ZNHnTPUtc7uSAvrchIPN3I8S6vUG# seal Sealer SealEvent
        #  'i' = DK58m521o6nwgcluK8Mu2ULvScXM9kB1bSORrxNSS9cn
        #  's' = 1
        #  'd' = EMrowWRk6u1imR32ZNHnTPUtc7uSAvrchIPN3I8S6vUG
    -QAM # Seal Counter SealSourceCouples count=12 quadlets
      MAAPEEbufBpvagqe9kijKISOoQPYFEOpy22CZJGJqQZpZEyP# seal Sealer SealTrans
        #  's' = f
        #  'd' = EEbufBpvagqe9kijKISOoQPYFEOpy22CZJGJqQZpZEyP


incoming = 
b'-FB6YKERICAAXixnEHeLJVa4LLNRRYVkLQsXHIDvllcmhDaahe5a_oMvXKePEMEvSn0o6Iv2-3gInTDMMDTV0qQEfooM-yTzkj6Kynn6MAABEMEvSn0o6Iv2-3gInTDMMDTV0qQEfooM-yTzkj6Kynn6-LBU-RAuDG9XhvcVryHjoIGcj5nK4sAE3oslQHWi4fBJre3NGwTQMAACEB9O4V-zUteZJJFubu1h0xMtzt0wuGpLMVj1sKVsElA_DK58m521o6nwgcluK8Mu2ULvScXM9kB1bSORrxNSS9cnMAAiEMrowWRk6u1imR32ZNHnTPUtc7uSAvrchIPN3I8S6vUG-QAMMABDEEbufBpvagqe9kijKISOoQPYFEOpy22CZJGJqQZpZEyP-RAXDMOmBoddcrRHShSajb4d60S6RK34gXZ2WYbr3AiPY1M0MACAEB9O4V-zUteZJJFubu1h0xMtzt0wuGpLMVj1sKVsElA_'

annotated = 
-FB6 # Key Event Counter FixedMessageBodyGroup count=122 quadlets
  YKERICAA # 'v' version  Verser Tag7 proto=KERI vrsn=2.00
  Xixn # 't' message type Ilker Tag3 Ilk=ixn
  EHeLJVa4LLNRRYVkLQsXHIDvllcmhDaahe5a_oMvXKeP # 'd' SAID Diger Blake3_256 
  EMEvSn0o6Iv2-3gInTDMMDTV0qQEfooM-yTzkj6Kynn6 # 'i' AID Prefixer Blake3_256 
  MAAB # 's' Number Short sn=1
  EMEvSn0o6Iv2-3gInTDMMDTV0qQEfooM-yTzkj6Kynn6 # 'p' prior SAID Diger Blake3_256 
  -LBU # 'a' Seal List Counter NonTransReceiptCouples count=84 quadlets
    -RAu # Seal Counter SealSourceTriples count=46 quadlets
      DG9XhvcVryHjoIGcj5nK4sAE3oslQHWi4fBJre3NGwTQMAACEB9O4V-zUteZJJFubu1h0xMtzt0wuGpLMVj1sKVsElA_# seal Sealer SealEvent
        #  'i' = DG9XhvcVryHjoIGcj5nK4sAE3oslQHWi4fBJre3NGwTQ
        #  's' = 2
        #  'd' = EB9O4V-zUteZJJFubu1h0xMtzt0wuGpLMVj1sKVsElA_
      DK58m521o6nwgcluK8Mu2ULvScXM9kB1bSORrxNSS9cnMAAiEMrowWRk6u1imR32ZNHnTPUtc7uSAvrchIPN3I8S6vUG# seal Sealer SealEvent
        #  'i' = DK58m521o6nwgcluK8Mu2ULvScXM9kB1bSORrxNSS9cn
        #  's' = 22
        #  'd' = EMrowWRk6u1imR32ZNHnTPUtc7uSAvrchIPN3I8S6vUG
    -QAM # Seal Counter SealSourceCouples count=12 quadlets
      MABDEEbufBpvagqe9kijKISOoQPYFEOpy22CZJGJqQZpZEyP# seal Sealer SealTrans
        #  's' = 43
        #  'd' = EEbufBpvagqe9kijKISOoQPYFEOpy22CZJGJqQZpZEyP
    -RAX # Seal Counter SealSourceTriples count=23 quadlets
      DMOmBoddcrRHShSajb4d60S6RK34gXZ2WYbr3AiPY1M0MACAEB9O4V-zUteZJJFubu1h0xMtzt0wuGpLMVj1sKVsElA_# seal Sealer SealEvent
        #  'i' = DMOmBoddcrRHShSajb4d60S6RK34gXZ2WYbr3AiPY1M0
        #  's' = 80
        #  'd' = EB9O4V-zUteZJJFubu1h0xMtzt0wuGpLMVj1sKVsElA_


incoming = 
b'-FCGYKERICAAXrotEDtBwgOB0uGrSMBJhOmnkRoCupjg-4sJApvOx04ujhKsEMEvSn0o6Iv2-3gInTDMMDTV0qQEfooM-yTzkj6Kynn6MAACEHeLJVa4LLNRRYVkLQsXHIDvllcmhDaahe5a_oMvXKePMAAC-LAhDH7p14xo09rob5cEupmo8jSDi35ZOGt1k4t2nm1C1A68DIAdqJzLWEwQbhXEMOFjvFVZ7oMCJP4XXDP_ILaTEBAQDKhYdMBeP6FoH3ajGJTf_4fH229rm_lTZXfYkfwGTMERMAAC-LAhEBvDSpcj3y0y9W2-1GzYJ85KEkDIPxu4y_TxAK49k7ciEEb97lh2oOd_yM3meBaRX5xSs8mIeBoPdhOTgVkd31jbECQTrhKHgrOXJS4kdvifvOqoJ7RjfJSsN3nshclYStgaMAAD-LALBG9XhvcVryHjoIGcj5nK4sAE3oslQHWi4fBJre3NGwTQ-LALBH7p14xo09rob5cEupmo8jSDi35ZOGt1k4t2nm1C1A68-LAA-LAA'

annotated = 
-FCG # Key Event Counter FixedMessageBodyGroup count=134 quadlets
  YKERICAA # 'v' version  Verser Tag7 proto=KERI vrsn=2.00
  Xrot # 't' message type Ilker Tag3 Ilk=rot
  EDtBwgOB0uGrSMBJhOmnkRoCupjg-4sJApvOx04ujhKs # 'd' SAID Diger Blake3_256 
  EMEvSn0o6Iv2-3gInTDMMDTV0qQEfooM-yTzkj6Kynn6 # 'i' AID Prefixer Blake3_256 
  MAAC # 's' Number Short sn=2
  EHeLJVa4LLNRRYVkLQsXHIDvllcmhDaahe5a_oMvXKeP # 'p' prior SAID Diger Blake3_256 
  MAAC # 'kt' Tholder signing threshold=2
  -LAh # 'k' Signing Key List Counter NonTransReceiptCouples count=33 quadlets
    DH7p14xo09rob5cEupmo8jSDi35ZOGt1k4t2nm1C1A68 # key Verfer Ed25519
    DIAdqJzLWEwQbhXEMOFjvFVZ7oMCJP4XXDP_ILaTEBAQ # key Verfer Ed25519
    DKhYdMBeP6FoH3ajGJTf_4fH229rm_lTZXfYkfwGTMER # key Verfer Ed25519
  MAAC # 'nt' Tholder rotation threshold=2
  -LAh # 'n' Rotation Key Digest List Counter NonTransReceiptCouples count=33 quadlets
    EBvDSpcj3y0y9W2-1GzYJ85KEkDIPxu4y_TxAK49k7ci # key digest Diger Blake3_256
    EEb97lh2oOd_yM3meBaRX5xSs8mIeBoPdhOTgVkd31jb # key digest Diger Blake3_256
    ECQTrhKHgrOXJS4kdvifvOqoJ7RjfJSsN3nshclYStga # key digest Diger Blake3_256
  MAAD # 'bt' Tholder Backer (witness) threshold=3
  -LAL # 'b' Backer (witness)Cut List Counter NonTransReceiptCouples count=11 quadlets
    BG9XhvcVryHjoIGcj5nK4sAE3oslQHWi4fBJre3NGwTQ # AID Prefixer Ed25519N
  -LAL # 'b' Backer (witness)Add List Counter NonTransReceiptCouples count=11 quadlets
    BH7p14xo09rob5cEupmo8jSDi35ZOGt1k4t2nm1C1A68 # AID Prefixer Ed25519N
  -LAA # 'c' Config Trait List Counter NonTransReceiptCouples count=0 quadlets
  -LAA # 'a' Seal List Counter NonTransReceiptCouples count=0 quadlets


incoming = 
b'-FDeYKERICAAXdipECQs0t3_GL7-B3q4kMU-qLeRCugTFjrxR15mxUwYWp8TECQs0t3_GL7-B3q4kMU-qLeRCugTFjrxR15mxUwYWp8TMAAA4AADA1s2c1s2c1s2-LAhDIR8GACw4z2GC5_XoReU4DMKbqi6-EdbgDZUAobRb8uVDN7WiKyjLLBTK92xayCuddZsBuwPmD2BKrl83h1xEUtiDOE5jmI9ktNSAddEke1rH2cGMDq4uYmyagDkAzHl5nfY4AADA1s2c1s2c1s2-LAhEKFoJ9Conb37zSn8zHLKP3YwHbeQiD1D9Qx0MagJ44DSEC7sCVf_rYJ_khIj7UdlzrtemP31TuHTPUsGjvWni8GZEHgewy_ymPxtSFwuX2KaI_mPmoIUkxClviX3f-M38kCDMAAD-LAhBIR8GACw4z2GC5_XoReU4DMKbqi6-EdbgDZUAobRb8uVBN7WiKyjLLBTK92xayCuddZsBuwPmD2BKrl83h1xEUtiBOE5jmI9ktNSAddEke1rH2cGMDq4uYmyagDkAzHl5nfY-LAA-LBI-RAuDIR8GACw4z2GC5_XoReU4DMKbqi6-EdbgDZUAobRb8uVMAADEKFoJ9Conb37zSn8zHLKP3YwHbeQiD1D9Qx0MagJ44DSDN7WiKyjLLBTK92xayCuddZsBuwPmD2BKrl83h1xEUtiMAAEEC7sCVf_rYJ_khIj7UdlzrtemP31TuHTPUsGjvWni8GZ-QAYMAAVEHgewy_ymPxtSFwuX2KaI_mPmoIUkxClviX3f-M38kCDMD4SEKFoJ9Conb37zSn8zHLKP3YwHbeQiD1D9Qx0MagJ44DSEMEvSn0o6Iv2-3gInTDMMDTV0qQEfooM-yTzkj6Kynn6'

annotated = 
-FDe # Key Event Counter FixedMessageBodyGroup count=222 quadlets
  YKERICAA # 'v' version  Verser Tag7 proto=KERI vrsn=2.00
  Xdip # 't' message type Ilker Tag3 Ilk=dip
  ECQs0t3_GL7-B3q4kMU-qLeRCugTFjrxR15mxUwYWp8T # 'd' SAID Diger Blake3_256 
  ECQs0t3_GL7-B3q4kMU-qLeRCugTFjrxR15mxUwYWp8T # 'i' AID Prefixer Blake3_256 
  MAAA # 's' Number Short sn=0
  4AADA1s2c1s2c1s2 # 'kt' Tholder signing threshold=['1/2', '1/2', '1/2']
  -LAh # 'k' Signing Key List Counter NonTransReceiptCouples count=33 quadlets
    DIR8GACw4z2GC5_XoReU4DMKbqi6-EdbgDZUAobRb8uV # key Verfer Ed25519
    DN7WiKyjLLBTK92xayCuddZsBuwPmD2BKrl83h1xEUti # key Verfer Ed25519
    DOE5jmI9ktNSAddEke1rH2cGMDq4uYmyagDkAzHl5nfY # key Verfer Ed25519
  4AADA1s2c1s2c1s2 # 'nt' Tholder rotation threshold=['1/2', '1/2', '1/2']
  -LAh # 'n' Rotation Key Digest List Counter NonTransReceiptCouples count=33 quadlets
    EKFoJ9Conb37zSn8zHLKP3YwHbeQiD1D9Qx0MagJ44DS # key digest Diger Blake3_256
    EC7sCVf_rYJ_khIj7UdlzrtemP31TuHTPUsGjvWni8GZ # key digest Diger Blake3_256
    EHgewy_ymPxtSFwuX2KaI_mPmoIUkxClviX3f-M38kCD # key digest Diger Blake3_256
  MAAD # 'bt' Tholder Backer (witness) threshold=3
  -LAh # 'b' Backer (witness)List Counter NonTransReceiptCouples count=33 quadlets
    BIR8GACw4z2GC5_XoReU4DMKbqi6-EdbgDZUAobRb8uV # AID Prefixer Ed25519N
    BN7WiKyjLLBTK92xayCuddZsBuwPmD2BKrl83h1xEUti # AID Prefixer Ed25519N
    BOE5jmI9ktNSAddEke1rH2cGMDq4uYmyagDkAzHl5nfY # AID Prefixer Ed25519N
  -LAA # 'c' Config Trait List Counter NonTransReceiptCouples count=0 quadlets
  -LBI # 'a' Seal List Counter NonTransReceiptCouples count=72 quadlets
    -RAu # Seal Counter SealSourceTriples count=46 quadlets
      DIR8GACw4z2GC5_XoReU4DMKbqi6-EdbgDZUAobRb8uVMAADEKFoJ9Conb37zSn8zHLKP3YwHbeQiD1D9Qx0MagJ44DS# seal Sealer SealEvent
        #  'i' = DIR8GACw4z2GC5_XoReU4DMKbqi6-EdbgDZUAobRb8uV
        #  's' = 3
        #  'd' = EKFoJ9Conb37zSn8zHLKP3YwHbeQiD1D9Qx0MagJ44DS
      DN7WiKyjLLBTK92xayCuddZsBuwPmD2BKrl83h1xEUtiMAAEEC7sCVf_rYJ_khIj7UdlzrtemP31TuHTPUsGjvWni8GZ# seal Sealer SealEvent
        #  'i' = DN7WiKyjLLBTK92xayCuddZsBuwPmD2BKrl83h1xEUti
        #  's' = 4
        #  'd' = EC7sCVf_rYJ_khIj7UdlzrtemP31TuHTPUsGjvWni8GZ
    -QAY # Seal Counter SealSourceCouples count=24 quadlets
      MAAVEHgewy_ymPxtSFwuX2KaI_mPmoIUkxClviX3f-M38kCD# seal Sealer SealTrans
        #  's' = 15
        #  'd' = EHgewy_ymPxtSFwuX2KaI_mPmoIUkxClviX3f-M38kCD
      MD4SEKFoJ9Conb37zSn8zHLKP3YwHbeQiD1D9Qx0MagJ44DS# seal Sealer SealTrans
        #  's' = 3e12
        #  'd' = EKFoJ9Conb37zSn8zHLKP3YwHbeQiD1D9Qx0MagJ44DS
  EMEvSn0o6Iv2-3gInTDMMDTV0qQEfooM-yTzkj6Kynn6 # 'di' Delegator AID Prefixer Blake3_256 


incoming = 
b'-FBaYKERICAAXdrtEKwDKG0L9pAMbzV2e31-I5ObiEfkptfs8VqXYiHGCL1vECQs0t3_GL7-B3q4kMU-qLeRCugTFjrxR15mxUwYWp8TMAABECQs0t3_GL7-B3q4kMU-qLeRCugTFjrxR15mxUwYWp8TMAAB-LALDJ0pLe3f2zGus0Va1dqWAnukWdZHGNWlK9NciJop9N4fMAAB-LALENX_LTL97uOSOkA1PEzam9vtmCLPprnbcpi71wXpmhFFMAAD-LALBIR8GACw4z2GC5_XoReU4DMKbqi6-EdbgDZUAobRb8uV-LALBJ0pLe3f2zGus0Va1dqWAnukWdZHGNWlK9NciJop9N4f-LAA-LAA'

annotated = 
-FBa # Key Event Counter FixedMessageBodyGroup count=90 quadlets
  YKERICAA # 'v' version  Verser Tag7 proto=KERI vrsn=2.00
  Xdrt # 't' message type Ilker Tag3 Ilk=drt
  EKwDKG0L9pAMbzV2e31-I5ObiEfkptfs8VqXYiHGCL1v # 'd' SAID Diger Blake3_256 
  ECQs0t3_GL7-B3q4kMU-qLeRCugTFjrxR15mxUwYWp8T # 'i' AID Prefixer Blake3_256 
  MAAB # 's' Number Short sn=1
  ECQs0t3_GL7-B3q4kMU-qLeRCugTFjrxR15mxUwYWp8T # 'p' prior SAID Diger Blake3_256 
  MAAB # 'kt' Tholder signing threshold=1
  -LAL # 'k' Signing Key List Counter NonTransReceiptCouples count=11 quadlets
    DJ0pLe3f2zGus0Va1dqWAnukWdZHGNWlK9NciJop9N4f # key Verfer Ed25519
  MAAB # 'nt' Tholder rotation threshold=1
  -LAL # 'n' Rotation Key Digest List Counter NonTransReceiptCouples count=11 quadlets
    ENX_LTL97uOSOkA1PEzam9vtmCLPprnbcpi71wXpmhFF # key digest Diger Blake3_256
  MAAD # 'bt' Tholder Backer (witness) threshold=3
  -LAL # 'b' Backer (witness)Cut List Counter NonTransReceiptCouples count=11 quadlets
    BIR8GACw4z2GC5_XoReU4DMKbqi6-EdbgDZUAobRb8uV # AID Prefixer Ed25519N
  -LAL # 'b' Backer (witness)Add List Counter NonTransReceiptCouples count=11 quadlets
    BJ0pLe3f2zGus0Va1dqWAnukWdZHGNWlK9NciJop9N4f # AID Prefixer Ed25519N
  -LAA # 'c' Config Trait List Counter NonTransReceiptCouples count=0 quadlets
  -LAA # 'a' Seal List Counter NonTransReceiptCouples count=0 quadlets

```

## References


[1]. Smith, Samuel M., Decentralized Autonomic Data (DAD) and the three R's of Key Management. RWOT6 https://github.com/WebOfTrustInfo/rwot6-santabarbara/blob/master/final-documents/DecentralizedAutonomicData.pdf

[1]: https://github.com/WebOfTrustInfo/rwot6-santabarbara/blob/master/final-documents/DecentralizedAutonomicData.pdf

[2]. Conway, et al, A DID for Everything. RWOT7. https://github.com/WebOfTrustInfo/rwot7-toronto/blob/master/final-documents/A_DID_for_everything.pdf

[2]: https://github.com/WebOfTrustInfo/rwot7-toronto/blob/master/final-documents/A_DID_for_everything.pdf

[3]. Smith, Samuel M., Key Event Receipt Infrastructure (KERI): Design and Build, http://arxiv.org/abs/1907.02143

[3]: http://arxiv.org/abs/1907.02143
