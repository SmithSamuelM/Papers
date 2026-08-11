# ACDC IPEX Negotiation

There are several mechanisms that could be used to negotiate a graduate disclosure. These include: attribute path lists, aggregate block label lists, metadata ACDCs, Schema, and Decompositions of Schema.   

The problem with negotiating with schema docompositions is that attributes appear disjoint in a schema so pathing to indicate an attribute gets complicated quickly and the pathing logic has to understand JSON schema syntax at a deep level.  Whereas in a metadata ACDC the attribute paths are not disjoint so indicating an attribute at any nesting level can be handled with a path string.   

So it feels like to me that negotiation should be using Composed Schema as type. So only the SAID of the schema is provided and assume the full schema is cached or attached. Then use pathing of attributes to request the level of partial disclsure and metadata ACDCs for terms negotiation prior to disclosure.

Desired attributes are requested as a list of paths of each attribute or block. If the path is a block then the whole block is meant to be disclosed. If the path is a field in a block only the fields in that block at that field level needs to be disclosed. Any fields whose value is a nested block in that block does not need to be expanded and disclosed.  

With nested partial disclosure in an ATT section in order to disclose any field in a block all the simple fields in that block must also be disclosed but non-simple fields i.e. fields whose value is a nested block need only disclose the SAID of the block. So specifiying any simple field via a path would get you all the simple fields in the block holding the specified leaf of the path.

Whereas specifying a block (directory) would be short hand for disclosing the full tree attached all the way down.

For ATT section ACDCs the pathing gets simple since its an array and only the unique field label needs to be specified to designate a given block.

For example
```
'a':
{
    'd': "Ebslde",
    'u': "Abldle"
    'i': "EDdecd",
    "grades":
    {
         "d": "Efghi",
         "u":  "Abdefs",    
         "gpa":  3.7,
        "transcript":
        {
             "d": "EBDE",
             "u":  "ACEF",    
               "math": 3.5,
              "english":3.8",
        }
     }
}
```

}

SAID of composed schema as type defines the ACDC in the apple: EABCDSS...
Attribute path list defines what attributes are being asked for: ['a/i', 'a/grades/']
So apply is asking for the top level of the "a" section via 'a/i'   or '/a/i' and for the full grades block and any nested subblocks of grades via 'a/grades/' or '/a/grades'
Or because grades is nested down from the top level of 'a'.  A path list of simply ["/a/grades/"

Where as a path list of  merely ['/a/grades/'] subsumes the other path list.

To get gpa (and the issuee aid) without the grades detail  would be  simply ['/a/grades/gpa']


In response to an offer, the chain to the apply could either reply with the same schema and path list, or the rule could be that an offer chained to the same apply that leaves the path list None implies it is offering the same path list.
and then the offer includes a metadata acdc that exposes the rule section and the att section said but not the pathed attributes.
The agree then chains to the offer which indicates agreement
The grant then provides an uncompacted version of the ACDC with the appropriate level of disclosure.


To generalize this to apply to those who want graduated disclosure of multiple ACDCs in a chain, I suggest a dict
where the item label is the SAID of the ACDC in the chain, and the value of the item is the path list of disclosed attributes.

The path list is not limited to attribute and aggregate sections; paths could include edge and rule section paths.

For example, two chained acdcs one with an attribute section and one with an aggregate section

{
"EABCDED": ["/a/gpa"],
"EBQRSTR" : ["/A/i","/A/photo"]
}

The path dict requests the gpa field in the ACDC with schema "EABCDED" which means that anything on the path to the "gpa" field must also be disclosed and also requests the selectively disclosable Issuee and photo blocks of the schema "EBQRSTR"



We cannot use ACDC SAIDs in an application in general because the actual ACDC SAID is assumed not to be known by the applicant. But for defined ACDCs, as would be the case for any ACDCs governed by an EGF, the  SAIDS of the ACDC schemas would be known. And so the request is for an unidentified ACDC but with a given schema SAID and the paths indicated by the path list.

When the ACDC disclosure includes a bespoke ACDC the ap
When the ACDC disclosure includes a bespoke ACDC, the applicant won't necessarily know the SAID of its schema in the application, but the initial offer can provide it via a bespoke metadata ACDC, and subsequent apply messages can negotiate on that bespoke ACDC schema SAID.


The paths shown use the "/' delimeter but for CESR compatibility we could use the CESR Pathing BASE64 compatible path delimeter "-".     Or if we  do a little work to name the path list  field normatively we could have the tooling auto convert from "/" delimited paths to more compact "-" delimited paths when using CESR serialization of the exchange message.
 Or we allow either delimeter and just inject and extract the path as a tuple for reasoning so we are delimiter independent at the reasoning level..
 
###  Pather for pathing

I agree that using the existing Pather makes the most sense. This does impose some subtle constraints on the serialization. Currently, pather is used in attachments to reason about the message to which the path is attached. The pathed material group code in the attachment tells the parser to interpret the encoded string as a path.

For generic exchange messages, field values that are strings do not know how to use pather serialization unless we make them special fields. So where we convert to a Pather is relevant. There are some choices and so more discussion before hardening is warranted.

Another related option not mentioned so far is which block in an exchange should request disclosure paths in an Apply go in. There is a query block and an attribute block. By design, there are two blocks so that the exchange (as well as the other message types, query and reply) can more easily model ReST endpoint logic to minimize the impedance mismatch for building KERI APIs. 

If one thinks of an http verb with a URL with path and query string and verb message body, the modeling is as follows. The route field in the exchange holds the path from the URL. The query block holds the query string as a field map, and the attribute block holds the verb message body.

So we can put disclosure request information in the apply in three places, the route, the query block, and the attribute block

The route prefix is /apply, but one could overload that and use /apply/schemasaid.
One could define a field in the query block to put a dictionary of paths
One could define a field in the apply block to put a dictionary of paths.

Because the vLEI did not employ graduated disclosure, there was no need to go to any lengths to define this stuff, but now we are discussing the syntax for negotiating over graduated disclosure and its worthwhile now  to expose these subtleties

### Reasoning with graphs vs reasoning with schema

The multi-ACDC dict is keyed by schema SAID ({"EABCDED": [...], "EBQRSTR": [...]}). That can't disambiguate a chain that contains two credentials of the same schema — two same-type creds collapse to one key. Since you already note the applicant can't know the actual ACDC SAIDs, keying by the edge/node label in the requester's own apply-schema (which the applicant does control) might be the more robust handle for chained disclosure.
You are right the simple example does not account for a chain with repeated schema. So we need to be more thoughtful in how we define it.

This opens up a bigger question, and your reference to the “apply-schema” itself brings in a related issue. This is an area where I have spent some significant effort. The W3C community wants to reason using knowledge graphs. A schema is a type of knowledge graph. A chained graph of ACDCs is also a type of knowledge graph. ACDCs can be modeled with labeled property graphs, which are also a type of knowledge graph.

The core insight for me is not whether we have the ability to reason about or with knowledge graphs, but what type of reasoning we are doing, and whether we should constrain the type of reasoning in a layering of concerns so as to better address the concerns at each layer. Knowledge graphs, as widely used in that community, do not, have as first-order design properties, any security, authenticity, verifiability, confidentiality, or privacy properties as layered concerns. They only model semantics, meaning. These non-semantic concepts are rarely employed, and when they are, they are bolt-ons to the standard knowledge graph.  I found the W3C approach of attempting to bolt on security to RDF graph expansions a pathological mess (excuse my strong language). What knowledge graphs are meant to do is reason about the semantics of what is in the graph, not reason about authenticity or correlatability as a precursor to what goes in a graph in the first place. These are different types of reasoning.

One of the prime motivations for ACDCs was to invert that layering. To build mechanisms for security, confidentiality, and correlation resistance in layers that are employed before the semantic reasoning happens.

So disclosure is reasoning about exposure. What parts to expose when. Not what those parts mean once they are exposed. So, to better separate concerns along a layering approach, my intuition is we should minimize the role the schema plays in this layer of reasoning, that is the what to expose layer. This is accomplished by using schema as type, i.e., the SAID of the schema, and the schema validation validates the type. This contraindicates using bespoke schema as the way to indicate exposure. And using schema to indicate exposure is more complicated because pathing in schema is disjoint and requires deep knowledge of the schema semantics to reason about exposure paths.

Reasoning about exposure of elements of a graph when those graph elements are expandable or  collapsable or compactifiable is reasoning about the graph structure not the semantics of the information contained in the graph structure. In this case, graduated disclosure, it seems to me we can reason about graph structure using a simplified expression of the graph, not the most complex expression of the graph (JSON Schema). So we should think about what the minimally sufficient means are for reasoning about graph structure not the most expressive means (JSON schema) for that graph structure we have available.

So if we think of an ordered dictionary as a graph, we can model a graph of ACDCs with a dictionary (associative array or field map; we already model the guts of each acdc with a dictionary), so we can extend that by modeling the full graph of ACDCs with an expanded dictionary. This allows us to use a pathing language that is as simple as we can make it. A path is a tuple of labels which can be compactly serialized to a string with a path delimiter.

It has to include a way to path along a chain of ACDCs not merely into an ACDC. As you point out, with the overly simple example I provided of a path list. But we could use an ordered path list to represent a branch in an ACDC chain (only one possible branch) or include the full edge labeling path from the leaf ACDC back up the tree when not a simple chain (one branch). We can have relative to ACDC paths and absolute paths that are paths through the aCDC graph.

The edge labels in the path enable one to explicitly trace which edge leads to which ACDC schema SAID and then path down into that ACDC to the disclosed leaf.

The insight is correct: a request to disclose that uses a path to a leaf implies a closure of all that must be disclosed in order to disclose the leaf, but no more. So compacted sibling branches stay compacted. But then we need a compact way of requesting full expansion, so I used the trailing “/“ to indicate that. This provides a compact way of saying expand all branches below this node.

A path to a leaf implies that the nodes along that path must be diclosed but not sibling branches and no branches below the node holding the leaf, but all leaves in the same node.  This is unambiguously implied by a leaf path.

Whereas a node path (trailing "/") already indicates more than a leaf. So we can let that indicate an expansion of the node and all children nodes but no sibling branches. I think this is reasonable. If we need siblinge branches then we specify a path to the sibling leaf or node.  The node path rule means if I want the whole graph then I specify a node path to the root node of the tree. If I want a subgraph then the node path to the top node of the sub graph.  If I only want specific leaves and this implies the nodes to get to the leaf then I use a leaf path. The leaf path is a scalpel the node path is a hammer.

 
 ## Clarifications
 
### Issuee AID for aggregative (selectively disclosable) ACDCs

I believe for the core SEDI identity credential, the appropriate choice is to use the selectively disclosable Aggregate section.  The state of Utah has not decided, but my suggestion is that it be selectively disclosable, given that most use cases seem to fit that model.  In many cases, the only disclosed field value would be the issuee AID field.

The "i" field for Issuee is optional. When not present, it means the ACDC is issued to whom it may concern (untargeted).

The operator I2I is intended to work for either att or agg ACDCs when an "i" field is present.

In the case where no "i" issuee aid field is provided, the I2I operator on an edge would fail. 

This is a case where the term "holder" is confusing, because anyone could "hold" and present an untargeted ACDC. Unlike W3C credentials, where the "holder" binding is variable, with both weak and strong forms. An ACDC either has a cryptographically verifiable binding (i.e. strongest) via an "i" field with the issuee aid or there is no cryptographically verifiable binding. It's a boolean concept.

To elaborate:
An attributive ACDC has an "a" section, and if its "a" section has an "i" field at its top level, then the value of that "i" field is the issuee AID.
An aggregative ACDC has an "A" section, and if the second block in its "A" section has an "i" field, then the value of the "i" field is the issuee AID.
An ACDC can only be either attributive or aggregative, not both (mutually exclusive)

The spec should be clarified to precisely describe where the 'i' field in an Agg section is found when present.  
The issuee in an ACDC with Agg section (selectively disclosable) is the "i" field in the second (index 1) attribute block. 

The SerderACDC class has two relevant properties that provide the issuer AID as appropriate for the type of ACDC (attributive or aggregative). The properties are the .iseaid and .issuee.  The .iseaid property returns the issuee aid or None (when the AID is untargeted). The .issuee property returns a Prefixer Matter primitive instance whose .qb64 property is the issuee aid or None.  For example, see these asserts in the test_selective_disclosure_aggregate_JSON()

```
    assert acdc.iseaid == BOB
    assert acdc.issuee.qb64 == BOB
```

### High Stakes Transactions

I think I misunderstood your question. I assumed you meant should a failed transaction that starts over be one or two transaction but you were asking if a single linear transaction should be one or two where the boundary is where the anchors occur.

For linear transactions I see no reason to split. What we are anchoring is the tail of the transaction not the head. Because the events in an exhange transaction are hash chained together, anchoring the tail also anchors the whole transation including the head.

Multiple anchors are not meant to anchor multiple transactions but as continencies for the transaction to proceed.
In a high-stakes transaction, the Discloser wants to see a perpetually verifiable commitment to the terms via the Agreement of the Disclosee before issuing a Grant to the agreeor. The Disclosee (agreeor, admitor) then wants to see that the Discloser (grantor) anchors the grant before producing an admit. Often there is mutual accountability in terms and conditions especially in high stakes transactions and so the Disclosee (recipient) wants to have perpetual verifiability of the Discloser's (grantor's) mutual committment before issuing its admit.

Why the admit, because in contract law, a potential point of legal enforcement friction is that the Disclosee claims they never received the grant. The admit message removes any plausible deniability by the Disclosee. The anchor of the Grant message is incentivized by the following admit. The grantor wants the admit but cant get it until they anchor their grant. The terms of the contract could include an escape clause for the grantor if there is no admit message which incentivizes the admit. And since it must be anchored, the admitor has a way to prove it later, to nullify the escape clause.

Typically high stakes transactions happen under the auspices of a trusted third party who ensures the transaction completes (settlment). But we want to have true peer-to-peer, legally enforceable transactions without trusted third parties, so we need a sequence of commitments that are are successively incentivized without a trusted third party to ensure the completion of the transaction (settlement).

So the Offer/Agree/Grant/Admit removes plausible deniability which is the bane of peer-to-peer interactions. This is related to the byzantine generals problem which requires a multi-phase commit to solve.

### Closure rules by section 

The grammar is uniform; the closure is not
This is a place I want to pick at. Your leaf-is-a-scalpel, node-is-a-hammer rule feels good. But what a leaf path closes over depend on the section, right?
Yes, I just haven't looked at the edge section specifically in light of this conversation to say that it's definitely a different closure, but I agree that it could be different, so we need to be more deliberate in defining what the closure is for each section.

ATT — the simple fields of a block share one SAID and one blinding nonce. You cannot reveal one without its simple siblings. So a leaf path closes over every simple field of its block, and nested sub-blocks stay SAIDs — exactly your rule.
Good

AGG — each element is blinded on its own, with its own d and u. A leaf to one element, say /A/over21, reveals that element and nothing beside it. There is no sibling closure, because there is no shared block.
That means the same syntax can have different closures. So the function that turns a path list into a disclosed set has to branch on section type. Make that the normative object — disclosureSet(graph, paths), computed identically on both sides. The path list is the seed; the disclosed set is what the function returns. Nowhere for two implementations to drift.
Yes I agree.
We need to extend that to the Edge and Rule sections. We can have blinded edges for example so a partial disclosure of an ACDC attached to a blinded edge potentially has its own combinatorics for closure logic. Likewise for blinded rule section legal language where the negotiation first uses open terms that enable disclosure of sensistive terms. This is often the case in domestic abuse situations where the legally enforceable terms would leak sensitive information so the verifier must first be put under confidentiality with respect to those sensitive terms before disclosing the sensitive terms.

Makes total sense. However, what are the expected consequences of a spurn at each of these stages? A spurn after offer is obvious. A spurn after agree just means the discloser backed out, right? But what about a spurn after grant? Is that just a simple "I didn't admit this after all, so I'm not bound by T&Cs" ?
No we want to avoid such assumption. An agree is legally binding not sending an admit does not convey backing out. The purpose of the admit is not to amend the agreement. That agree defines the agreement. The admit is a corner case that enables easy enforcement by removing plausible deniablity. Confidentiality is about how one came by the information. So without an admit, the agreeer can claim they could not have leaked because they did not actually get the grant. The grant transmission failed. So the admit removes that excuse. The incentive for the admit would come from mutual commitments on the part of the grantor. The grantor gets an escape clause unless the admitter verifiably admits. So when and if enforcement happens, the grantor can't back out if the admitter produces proof of admission, which also means the admitter has no plausible deniability. It balances the enforceability by both parties.

I see this kind of language in legal contracts and NDAs where the lawyers attempt to close off arguable defenses, not changing the terms but finding ways to make the contract more easily enforceable.

This is why its related to the byzantine generals problem, if I send an order to attack I need an acknowledgment the other party got the order or I don't attack, but then the other party needs to know I got their acknowledgment or they don't attack which means I need to know they got my acknowledgement of their acknowledgment. It never resolves unless there is a ratchet mechanism and that looks like a 3 of 4 threshold.

In legal contracts the problem is not so dire as the byantine generals problem. The problem is enforceability and eliminating easy defences to enforcement like I did not get the grant so even though I agreed to treat the data as confidential, I never got it so you can't punish me. The admit removes the easy out. But to incentivize the admit the grantor needs an escape clause. Neither becomes important unless and until there is an enforcement action and basically they nullify each other. Its a clever legal mechanism for confidentiality provisions.

there are other legal mechanisms like a blanket clause that says no matter how you get this information you treat it as confidential going forward, but how can I enforce if they don't know what the information is in the first place. Usually this means I have to hand deliver the information so I can simultaneously prove they got it as they get it.
When this happens remotely over the internet then that provability goes away.

So when we are explicitly not disclosing until the terms are agreed to, we can't use these blanket clauses over the internet. The plausible deniability that the grant was never recieved offers and easy out. The admit message defeats that easy out. The challenge is then incentivizes the admit. Even without an admit, we can enforce the agreed-upon terms, but now we have a burden of proof problem. We have to show evidence that the receiver did indeed get the information in some way.

This is why the duty of data loyalty is so useful. Because with the regulation of the duty of data loyalty, it doesn't matter how the receiver got the information, if they use information about us, no matter the source, in a self-dealing (not in our best interests) way, we have recourse.

So CLC and DDL are mutually reinforcing. Once data processors understand that they have unlimited liability with a duty of data loyalty, then they will earnestly want a regulatory safe harbor, and in order to get that regulatory safe harbor, we can make it so that they have to provide the admission. So now we have an incentive that forces the receiver to remove their own plausible deniability. It really ties the bow on CLC and the duty of data loyalty in my opinion.


I think I misunderstood your question. I assumed you meant should a failed transaction that starts over be one or two transaction but you were asking if a single linear transaction should be one or two where the boundary is where the anchors occur.

For linear transactions I see no reason to split. What we are anchoring is the tail of the transaction not the head. Because the events in an exhange transaction are hash chained together, anchoring the tail also anchors the whole transation including the head.

Multiple anchors are not meant to anchor multiple transactions but as continencies for the transaction to proceed.
In a high-stakes transaction, the Discloser wants to see a perpetually verifiable commitment to the terms via the Agreement of the Disclosee before issuing a Grant to the agreeor. The Disclosee (agreeor, admitor) then wants to see that the Discloser (grantor) anchors the grant before producing an admit. Often there is mutual accountability in terms and conditions especially in high stakes transactions and so the Disclosee (recipient) wants to have perpetual verifiability of the Discloser's (grantor's) mutual committment before issuing its admit.

Why the admit, because in contract law, a potential point of legal enforcement friction is that the Disclosee claims they never received the grant. The admit message removes any plausible deniability by the Disclosee. The anchor of the Grant message is incentivized by the following admit. The grantor wants the admit but cant get it until they anchor their grant. The terms of the contract could include an escape clause for the grantor if there is no admit message which incentivizes the admit. And since it must be anchored, the admitor has a way to prove it later, to nullify the escape clause.

Typically, high-stakes transactions happen under the auspices of a trusted third party who ensures the transaction completes (settlement). But we want to have true peer-to-peer, legally enforceable transactions without trusted third parties, so we need a sequence of commitments that are are successively incentivized without a trusted third party to ensure the completion of the transaction (settlement).

So the Offer/Agree/Grant/Admit removes plausible deniability which is the bane of peer-to-peer interactions. This is related to the byzantine generals problem which requires a multi-phase commit to solve.

Both can work. We have to decide if a spurn ends the transaction and one starts over with a new xip for the apply. I think this is cleaner. But technically one could include the spurn in the transaction and chain a new apply there and just keep going. I can think of use cases where the actualy negotional back and forth wants to be memorialized not the linear final result. But for most use cases the final linear result is best.

## New Unary Edge operators

### Unary Delegative Operators

With the exception of the NOT operator, the current set of Unary operators are delegative. They modulate delegation relations between the near and far ACDCs one an edge.

| Unary Operator | Description | Default|
|:-:|:--|:--|
|`I2I`| Issuer-To-Issuee, The Issuer AID of this ACDC MUST be the Issuee AID of the node this Edge points to.  | Yes |
|`NI2I`| Not-Issuer-To-Issuee, The Issuer AID of this ACDC MAY or MAY not be the Issuee AID of the node that this Edge points to. |  No |
|`DI2I`| Delegated-Issuer-To-Issuee, The Issuer AID of this ACDC MUST be either the Issuee AID or a delegated AID of the Issuee AID of the node this Edge points to. | No |
|`NOT`| Logical NOT. The validity of the node this Edge points to is inverted. If valid, then not valid. If invalid, then valid.  | No |

When the Operator, `o`, field is missing or empty or is present but does not include any of the `I2I`, `NI2I` or `DI2I` Operators then:

- If the node pointed to by the Edge is a targeted ACDC, i.e., has an Issuee, then the `I2I` Operator MUST be appended to the Operator, `o`, field's effective list value.

- If the node pointed to by the Edge block is an Untargeted ACDC i.e., does not have an Issuee, then the `NI2I` Operator MUST be appended to the Operator, `o`, field's effective list value.

Many ACDC chains use Targeted ACDCs (i.e., have Issuees). A chain of Issuer-To-Issuee-To-Issuer Targeted ACDCs in which each Issuee becomes the Issuer of the next ACDC in the chain can be used to provide a chain-of-authority. A common use case of a chain-of-authority is a delegation chain for authorization.

The `I2I` unary operator, when present, means that the Issuer AID of the current ACDC in which the Edge resides MUST be the Issuee AID of the node to which the Edge points. Therefore, to be valid, the ACDC node pointed to by this Edge MUST be a Targeted ACDC. This is the default value when the Operator 'o' field value is missing or empty.

The `NI2I` unary Operator, when present, removes or nullifies any requirement expressed by the `I2I` Operator described above. In other words, any REQUIREMENT that the Issuer AID of the current ACDC in which the Edge resides MUST be the Issuee AID, if any, of the node the Edge points to is relaxed (not applicable). To clarify, when operative (present), the `NI2I` Operator means that both an Untargeted ACDC or Targeted ACDC, as the node pointed to by the Edge, MAY be valid even when Untargeted or if Targeted even when the Issuer of the ACDC in which the Edge appears is not the Issuee AID, of that node to which the Edge points.

The `DI2I` unary Operator, when present, expands the class of allowed Issuer AIDs of the node the Edge resides in to include not only the Issuee AID but also any delegated AIDs of the Issuee of the node to which the Edge points.  Therefore, to be valid, the ACDC node pointed to by this Edge MUST be a Targeted ACDC.

The `NOT` unary Operator, when present, inverts the validation truthiness of the node pointed to by this Edge. If this Edge's far node ACDC is invalid, then the presence of the `NOT` operator makes this Edge valid. Conversely, if this Edge's far node ACDC is valid, then the presence of the `NOT` Operator makes this Edge invalid.

### Unary Identity and Equivalence operators

The unary operators needed by the vLEI ecosystem were delegative, i.e. the associated edge provides a delegation relationship. The SEDI ecosystem, however, benefits from additional classes of unary operators, (the worked example is such a case). In SEDI, a common case is that an edge provides an identity relationship or an equivalence relationship.  

The syntax convention for edge operators in general is to use Base64 characters for the operator symbols to ensure the CESR serialization of the operator is compact. So far only uppercase and number characters have been used. This follows a convention in automated reasoning systems to make the operator symbols stand out from their operand symbols which are lowercase.

An edge points backward from the near side ACDC holding to the edge to the far side acdc whose SAID is the node `n` field in the edge block.  

In the case of `I2I` the number `2` is used to symbolize delegation that flows down from far side  to (`2`) near side on an edge that points from near side to (`2`) far side. The delegation authority flows in the opposite direction of the edge. To clarify, the edge points from near `2` far  and delegation authority points from far `2` near where far is the delegator and near is the delegatee.

To clarify, `I2I`  means the Issuer AID of the near side is the Issuee AID of the far side. This means the far side ACDC Issuer delegated issuance authority to its Issuee AID, which AID is equal to the near side Issuer AID. To elaborate, in order for the edge to be valid, the near side Issuer AID MUST equal the far side's Issuee AID.
In the shorthand expression Issuer-to-Issuee, the "to" is pointing along the edge direction not the delegation authority direction.  

This convention can include a modifier on the identifier relationship defined by the edge. 

For example: `DI2I` means the near side Issuer AID MUST either be the far side Issuee AID or be a delegated AID of the far side Issuee AID.  So from a delegation authority point-of-view the "D" modifies who the delegatee can be. From an edge point of view it modifies what the near side Issuer AID can be for the edge to be valid, and that modification is that the near side Issuer AID can be equal to the far side Issuee AID or a delegated AID of the far side Issuee AID.

For SEDI, in addition to delegation relationships, we need identity and equivalence relationships. Specifically relationships between the near side issuee AID and the far side Issuee AID and also the near side Issuer AID and the far side Issuer AID. To clarify, these are not delegation relationships but either an identity or equivalence relationship. 

For example, one identity relationship (reflexive) is that the Issuee AID on the near side must equal the Issuee AID on the far side of the edge. Another would be that the Issuer AID on the near side must equal the Issuer AID on the far side of the edge. This poses a problem for the syntax because Issuer and Issuee both start with the letter "I". This is not a problem for a delegation relationship; there is only one pairing between Issuer and Issuee that is meaningful, so I2I is not ambiguous. For identity and equivalence relationships, however, we need to disambiguate Issuer from Issuee. 

Let's use E to represent IssueE, and continue to use I to represent Issuer. 
We could then alias the existing `I2I` operator to be `I2E` for Issuer to Issuee.  In hindsight using `I2I` was nearsighted. But we can keep it and add the alias `I2E` or replace it with `I2E` or leave as is. Likewise for the pair `DI2I` and `DI2E`, and the pair `NI2I` and `NI2E`.

Let's also use the number `1` to represent an identity relationship, which is evocative of the number `1` being the multiplicative identity. This is not the same as a delegation relationship. Delegation implies some form of transitivity.

So we have the new proposed identity relationship operators

`I1I` which means near side Issuer AID must equal far side Issuer AID.
`E1E` which means near side IssueE AID must equal far side IssueE AID.

And we could then have in addition.
`I1E` which means near side Issuer AID must equal far side IssueE AID.  This would not imply a delegation relationship, just an equality. So its not the same semantics as `I2E` or
`E1I` which means near side IssueE AID must equal far side Issuer AID.  

In mathematics, an equivalence relationship adds symmetry and transitivity to the reflexivity of an identity relationship. While not the same mathematically, in ACDC land an analogous type of equivalence relationship is delegation. And there are two senses of the use of the term delegation in KERI/ACDC. We have ACDC edges that delegate and we have delegated identifiers this are not mutually exclusive so we can have delegated edges between delegated identifiers as is the case for the `DI2I` operator.

So we can apply the same modifier to our identity relationships.
`DI1I` would mean that the near side Issuer MUST must be either the far side Issuer AID or a delegated AID of the far side Issuer AID. This would allow an agency relationship where say a driver's license was issued by a different Issuer AID from the core SEDI credential but must be a delegated AID. Or we don't have an identity operator for the Issuer on the edge but allow the EGF to specify the relationship.

So to describe more complex equivalent operators where what is meant by "equivalence" may be ecosystems specific. If we try to use unique operators for each ecosystem specific definition of equivalence we create for us a difficult configuration management task of tracking what might be an unlimited number of unary operators.  If instead we default to saying that "equivalence" is defined by the EGF for a given set of ACDC types then we can define a smaller set of Unary operators whose operation is assigned on an ecosystem (EGF) basis.

The tradeoff is that we want to automate the process of validating or invalidating edges, so the more precisely we can define operators the more we can performance optimize tooling that is less prone to implementation errors.
So its a tradespace judgement call.

Lets use the number `3` to represent generic equivalence relationships where the actual meaning of equivalence is EGF defined. The `3` is evocative of the mathematical equivalence relation that has 3 properties, (reflexivity, symmetry, and transitivity). This provides a placeholder for explicitly requiring such a relationship on an edge, rather than merely implying it. But the relationship is looked up or relative to the EGF.

This gives us:
`I3I` and `E3E` for EGF defined edge equivalence relations between near side and far side Issuers and near side and far side IssueEs.


### Presentation Registries
However, there is some discussion about how to manage presentation registries. Should they be optional on a citizen by citizen basis. Or even a bulk issuance set member basis.  A presentation registry enables the Issuee to detect impersonation fraud of a compromise of its signing keys, so high stakes presentations want to to use a presentation registry, which requires that the "rd" field be populated at the time of issuance to force the verifier to verify the presentation against the presentation registry controlled by the issuee.    

This leads me to believe that we need to find a high-stakes use case where the Issuee would want to enforce a presentation registry. Maybe a drug prescription presentation where the prescription is either an ACDC or the cargo of a bespoke ACDC.


## Diagrams
### Blinded State Registry
![Blinded State Registry](media/17841393245934.jpg)

### Anchored IPEX Exchange Transaction Simple
![Anchored Exchange Transaction](media/17841393711052.jpg)

### Detailed Anchored IPEX Exchange Transaction
![Detailed Anchored Exchange Transaction](media/17841394443807.jpg)

### Detailed Anchored IPEX Exchange Transaction with Presentation Registry
![Presentation Registry Anchored Exchange Transaction](media/17841394794090.jpg)


### Example SEDI

So a little asynchrony here in thought. I have not digested your latest set of lengthy commentary. But I wanted to capture my second thoughts on this question first.

After more thought, I think my answer to the SEDI iidentity credential as an aggregate credential (array of attributes instead of nested field map). I realized I was falling into the trap of letting prior usage dictate instead of first principles.

So I now think that the SEDI core identity core identity credential should be an attribute credentials. Let me explain why.

From a data correlation perspective (the first principle). The one feature that an aggrative credential provides over a attributative credential is that the attribute blocks at the top level are not labeled with meaningful labels. So the position of a block within the array can be managed so as not to leak information where such information could be leaked via relative position. In the case of the core SEDI identity credential, there are no fields that benefit from such arrangment. The field labels will be well known and all the fields are a defined set so there is no “advantage” to using the aggrative form and there is a definite disadvantage to using the aggrative form. The disadvantage is that reasoning about the fields to be disclosed now does not use fully labeled paths but either must a derived path. If one says that the array index is in the path, then one defeats the one property of the array that one wanted to preserve. That means pathing for apply messages means that under the hood one has to convert an abstraction to a path without being given the index. One has to create a reverse lookup table that maps the unique label embedded in each attribute block into an index so that a path can look up the array index.

While this is no terribly complex, since there is no advantage to using an array for this credential, more complexity for the most used credential, the core SEDI identity credential feels like a poor decision.

So, given you have already done an example with an aggregation. I think it’s useful example still. But it would be great to do another example one where the core SEDI credential was instead an attributive one, each attribute besides the issuer aid, gets a nested labeled partially disclosable block with a said and uuid in its block.

This brings up the question of what is a valid use case for the aggrative credential that benefits from not labeling the attribute blocks where relative position in the array could leak information. The use case that I can think of is an array of Boolean flags for age thresholds. Each age threshold has an element so that the number of elements does not leak anything about which age thresholds are present. All are present but each is true or false.

And to go further, one could add empty elements in the array and randomize the relative positions in the array of the age thresholds so that a disclosure of given array of age threshold does not leak information about which elements in the array correspond to which ages. But the later one is probably overkill.

An aggrative array in this case is a poor mans sparse merkle tree.

So in hindsight I got the answer backwards. A better example would be to use an attributive ACDC for the core SEDI identity credential and use an aggrative credential for the proof of age credential only includes several Boolean flags for a set of ages like 13, 16, 18, 21. For the most important minimum legal ages

The plan is to eventually use a sparse merkle tree for this but using the Boolean array replicates how mdoc does so its at least as good as mdoc

Anyway appreciate your feedback thoughts as too the inversion of the example.

# Tension between linkabiilty, correlatability, decorrelation, and exploitation

@dhh1128 

Your question about standing edges brings up the broader issue of what is the set of protection measures for privacy. Its easy to only think along one axis of privacy protection mechanisms.  My view is that privacy protection by merely technical means is an exercise in diminishing returns. Only a comprehensive approach that combines technical, legal, and regulatory mechanisms will result in meaningful reductions in data-driven exploitation.

So any analysis is incomplete if it does not also include a more comprehensive take on  the different mechanisms.

So I go out of my way to distinguish between (un)linkability (correlation of shared identifiers) and (un)correlatability to a natural person i.e. reidentifiability.
As you well know and as commonly used in the decentralized identity community, (un)linkability has two types: verifier-to-verifier (vendor-to-vendor) and verifier-to-issuer (vendor-to-issuer).  

Vendor-to-vendor linkability means that when a given presenter presents a given credential to two different verifiers in two different presentations, there are cryptographically verifiable commitments to identifiers that are shared between those two presentations.  In other words, the intersection of the two sets of identifiers used in the two presentations is not empty (the identifiers cannot be partitioned between the two presentation contexts. (In the rest of this discussion, i will focus on vendor-to-vendor (un) linkability as it exhibits differently and has different tradeoffs in its mitigation than vendor-to-issuer(un) linkability

Uninkability does not mean uncorrelatability of the presenter as a natural person between presentation contexts. Mere unlinkability by itself does not guarantee that the two verifiers cannot figure out that the same presenter (as a natural person) was the presenter in both presentations.  This is because all real-world presentation contexts are leaky, and they can leak enough information to identify the presenter as a natural person across presentation contexts, regardless of whether any identifiers were used in the presentation, much less whether any explicitly used identifiers were shared between presentations. A set of vacuous presentations across different contexts can be correlated to the same natural person via contextual leakage. So any analysis that claims "privacy" without considering contextual leakage is potentially misleading and may be irresponsible.

A stronger goal that can result in real functional privacy is that vendors do not exploit correlatable information between presentations. This is much stronger because it is not the fact of linkage or even the fact of correlation that matters to the presenter but the fact of exploitation. 

Were they indeed exploited as a result of correlation?  If two presentations are surveilled and correlated, and this correlation includes explicitly presented information as well as contextually leaked information, but the correlator does not exploit that information against the best interests of the presenter, then the information is functionally decorrelated. It is correlatable, but it is not exploited in correlated form, so it is decorrelated.  

The purpose of contractually protected disclosure is to decorrelate correlatable information.  The anti-assimilation clause in the example is meant to retroactively remove information the vendor has already collected through contextual leakage and decorrelate it so it is not used to exploit the presenter. CLC acts to continually decorrelate the information as it flows downstream to other consumers of the data.  That does not mean it is not a good idea to use the principle of least disclosure in the process of graduated disclosure. This is because one type of attack is when a malicious party impersonates a legitimate actor in order to harvest correlatable information before reaching a point of verifiability of the potential disclosee by the discloser.    The sooner one cuts off the interaction, the more expensive it is for the bad actor to harvest. So it adds an ephemeral time value to information protection.  It doesn't prevent correlatability but it fosters decorrelation by shifting the economics.

Now, regulatory and legal means do not prevent illegitimate actors from acting illegitimately. But for many use cases, often the most important ones,  the presenter's best defense against illegitimate exploitation of their data is to not do business with illegitimate actors, not to falsely assume that they are protected by mere unlinkability.

This becomes more useful when the regulatory regimes remove easy outs for illegitimate actors.  

This is why Utah States recently passed SEDI law  is so profoundly important. The law mandates a duty of data loyalty that makes it illegal to exploit data collected about a user against the users best interests. It is comprehensive in scope.  

The current bill does not define the precise enforcement mechanisms. These enforcement mechanisms are meant to be codified in the 2027 legislative session.   Where this impacts the KF SEDI work is that the contractually protected disclsoure can be used to remove loop holes and remove friction from enforcement.

For the first time, comprehensive functional privacy becomes possible where data is functionally decorrelated, i.e., not used to exploit regardless of its linkability or degree of correlation.

Given the inherent correlatability of information, even when using mechanisms like bulk issuance to eliminate linkability and make correlation more difficult, one could ask why even bother with bulk issuance. One cogent reason to use bulk issuance despite its eventual defeat by a super correlator (an AI) is that partitioning identifier sets makes enforcement easier. If one limits an identifier set to a given vendor context where the vendor makes non-repudiable agreements vis-a-vis that identifier set, then any appearance of that identifier set downstream of that vendor outside the bounds of its permitted usage becomes prima facie evidence of non-compliance with data loyalty. Moreover, having concrete identifier sets enables strict safe harbors and protects vendors from the potentially unlimited liability that the duty of data loyalty imposes. An audit is exculpatory under the safe harbor only if all data items used by the vendor are provenanced via verifiable identifier sets. So granular partitioning of identifier sets across vendors makes provenance granular and enforcement precise. Precise enforcement is more effective as it makes enforcement more cost-effective and therefore more broadly employed, which enhances deterrence. Its creates a virtuous feedback loop which changes the economics of correlation and exploitation.


It is easy to show that all information intentionally disclosed or inadvertently leaked by a natural person, either in person or over the public internet, is eventually correlatable, given enough coverage by correlators. Empirically, unlinkability is easily defeated given the current coverage of AIs. This is because a natural person, as a bag of atoms, is not reproducible (physically clonable) and therefore their actual expression on the internet and in person is limited in extent, they cannot be everywhere at once, so by a process of elimination (type of correlation) using interaction graphs they can be reidentified with their online behavior. An interaction graph maps all online interactions between the potential set of participants which can be as large as everyone in the world for an AI. Given this degree of coverage, one  can eventually with high confidence reidentify every natural person via their online behavior even when using anonymization. This is because the public internet is routable and routing employs a static, identifiable structure, so this structure seeds the expansion algorithm that eventually correlates participants via their interactions.  (The technical class of algorithms often uses the term "seed set expansion algorithm" where a seed of identified interactions is used with an interaction graph that includes unknown participants that over time as more interactions are collected will eventually identify all participants. And once one has a corpus of interactions with a high percentage identified, the ability to maintain that graph gets easier and cheaper.

The current set of AIs individually already has this degree of coverage. The rise of deep fakes only makes this correlatiblity a sure thing, because the AI countermeasure to a deep-faked online  persona is to create an even more high-fidelity real persona, which requires more correlation. To clarify, the industry-adopted counter-intelligence protections from deep-fakes digital personas are real digital personas that rely on even better correlation of real behavior. Thus, the identity security world is moving to security that requires correlating and tracking the real person's online behavior at increasingly invasive levels so as to build a higher fidelity persona than the deep fake persona. We have an arms race with billions being expended to produce higher and higher fidelity correlated real personas in order to distinguish from increasingly higher fidelity deep faked personas.

The only way to be protected from such correlation is to truly and profoundly go off grid.  This is highly impractical for the vast majority. So its not a viable goal.  So the viable goal can not be uncorrelatability it must be decorrelation.
