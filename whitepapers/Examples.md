
#### Private ACDC with compact and uncompacted variants

Consider the following composed schema that supports both compact and uncompacted variants of a private ACDC. The variants are supported by the `oneOf` composition operator in the schema. The ACDC is private due to the existence of UUID `u` fields that support partial disclosure via the most compact SAID algorithm. 

The schema is first defined by a Python dictionary. This is then serialized into a compact JSON (with no whitespace) to compute its SAID as the value of the `$id` field.  The schema, as shown below, includes this computed SAID but includes whitespace for better readability. When mapping from Python to JSON, the python booleans True and False map to true and false, and the Python None maps to null.

```json
{
  "$id": "E46jrVPTzlSkUPqGGeIZ8a8FWS7a6s4reAXRZOkogZ2A",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Example ACDC Schema",
  "description": "Example JSON Schema ACDC.",
  "credentialType": "ACDCExample",
  "type": "object",
   "required":
  [
    "v",
    "d",
    "u",
    "i",
    "rd",
    "s",
    "a",
    "e",
    "r"
  ],
  "properties":
  {
    "v":
    {
      "description": "ACDC version string",
      "type": "string"
    },
    "d":
    {
     "description": "ACDC SAID",
      "type": "string"
    },
    "u":
    {
     "description": "ACDC UUID",
      "type": "string"
    },
    "i":
    {
      "description": "Issuer AID",
      "type": "string"
    },
    "rd":
    {
      "description": "Registry SAID",
      "type": "string"
    },
    "s":
    {
      "description": "Schema Section",
      "oneOf":
      [
        {
          "description": "Schema Section SAID",
          "type": "string"
        },
        {
          "description": "Schema Detail",
          "type": "object"
        }
      ]
    },
    "a":
    {
      "description": "Attribute Section",
      "oneOf":
      [
        {
          "description": "Attribute Section SAID",
          "type": "string"
        },
        {
          "description": "Attribute Detail",
          "type": "object",
          "required":
          [
            "d",
            "u",
            "i",
            "score",
            "name"
          ],
          "properties":
          {
            "d":
            {
              "description": "Attribute Section SAID",
              "type": "string"
            },
            "u":
            {
             "description": "Attribute Section UUID",
              "type": "string"
            },
            "i":
            {
              "description": "Issuee AID",
              "type": "string"
            },
            "score":
            {
              "description": "Test Score",
              "type": "integer"
            },
            "name":
            {
              "description": "Test Taker Full Name",
              "type": "string"
            }
          },
          "additionalProperties": false
        }
      ]
    },
    "e":
    {
      "description": "Edge Section",
      "oneOf":
      [
        {
          "description": "Edge Section SAID",
          "type": "string"
        },
        {
          "description": "Edge Detail",
          "type": "object",
          "required":
          [
            "d",
            "u",
            "boss"
          ],
          "properties":
          {
            "d":
            {
              "description": "Edge Section SAID",
              "type": "string"
            },
            "u":
            {
             "description": "Edge Section UUID",
              "type": "string"
            },
            "boss":
            {
              "description": "Boss Edge",
              "type": "object",
              "required":
              [
                "d",
                "u",
                "n",
                "s",
                "w"
              ],
              "properties":
              {
                "d":
                {
                  "description": "Edge SAID",
                  "type": "string"
                },
                "u":
                {
                  "description": "Edge UUID",
                  "type": "string"
                },
                "n":
                {
                  "description": "Far Node SAID",
                  "type": "string"
                },
                "s":
                {
                  "description": "Far Node Schema SAID",
                  "type": "string",
                  "const": "EiheqcywJcnjtJtQIYPvAu6DZAIl3MORH3dCdoFOLe71"
                },
                "w":
                {
                  "description": "Edge Weight",
                  "type": "string"
                }
              },
              "additionalProperties": false
            }
          },
          "additionalProperties": false
        }
      ]
    },
    "r":
    {
      "description": "Rule Section",
      "oneOf":
      [
        {
          "description": "Rule Section SAID",
          "type": "string"
        },
        {
          "description": "Rule Detail",
          "type": "object",
          "required":
          [
            "d",
            "u",
            "warrantyDisclaimer",
            "liabilityDisclaimer"
          ],
          "properties":
          {
            "d":
            {
              "description": "edge section SAID",
              "type": "string"
            },
            "u":
            {
              "description": "Rule Section UUID",
              "type": "string"
            },
            "warrantyDisclaimer":
            {
              "description": "Warranty Disclaimer Clause",
              "type": "object",
              "required":
              [
                "d",
                "u",
                "l"
              ],
              "properties":
              {
                "d":
                {
                  "description": "Clause SAID",
                  "type": "string"
                },
                "u":
                {
                  "description": "Clause UUID",
                  "type": "string"
                },
                "l":
                {
                  "description": "Legal Language",
                  "type": "string"
                }
              },
              "additionalProperties": false
            },
            "liabilityDisclaimer":
            {
              "description": "Liability Disclaimer Clause",
              "type": "object",
              "required":
              [
                "d",
                "u",
                "l"
              ],
              "properties":
              {
                "d":
                {
                  "description": "Clause SAID",
                  "type": "string"
                },
                "u":
                {
                  "description": "Clause UUID",
                  "type": "string"
                },
                "l":
                {
                  "description": "Legal Language",
                  "type": "string"
                }
              },
              "additionalProperties": false
            }
          },
          "additionalProperties": false
        }
      ]
    }
  },
  "additionalProperties": false
}
```


Public uncompacted variant

```json
{
  "v":  "ACDC10JSON00011c_",
  "d":  "EBdXt3gIXOf2BBWNHdSXCJnFJL5OuQPyM5K0neuniccM",
  "u":  "0ANghkDaG7OY1wjaDAE0qHcg",
  "i":  "EmkPreYpZfFk66jpf3uFv7vklXKhzBrAqjsKAn2EDIPM",
  "rd": "EMwsxUelUauaXtMxTfPAMPAI6FkekwlOjkggtymRy7x",
  "s":  "E46jrVPTzlSkUPqGGeIZ8a8FWS7a6s4reAXRZOkogZ2A",
  "a":
  {
    "d": "EgveY4-9XgOcLxUderzwLIr9Bf7V_NHwY1lkFrn9y2PY",
    "u":  "0ANghkDaG7OY1wjaDAE0qHcg",
    "i": "EpZfFk66jpf3uFv7vklXKhzBrAqjsKAn2EDIPmkPreYA",
    "score": 96,
    "name": "Jane Doe"
  },
  "e":
  {
    "d": "EerzwLIr9Bf7V_NHwY1lkFrn9y2PgveY4-9XgOcLxUdY",
    "boss":
    {
      "d": "E9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NHwY1lkFrn",
      "u":  "0ANghkDaG7OY1wjaDAE0qHcg",
      "n": "EIl3MORH3dCdoFOLe71iheqcywJcnjtJtQIYPvAu6DZA",
      "s": "EiheqcywJcnjtJtQIYPvAu6DZAIl3MORH3dCdoFOLe71",
      "w": "high"
    }
  },
  "r":
  {
    "d": "EwY1lkFrn9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NA",
    "warrantyDisclaimer":
    {
      "d": "EXgOcLxUdYerzwLIr9Bf7V_NAwY1lkFrn9y2PgveY4-9",
      "u":  "0ANghkDaG7OY1wjaDAE0qHcg",
      "l": "Issuer provides this credential on an \"AS IS\" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied, including, without limitation, any warranties or conditions of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A PARTICULAR PURPOSE"
    },
    "liabilityDisclaimer":
    {
      "d": "EY1lkFrn9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NAw",
      "u":  "0ANghkDaG7OY1wjaDAE0qHcg",
      "l": "In no event and under no legal theory, whether in tort (including negligence), contract, or otherwise, unless required by applicable law (such as deliberate and grossly negligent acts) or agreed to in writing, shall the Issuer be liable for damages, including any direct, indirect, special, incidental, or consequential damages of any character arising as a result of this credential. "
    }
  }
}
```


Public compact variant

```json
{
  "v":  "ACDC10JSON00011c_",
  "d":  "EBdXt3gIXOf2BBWNHdSXCJnFJL5OuQPyM5K0neuniccM",
  "i":  "EmkPreYpZfFk66jpf3uFv7vklXKhzBrAqjsKAn2EDIPM",
  "rd": "EMwsxUelUauaXtMxTfPAMPAI6FkekwlOjkggtymRy7x",
  "s":  "E46jrVPTzlSkUPqGGeIZ8a8FWS7a6s4reAXRZOkogZ2A",
  "a":  "EgveY4-9XgOcLxUderzwLIr9Bf7V_NHwY1lkFrn9y2PY",
  "e":  "ERH3dCdoFOLe71iheqcywJcnjtJtQIYPvAu6DZIl3MOA",
  "r":  "Ee71iheqcywJcnjtJtQIYPvAu6DZIl3MORH3dCdoFOLB"
}
```

#### Compact public ACDC example

An example of a fully compact public ACDC is shown below. The ACDC is public because the `u` field is missing.

```json
{
  "v":  "ACDC10JSON00011c_",
  "d":  "EBdXt3gIXOf2BBWNHdSXCJnFJL5OuQPyM5K0neuniccM",
  "i":  "EFk66jpf3uFv7vklXKhzBrAqjsKAn2EDIPMmkPreYpZf",
  "rd": "EMwsxUelUauaXtMxTfPAMPAI6FkekwlOjkggtymRy7x",
  "s":  "E46jrVPTzlSkUPqGGeIZ8a8FWS7a6s4reAXRZOkogZ2A",
  "a":  "EgveY4-9XgOcLxUderzwLIr9Bf7V_NHwY1lkFrn9y2PY",
  "e":  "ERH3dCdoFOLe71iheqcywJcnjtJtQIYPvAu6DZIl3MOA",
  "r":  "Ee71iheqcywJcnjtJtQIYPvAu6DZIl3MORH3dCdoFOLB"
}
```

The schema for the Compact public ACDC example above is provided below.

```json
{
  "$id": "EBdXt3gIXOf2BBWNHdSXCJnFJL5OuQPyM5K0neuniccM",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Compact Public ACDC",
  "description": "Example JSON Schema for a Compact private ACDC.",
  "credentialType": "CompactPublicACDCExample",
  "version": "1.0.0",
  "type": "object",
  "required":
  [
    "v",
    "d",
    "u",
    "i",
    "rd",
    "s",
    "a",
    "e",
    "r"
  ],
  "properties":
  {
    "v":
    {
      "description": "ACDC version string",
      "type": "string"
    },
    "d":
    {
     "description": "ACDC SAID",
      "type": "string"
    },
    "i":
    {
      "description": "Issuer AID",
      "type": "string"
    },
    "rd":
    {
      "description": "Registry SAID",
      "type": "string"
    },
    "s": 
    {
      "description": "schema SAID",
      "type": "string"
    },
    "a": 
    {
      "description": "attribute SAID",
      "type": "string"
    },
    "e": 
    {
      "description": "edge SAID",
      "type": "string"
    },
    "r": 
    {
      "description": "rule SAID",
      "type": "string"
    }
  },
  "additionalProperties": false
}
```


#### Compact private ACDC example.

An example of a fully compact private ACDC is shown below. The ACDC is private because the `u` field is non-empty.

```json
{
  "v":  "ACDC10JSON00011c_",
  "d":  "EBdXt3gIXOf2BBWNHdSXCJnFJL5OuQPyM5K0neuniccM",
  "u":  "0ANghkDaG7OY1wjaDAE0qHcg",
  "i":  "EmkPreYpZfFk66jpf3uFv7vklXKhzBrAqjsKAn2EDIPM",
  "rd": "EMwsxUelUauaXtMxTfPAMPAI6FkekwlOjkggtymRy7x",
  "s":  "E46jrVPTzlSkUPqGGeIZ8a8FWS7a6s4reAXRZOkogZ2A",
  "a":  "EgveY4-9XgOcLxUderzwLIr9Bf7V_NHwY1lkFrn9y2PY",
  "e":  "ERH3dCdoFOLe71iheqcywJcnjtJtQIYPvAu6DZIl3MOA",
  "r":  "Ee71iheqcywJcnjtJtQIYPvAu6DZIl3MORH3dCdoFOLB"
}

```

The schema for the Compact private ACDC example above is provided below.

```json
{
  "$id": "EBdXt3gIXOf2BBWNHdSXCJnFJL5OuQPyM5K0neuniccM",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Compact Private ACDC",
  "description": "Example JSON Schema for a Compact private ACDC.",
  "credentialType": "CompactPrivateACDCExample",
  "version": "1.0.0",
  "type": "object",
  "required":
  [
    "v",
    "d",
    "u",
    "i",
    "rd",
    "s",
    "a",
    "e",
    "r"
  ],
  "properties":
  {
    "v":
    {
      "description": "ACDC version string",
      "type": "string"
    },
    "d":
    {
     "description": "ACDC SAID",
      "type": "string"
    },
    "u":
    {
     "description": "ACDC UUID",
      "type": "string"
    },
    "i":
    {
      "description": "Issuer AID",
      "type": "string"
    },
    "rd":
    {
      "description": "Registry SAID",
      "type": "string"
    },
    "s": 
    {
      "description": "schema SAID",
      "type": "string"
    },
    "a": 
    {
      "description": "attribute SAID",
      "type": "string"
    },
    "e": 
    {
      "description": "edge SAID",
      "type": "string"
    },
    "r": 
    {
      "description": "rule SAID",
      "type": "string"
    }
  },
  "additionalProperties": false
}
```




#### Edge Section Examples

Consider four different ACDCs in compact form (see below):
The Issuer of the first is named Amy with AID, `EmkPreYpZfFk66jpf3uFv7vklXKhzBrAqjsKAn2EDIPM`.
The Issuer of the second is named Bob with AID, `EFk66jpf3uFv7An2EDIPMvklXKhmkPreYpZfzBrAqjsK`
The Issuer of the third is named Cal with AID, `EDIPMvklXKhmkPreYpZfzBrAqjsKFk66jpf3uFv7An2E`.
The Issuer of the fourth is named Deb with AID, `EAqjsKFk66jpf3uFv7An2EDIPMvklXKhmkPreYpZfzBr`. 

Notice that the AID of each Issuer appears in the Issuer, `i` field of its respective ACDC below.

Issued by Amy:

```json
{
  "v":  "ACDCCAAJSONAACD.",
  "d":  "EBdXt3gIXOf2BBWNHdSXCJnFJL5OuQPyM5K0neuniccM",
  "u":  "0ANghkDaG7OY1wjaDAE0qHcg",
  "i":  "EmkPreYpZfFk66jpf3uFv7vklXKhzBrAqjsKAn2EDIPM",
  "rd": "EMwsxUelUauaXtMxTfPAMPAI6FkekwlOjkggtymRy7x",
  "s":  "E46jrVPTzlSkUPqGGeIZ8a8FWS7a6s4reAXRZOkogZ2A",
  "a":  "EgveY4-9XgOcLxUderzwLIr9Bf7V_NHwY1lkFrn9y2PY",
  "e":  "EFOLe71iheqcywJcnjtJtQIYPvAu6DZIl3MOARH3dCdo",
  "r":  "Ee71iheqcywJcnjtJtQIYPvAu6DZIl3MORH3dCdoFOLB"
}
```

Issued by Bob:

```json
{
  "v":  "ACDCCAAJSONAACD.",
  "d":  "ECJnFJL5OuQPyM5K0neuniccMBdXt3gIXOf2BBWNHdSX",
  "u":  "0AG7OY1wjaDAE0qHcgNghkDa",
  "i":  "EFk66jpf3uFv7An2EDIPMvklXKhmkPreYpZfzBrAqjsK",
  "rd": "EMwsxUelUauaXtMxTfPAMPAI6FkekwlOjkggtymRy7x",
  "s":  "EGeIZ8a8FWS7a6s4reAXRZOkogZ2A46jrVPTzlSkUPqG",
  "a":  "EBf7V_NHwY1lkFrn9y2PYgveY4-9XgOcLxUderzwLIr9",
  "r":  "EMORH3dCdoFOLBe71iheqcywJcnjtJtQIYPvAu6DZIl3"
}
```

Issued by Cat:

```json
{
  "v":  "ACDCCAAJSONAACD.",
  "d":  "EK0neuniccMBdXt3gIXOf2BBWNHdSXCJnFJL5OuQPyM5",
  "u":  "0ADAE0qHcgNghkDaG7OY1wja",
  "i":  "EDIPMvklXKhmkPreYpZfzBrAqjsKFk66jpf3uFv7An2E",
  "rd": "EMwsxUelUauaXtMxTfPAMPAI6FkekwlOjkggtymRy7x",
  "s":  "EFWS7a6s4reAXRZOkogZ2A46jrVPTzlSkUPqGGeIZ8a8",
  "a":  "EIr9Bf7V_NHwY1lkFrn9y2PYgveY4-9XgOcLxUderzwL",
  "r":  "EBe71iheqcywJcnjtJtQIYPvAu6DZIl3MORH3dCdoFOL"
}
```

Issued by Deb:

```json
{
  "v":  "ACDCCAAJSONAACD.",
  "d":  "EBWNHdSXCJnFJL5OuQPyM5K0neuniccMBdXt3gIXOf2B",
  "u":  "0AHcgNghkDaG7OY1wjaDAE0q",
  "i":  "EAqjsKFk66jpf3uFv7An2EDIPMvklXKhmkPreYpZfzBr",
  "rd": "EMwsxUelUauaXtMxTfPAMPAI6FkekwlOjkggtymRy7x",
  "s":  "EAXRZOkogZ2A46jrVPTzlSkUPqGGeIZ8a8FWS7a6s4re",
  "a":  "EFrn9y2PYgveY4-9XgOcLxUderzwLIr9Bf7V_NHwY1lk",
  "r":  "EH3dCdoFOLBe71iheqcywJcnjtJtQIYPvAu6DZIl3MOR"
}
```



##### Two Edges

Suppose that the Edge Section of the ACDC issued by Amy, when expanded, has two Edges labeled `poe` for proof-of-entitlement and `data`.  The `poe` Edge links back to the ACDC issued by Bob's AID and the `data` Edge links back to the ACDC issued by Cat's AID. 

Edge section expanded:

```json
{
  "e":
  {
    "d": "EFOLe71iheqcywJcnjtJtQIYPvAu6DZIl3MOARH3dCdo",
    "u": "0AwjaDAE0qHcgNghkDaG7OY1",
    "work":
    {
      "d": "E2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NHwY1lkFrn9y",
      "u": "0ANghkDaG7OY1wjaDAE0qHcg",
      "n": "ECJnFJL5OuQPyM5K0neuniccMBdXt3gIXOf2BBWNHdSX",
      "s": "ELIr9Bf7V_NHwY1lkFrn9y2PgveY4-9XgOcLxUdYerzw"
    },
    "play":
    {
      "d": "ELxUdYerzwLIr9Bf7V_NHwY1lkFrn9y2PgveY4-9XgOc",
      "u": "0ADAE0qHcgNghkDaG7OY1wja",
      "n": "EK0neuniccMBdXt3gIXOf2BBWNHdSXCJnFJL5OuQPyM5",
      "s": "EHwY1lkFrn9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_N",
      "o": "NI2I"
    }
  }
}
```

Edge section (compact private edges):

```json
{
  "e":
  {
    "d": "EFOLe71iheqcywJcnjtJtQIYPvAu6DZIl3MOARH3dCdo",
    "u": "0AwjaDAE0qHcgNghkDaG7OY1",
    "poe": "E2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NHwY1lkFrn9y",
    "data": "ELxUdYerzwLIr9Bf7V_NHwY1lkFrn9y2PgveY4-9XgOc"
  }
}
```

Edge section subschema:

```json
"e":
{
  "description": "edge section",
  "oneOf":
  [
    {
      "description": "edge section SAID",
      "type": "string"
    },
    {
      "description": "edge section detail",
      "type": "object",
      "required":
      [
        "d",
        "u",
        "poe",
        "data"
      ],
      "properties":
      {
        "d":
        {
          "description": "edge section SAID",
          "type": "string"
        },
        "u":
        {
          "description": "edge section UUID",
          "type": "string"
        },
        "poe":
        {
          "description": "proof of entitlement edge",
          "oneOf":
          [
            {
              "description": "compact form edge detail SAID",
              "type": "string"
            },
            {
              "description": "edge detail",
              "type": "object",
              "required":
              [
                "d",
                "u",
                "n",
                "s"
              ],
              "properties":
              {
                "d":
                {
                  "description": "edge SAID",
                  "type": "string"
                },
                "u":
                {
                  "description": "edge UUID",
                  "type": "string"
                },
                "n":
                {
                  "description": "far node SAID",
                  "type": "string"
                },
                "s":
                {
                  "description": "far node schema SAID",
                  "type": "string",
                  "const": "ELIr9Bf7V_NHwY1lkFrn9y2PgveY4-9XgOcLxUdYerzw",
                }
              },
              "additionalProperties": false
            }
          ]
        },
        "data":
        {
          "description": "data edge",
          "oneOf":
          [
            {
              "description": "compact form edge detail SAID",
              "type": "string"
            },
            {
              "description": "data edge detail",
              "type": "object",
              "required":
              [
                "d",
                "u",
                "n",
                "s",
                "o"
              ],
              "properties":
              {
                "d":
                {
                  "description": "edge SAID",
                  "type": "string"
                },
                "u":
                {
                  "description": "edge UUID",
                  "type": "string"
                },
                "n":
                {
                  "description": "far node SAID",
                  "type": "string"
                },
                "s":
                {
                  "description": "far node schema SAID",
                  "type": "string",
                  "const": "EHwY1lkFrn9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_N",
                },
                "o":
                {
                  "description": "unary edge operator",
                  "type": "string"
                },
              },
              "additionalProperties": false
            },
          ]
        }
      },
      "additionalProperties": false
    }
  ]
}
```

Notice that the SAID, `d` field value in the Edge Section (top-level Edge-group) block is the same as the value of the Edge Section, `e` field in the ACDC issued by Amy. Also, notice that the Node, `n` field value of the `bob` Edge block is the value of the SAID, `d` field of the ACDC issued by Bob, and the Node, `n` field value of the `cat` Edge block is the value of the SAID, `d` field of the ACDC issued by Cat. Further, notice that the top-level Edge-group of the ACDC issued by Amy has no Operator field. This means that the default m-ary operator `AND` is applied. Therefore Amy's ACDC is invalid unless both the linked ACDCs issued by Bob and Cal are valid. Moreover, notice that the Edge block labeled `poe` in Amy's ACDC has no operator, `o` field. This means that the default unary Operator `I2I` is applied. This means that Bob's ACDC must designate Amy's AID as the Issuee in order for this edge to be valid. Finally, notice that the Edge block labeled `data` in Amy's ACDC has an Operator, `o` field value of `NI2I`. This means that there is no requirement that Cat's ACDC designates Amy's AID as the Issuee in order for this Edge to be valid. If it does, fine; if not, also fine.

Suppose that the expanded Attribute section of the ACDC issued by Bob is as follows:

```json
"a":
{
  "d": "EgveY4-9XgOcLxUderzwLIr9Bf7V_NHwY1lkFrn9y2PY",
  "u": "0ADAE0qHcgNghkDaG7OY1wja",
  "i": "EmkPreYpZfFk66jpf3uFv7vklXKhzBrAqjsKAn2EDIPM",
}
```

Because the value of the Issuee, `i`, field in Bob's Attribute section is Amy's AID, the default `I2I` Operator on Amy's Edge labeled `poe` is satisfied. Thus, Amy's ACDC validates with respect to its Edges.

Both Edges can be individually compacted and private because they include both `d` and `u` fields. The Schema allows this compact Edge form with a `oneOf` composition on each of the Edges. Notice that in the compact Edge form the value of each labeled Edge field is the SAID, `d` field value of its expanded form.  To elaborate, the Edge section can be expressed in one of three forms. These are:
- compact private form, as a whole, because its schema uses the `oneOf` composition.
- partially expanded form with compact private Edges because each Edge's subschema uses the `oneOf` composition.
- fully expanded form with fully expanded Edge blocks because of the combination of `oneOf` compositions at both the section and Edge levels.

##### Nested edge group
 
In contrast to the previous example, this example shows a nested Edge-group in the ACDC issued by Amy. Amy's Edge Section when expanded, has three Edges labeled `poe`, `sewer`, and `gas` as shown below, where each of these labeled Edges links back to the ACDCs issued respectively by Bob's, Cat's, and Deb's AIDs. The nested Edge-group has label `poa` for proof-of-address. Some of the field values in the compact version of the ACDC issued by Amy must change because the Edge section and Schema are both different.

Issued by Amy:

```json
{
  "v":  "ACDCCAAJSONAACD.",
  "d":  "EHdSXCJnBWNFJL5OuQPyM5K0neunicIXOf2BcMBdXt3g",
  "u":  "0ADaG7OY1wjaDAE0qHcgNghk",
  "i":  "EmkPreYpZfFk66jpf3uFv7vklXKhzBrAqjsKAn2EDIPM",
  "rd": "EMwsxUelUauaXtMxTfPAMPAI6FkekwlOjkggtymRy7x",
  "s":  "EFWS7a6s4reAXRZOkogZ2A46jrVPTzlSkUPqGGeIZ8a8",
  "a":  "EgveY4-9XgOcLxUderzwLIr9Bf7V_NHwY1lkFrn9y2PY",
  "e":  "EJtQIYPvAu6DZIl3MOARH3dCdoFOLe71iheqcywJcnjt",
  "r":  "Ee71iheqcywJcnjtJtQIYPvAu6DZIl3MORH3dCdoFOLB"
}
```


Edge section:
```json
{
  "e":
  {
    "d": "EJtQIYPvAu6DZIl3MOARH3dCdoFOLe71iheqcywJcnjt",
    "u": "0AwjaDAE0qHcgNghkDaG7OY1",
    "work":
    {
      "d": "E2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NHwY1lkFrn9y",
      "u": "0ANghkDaG7OY1wjaDAE0qHcg",
      "n": "ECJnFJL5OuQPyM5K0neuniccMBdXt3gIXOf2BBWNHdSX",
      "s": "ELIr9Bf7V_NHwY1lkFrn9y2PgveY4-9XgOcLxUdYerzw"
    },
    "play":
    {
      "d": "E2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NHwY1lkFrn9y",
      "u": "0ANghkDaG7OY1wjaDAE0qHcg",
      "o": "OR",
      "hard":
      {
        "d": "ELxUdYerzwLIr9Bf7V_NHwY1lkFrn9y2PgveY4-9XgOc",
        "u": "0ADAE0qHcgNghkDaG7OY1wja",
        "n": "EK0neuniccMBdXt3gIXOf2BBWNHdSXCJnFJL5OuQPyM5",
        "s": "EHwY1lkFrn9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_N",
        "o": "NI2I"
      },
      "easy":
      {
        "d": "EHwY1lkFrn9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_N",
        "u": "0AAE0qHcgNghkDaG7OY1wjaD",
        "n": "EBWNHdSXCJnFJL5OuQPyM5K0neuniccMBdXt3gIXOf2B",
        "s": "EFrn9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NHwY1lk",
      }
    }
  }
}
```

Edge section subschema:

```json
"e":
{
  "description": "edge section",
  "oneOf":
  [
    {
      "description": "edge section SAID",
      "type": "string"
    },
    {
      "description": "edge detail",
      "type": "object",
      "required":
      [
        "d",
        "u",
        "poe",
        "poa"
      ],
      "properties":
      {
        "d":
        {
          "description": "edge section SAID",
          "type": "string"
        },
        "u":
        {
          "description": "edge section UUID",
          "type": "string"
        },
        "poe":
        {
          "description": "entitlement edge",
          "type": "object",
          "required":
          [
            "d",
            "u",
            "n",
            "s"
          ],
          "properties":
          {
            "d":
            {
              "description": "edge SAID",
              "type": "string"
            },
            "u":
            {
              "description": "edge UUID",
              "type": "string"
            },
            "n":
            {
              "description": "far node SAID",
              "type": "string"
            },
            "s":
            {
              "description": "far node schema SAID",
              "type": "string",
              "const": "ELIr9Bf7V_NHwY1lkFrn9y2PgveY4-9XgOcLxUdYerzw",
            }
          },
          "additionalProperties": false
        },
        "poa":
        {
          "description": "proof of address group",
          "type": "object",
          "required":
          [
            "d",
            "u",
            "o",
            "sewer",
            "gas"
          ],
          "properties":
          {
            "d":
            {
              "description": "edge SAID",
              "type": "string"
            },
            "u":
            {
              "description": "edge UUID",
              "type": "string"
            },
            "o":
            {
              "description": "m-ary group operator",
              "type": "string",
              "const": "OR"
            },
            "sewer":
            {
              "description": "sewer address edge",
              "type": "object",
              "required":
              [
                "d",
                "u",
                "n",
                "s"
              ],
              "properties":
              {
                "d":
                {
                  "description": "edge SAID",
                  "type": "string"
                },
                "u":
                {
                  "description": "edge UUID",
                  "type": "string"
                },
                "n":
                {
                  "description": "far node SAID",
                  "type": "string"
                },
                "s":
                {
                  "description": "far node schema SAID",
                  "type": "string",
                  "const": "EHwY1lkFrn9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_N",
                },
              },
              "additionalProperties": false
            },
            "gas":
            {
              "description": "gas address edge",
              "type": "object",
              "required":
              [
                "d",
                "u",
                "n",
                "s"
              ],
              "properties":
              {
                "d":
                {
                  "description": "edge SAID",
                  "type": "string"
                },
                "u":
                {
                  "description": "edge UUID",
                  "type": "string"
                },
                "n":
                {
                  "description": "far node SAID",
                  "type": "string"
                },
                "s":
                {
                  "description": "far node schema SAID",
                  "type": "string",
                  "const": "EFrn9y2PgveY4-9XgOcLxUdYerzwLIr9Bf7V_NHwY1lk",
                },
              },
              "additionalProperties": false
            }
          },
          "additionalProperties": false
        },
      },
      "additionalProperties": false
    }
  ]
}
```

Notice that the SAID, `d` field value in the Edge Section (top-level Edge-group) block is the same as the value of the Edge Section, `e` field in the ACDC issued by Amy. Also, notice that the Node, `n` field value of the `poe` Edge block is the value of the SAID, `d` field of the ACDC issued by Bob, the Node, `n` field value of the `sewer` Edge block is the value of the SAID, `d` field of the ACDC issued by Cat, and the Node, `n` field value of the `gas` Edge block is the value of the SAID, `d` field of the ACDC issued by Deb. Further, notice that the top-level Edge-group of the ACDC issued by Amy has no Operator field. This means that the default m-ary Operator `AND` is applied. This top-level Edge-group includes one Edge labeled `poe` and a nested Edge-group labeled `poa`. This nested Edge group has two Edges labeled `sewer` and `gas`. The Edge-group's Operator, `o` field value is `OR`. This means that the Edge-group is valid if either of its Edges is valid. The unary Operators on the `poe`, `sewer`, and `gas` edges are the default `I2I` because the Operator, `o` field is missing in each of the associated Edge blocks. This means that each of the ACDCs issued by Bob, Cat, and Deb must designate Amy's AID as the Issuee in order for the associated Edge to be valid. But as long as the `poe` Edge is valid, only one of the Edges, `sewer` or `gas`, must be valid for Amy's ACDC to be valid with respect to its Edges.

To clarify, with this version of the Edge Section, Amy's ACDC is valid with respect to its Edges if the ACDC issued by Bob is valid, and either Cat's or Deb's ACDCs are valid.  Amy's Edge section with nested Edge-group provides a sub-graph with an `AND-OR` logic tree on its three Edges. This is suitable for many types of business logic, such as KYC, for example, where the combination of a proof of entitlement (`poe`) and a proof of one of two types of addresses (`sewer` or `gas`) is required.

The three Edges and the nested Edge-group could each be individually compacted and private because they include both `d` and `u` fields. To simplify the example, however, the `oneOF` composition was not applied to the individual Edges and nested Edge group. Therefore, the simplified Schema only allows the expanded form of the individual Edges and nested Edge group.  Nonetheless, the Edge section, as a whole can be expressed in compact private form because its Schema uses the `oneOf` composition. 

##### Compact public edge section example

Suppose instead Amy is not concerned about privacy at either the section or the individual Edge and Edge group level. Amy therefore could benefit from using an expanded Edge Section that is more compact. Furthermore,  Amy's ACDC may not benefit from specifying a different Schema constraint on the far nodes of its Edges. Therefore, compared to the example above, several fields could be eliminated. These include all the `u` fields, all but the top-level Edge Section `d` field, and all the `s` fields.

Issued by Amy:

```json
{
  "v":  "ACDCCAAJSONAACD.",
  "d":  "EBWNFJL5OuQPyM5K0neunicIXOf2BcMBdXt3gHdSXCJn",
  "u":  "0AG7OY1wjaDAE0qHcgNghkDa",
  "i":  "EmkPreYpZfFk66jpf3uFv7vklXKhzBrAqjsKAn2EDIPM",
  "rd": "EMwsxUelUauaXtMxTfPAMPAI6FkekwlOjkggtymRy7x",
  "s":  "EGGeIZ8a8FWS7a6s4reAXRZOkogZ2A46jrVPTzlSkUPq",
  "a":  "EgveY4-9XgOcLxUderzwLIr9Bf7V_NHwY1lkFrn9y2PY",
  "e":  "EFOLe71iheqcywJcnjtJtQIYPvAu6DZIl3MOARH3dCdo",
  "r":  "Ee71iheqcywJcnjtJtQIYPvAu6DZIl3MORH3dCdoFOLB"
}
```


Edge section (simple compact edges):
```json
{
  "e":
  {
    "d": "EFOLe71iheqcywJcnjtJtQIYPvAu6DZIl3MOARH3dCdo",
    "poe": "ECJnFJL5OuQPyM5K0neuniccMBdXt3gIXOf2BBWNHdSX",
    "poa":
    {
      "o": "OR",
      "sewer": "EK0neuniccMBdXt3gIXOf2BBWNHdSXCJnFJL5OuQPyM5",
      "gas": "EBWNHdSXCJnFJL5OuQPyM5K0neuniccMBdXt3gIXOf2B"
    }
  }
}
```

Edge section subschema:

```json
"e":
{
  "description": "edge section",
  "oneOf":
  [
    {
      "description": "edge section SAID",
      "type": "string"
    },
    {
      "description": "edge detail",
      "type": "object",
      "required":
      [
        "d",
        "poe",
        "poa"
      ],
      "properties":
      {
        "d":
        {
          "description": "edge section SAID",
          "type": "string"
        },
        "poe":
        {
          "description": "entitlement edge",
          "oneOf":
          [
            {
              "description": "simple compact form far node SAID",
              "type": "string"
            },
            {
              "description": "edge detail",
              "type": "object",
              "required":
              [
                "n",
              ],
              "properties":
              {
                "n":
                {
                  "description": "far node SAID",
                  "type": "string"
                }
              },
              "additionalProperties": false
            }
          ]
        },
        "poa":
        {
          "description": "proof of address group",
          "type": "object",
          "required":
          [
            "o",
            "sewer",
            "gas"
          ],
          "properties":
          {
            "o":
            {
              "description": "m-ary group operator",
              "type": "string",
              "const": "OR"
            },
            "sewer":
            {
              "description": "sewer address edge",
              "oneOf":
              [
                {
                  "description": "simple compact form far node SAID",
                  "type": "string"
                },
                {
                  "description": "edge detail",
                  "type": "object",
                  "required":
                  [
                    "n",
                  ],
                  "properties":
                  {
                    "n":
                    {
                      "description": "far node SAID",
                      "type": "string"
                    }
                  },
                  "additionalProperties": false
                }
              ]
            },
            "gas":
            {
              "description": "gas address edge",
              "oneOf":
              [
                {
                  "description": "simple compact form far node SAID",
                  "type": "string"
                },
                {
                  "description": "edge detail",
                  "type": "object",
                  "required":
                  [
                    "n",
                  ],
                  "properties":
                  {
                    "n":
                    {
                      "description": "far node SAID",
                      "type": "string"
                    }
                  },
                  "additionalProperties": false
                }
              ]
            }
          },
          "additionalProperties": false
        },
      },
      "additionalProperties": false
    }
  ]
}
```

Notice how much more compact is the Edge section in partially expanded form. As before, notice that the SAID, `d` field value in the Edge Section (top-level Edge-group) block is the same as the value of the Edge Section, `e` field in the ACDC issued by Amy. Also, notice that the value of the `poe` field is the value of the SAID, `d` field of the ACDC issued by Bob. This is the simple compact form of an Edge described above. Likewise, for the `sewer` field value and the `gas` field value which are respectively the value of the SAID, `d` field of the ACDCs issued by Cal and Deb. All the Edges and nested Edge-groups are public because they include a `u` field. The schema uses the `oneOf` composition operator on all three Edges. This indicates that the compact form is simple compact form because their expanded block form only includes a Node, `n` field and not a SAID, `d` field.

Otherwise, this example's semantics are the same as the previous example, just more compact.

##### Edge Examples Summary

As the examples above have shown, the Edge Section syntax enables the composable and extensible but efficient expression of aggregating (m-ary) and unary Operators both singly and in combination as applied to nestable groups of Edges.  The intelligent defaults for the Operator, `o`, field, namely, `AND` for m-ary Operators and `I2I` for unary Operators, means that in many use cases, the Operator, `o`, field, does not even need to be present. This syntax is sufficiently general in scope to satisfy the contemplated use cases, including those with more advanced business logic. 


