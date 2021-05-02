---
tags: ACDC, KERI, JSON-LD, RDF
email: sam@samuelsmith.org
version: 1.00
---

# VC Spec Enhancement Proposal

[![hackmd-github-sync-badge](https://hackmd.io/rnY5xiXESWqWBlyDPHDorA/badge)](https://hackmd.io/rnY5xiXESWqWBlyDPHDorA)


Strategic Technology Choices vis-a-vis the JSON-LD/RDF End State.


## Adoption Strategy

The purpose of this write up is to better memorialize my increasingly worrisome concerns about the adoption path for Verifiable Credentials. My concerns started with security of VCs, as a member of a more general class of data that may be described to as securely attributed or provenanced data. Other terms include authentic data and more specifically provenanced data or authentic chained data containers. My interest to design the tech and follow an adoption strategy with end state being universal adoption of securely attributed data. This fixes the broken internet. It may never reach that
end state but I hope not because of any technical reason nor because of a bad adoption strategy.

In my opinion the primary role of an identifier system to solve the secure attribution problem. The core of such a solution is to establish control authority over an identifier where such control authority consists of a set of asymmetric (public, privae) key pairs for non-repudiable digital signature(s) (PKI). Other cryptographic operations may serve a similar role but the simplest most universal method of establishing provable control authority is an asymmetric key pair based digital signature on some statement. Only the controller of the private keys may make a verifiable non-repudiable commitment to some statement via such a signature. Any second party may verify that signature given the public key. The role of any identifier system is to securely map identifiers to the authoritative public keys in order to cryptographically verify the signatures and thereby make sucure attribution. Secure attribution via digital signatures is a cryptographic way of establishing the authenticity of any signed statement.

KERI is one such identifier system. DID indentifiers belonging to the named space of some DID method are another such system. 

Once one can make secure attribution then one can built secure transactional system on top of that secure attribution system. One such transactional system is a Verifiable Credential (VC) but that is only one. The end state for decentralized identifier systems is to enable all transactional systems to be securely attributed using that decentralize identifier system.

This is a very bold and ambitious goal that requires a realistic and sane approach to adoption. Any adoption barrier that reasonably would significantly prevent reaching that end goal must be carefully considered. Any technology related adoption barrier that would prevent such broad adoption should be  discarded unless there is no viable alternative.

### Adoption Strategies w.r.t. Adoption State

Often a new technology may choose as its adoption strategy to leverage some other pre-existing technology, especially pre-existing open standard technology. The reputation of the pre-existing standard may reduce the perceived risk of the new technology enough to gain more rapid adoption of the new tech. Often the new tech, by virtue of its new features is not an exact fit for the pre-existing standard. However the much larger adoption status of the pre-existing standard means that adapting it or modifying it even if not the most elegant approach pays off with accelerated adoption of the new tech. Over time the new tech may grow big enough to incentivize the pre-existing standard to change to better accommodate the new tech.

However, on the other hand if the pre-existing tech is at best a limited adoption standard that fits a narrow niche where its technology is either too special case or is not scalable enough that it will likely never be widely adopted or its market penetration has already peaked, then it may be a poor choice for the new tech strategically. Moreover if the old standard and the new tech communities are not fundamentally aligned in terms of purpose and approach then there may be too much friction from that misalignment to ever attain first class status and receive first class support for the new tech from the pre-existing community. This misalignment friction would make the adoption journey net counter productive for the new tech.

How this relates to RDF and JSON-LD.  RDF is old tech that long ago peaked in its adoption curve. JSON-LD is an attempt to refresh that old tech and bring new life to its adoptability. Based on my most recent preliminary investigation into the end state for JSON-LD and where the JSON-LD community intends to go, I believe that its adoptability will never scale to level needed nor be net advantageous to the goal of supporting the majority use cases for provenanced data, where VCs are a seminal example. The purpose of this write up is to better understand and confirm the validity of that preliminary investigation. 

In short the problem is that JSON-LD/RDF emphasizes interoperable extensible semantics at the cost of interoperable security and trust. It also uses the weakest form for semantic inference that has long been relegated to legacy status, i.e. it does not represent the state of the art for semantic inference. These two fundamental technical distinctions are why I believe JSON-LD/RDF has a hard adoption ceiling that is much too low to meet the goals of VC adoption. I also see strong evidence that this technical distinction will becomer wider and harder to bridge going forward because the growth is happening outside of JSON-LD/RDF at such a rapid pace that there is no chance for JSON-LD/RDf to ever catch up. Its already lost the adoption battle.

The best case that we could hope for given the current VC spec and the bias of the JSON-LD community to to change the spec to support non schema.org non RDF schema. This would be at best an uneasy compromise equivalent to the shotgun wedding compromise reached for the DID spec where JSON-LD is not required. A second best case would be an exception where a special form of immutable schema (i.e. not schema.org) but that still used JSON-LD syntax. But this is antithetical to the core values of the JSON-LD community and would be a source of never ending friction. Which by itself poses a significant adoption barrier as discussion would tend to quickly degrade into unproductive disagreement and political posturing as it often does in the DID WG for the same reasons.

To better understand the core problem mentioned above, I first review the prioritization of system design principles associated with solving interoperability for both the secure attribution problem and the semantic normalization problem. i.e. interoperable security and interoperable semantics.


### System Design Principles

Interoperability layers:
- Security first always.  
- Semantic second always.

Interoperable Secure Attribution of data first; interoperable semantics of data second.  
  
If we want move data across trust domain boundaries then we need an interoperable protocol that enables the secure attribution (authenticity) of the data at each boundary.  In other words secure attribution must be first be established when data crosses a trust domain boundary. If we have a universal decentralized protocol for establishing secure attribution then those trust domain boundaries can belong to decentralized entities with no loss of security.

Best practices for security system design means we want to discard any data
that is not securely attributed as soon as possible in our processing stack. Pushing it further up
in the chain exponentially expands the attack surface for DDOS attacks and transaction malleability attacks. It also exponentially expands the vulnerability surface for correct secure code delivery and execution.
  
After we have proven secure attribution so we can move the data across the trust domain boundary securely, then and only then may we or should we address the secondary problem of moving the data semantics across the trust domain boundary. There is no point in moving the semantics across a trust
domain boundary for data that will be discarded because is is not securely attributed. Its putting the semantic cart in front of the security horse.

This puts interoperable security in the primary position and interoperable semantics in the secondary position in our system design.   

To reiterate this layering follows best principles for protocol design layering. Session and Presentation lauyers come before the Application Layer.
Payload semantic verification and integration happens at the Application layer not at the Session and Presentation layers where secure attribution happens. Any good IT protocol security person would be surprised at any suggestion that security should happen after data semantic integration into some backend database (aka an RDF graph).

Linked Data inverts this prioritization by putting security behind semantics (aka RDF expansion and normalization first, signature verification second). Given the number of proposed expanded RDF serialization signature standards, the JSON-LD community is blithely unashamed to propose such an inverted security layering. See section below on end state for JSON-LD RDF security.

Using hash links on the schema.org schema does not guarantee the proper form of immutability that ensures an issued VC is verifiable. It only enables detection of mutability after the fact. This means that some authority besides the issuer is enabled to effectively revoke any json-ld credential where the signatures are on the over the wire JSON-LD.  The VC is effectively revoked because it is not verifiable due to the mutation of the schema. The issuer must then revoke and reissue not for any valid reason of the issuer but merely due to the mutation of the schema which is not under the control of the issuer. This makes schema.org an attack vulnerability. It creates a treasure trove for attacks. Any attacker who hacks the schema.org registry could force all issued VCs everywhere to be unverifiable.

As a result we lose interoperable security in any reasonably scalable way which means we can't solve the trust spanning layer problem of universally secure attribution of data.  
  
Instead we have Balkanized our trust domains into at the very least RDF and non-RDF. And RDF will always be dwarfed by Non-RDF for many reasons not the least of which are inverted (bad) security design described above. 


## Survey of the Market for Semantic Interoperability

Web 3.0 10 years ago meant semantic web enabled = RDF = legacy tech
Web 3.0 now means crypto enabled = trust = security = new tech


Semantic interoperability levels:
- API level 
    - Consume data using OpenAPI JSON schema, custom schema  
    - Single backend database SQL, NoSQl, Graph
- Data Lake level  
    - - Consume JSON schema or custom schema. Map to data lake 
    - Multi backend databases, cross database schema enforcement
- Knowledge Graph level  
    - Depending on the graph database: consume JSON Schema, custom schema or schema.org schema
    - Graph databases (Property Graphs, Weighted Directed Edge Graphs, Is-a has-a triples)
        - Open property graphs (Neo4j, Arrango, various GraphQL)
        - Repurposed SQL or other NoSQL to act like property graphs
        - Proprietary property graphs
        - Machine Learning Oriented graphs (weighted directed edge property graphs)
        - RDF graphs (JSON-LD)
    - Knowledge graph schema integration  
        - indirect: consume JSON schematized documents and map to backend graph database format
        - direct: consume JSON-LD, Owl etc schema and expand to RDF graph database format
    - Knowledge graph inference  
        - weakest inference is RDF (only semantic inference compatible with URI links)
        - strongest inference is weighted directed edge property graphs 
        - mine legacy open RDF graphs and convert to directed edge property graphs to perform inference.

[Knowledge Graphs 2021 survey](https://arxiv.org/pdf/2003.02320.pdf)
[Knowledge Graph Wikipedia](https://en.wikipedia.org/wiki/Knowledge_graph)

### Market Segments for Semantic Interoperability

- API level: 90% + json schema or custom schema (relative to the other two)
- Data Lake level: ?? (relative to knowledge graphs)
- Knowledge Graph level ranking by market adoption: ?? (relative to data lakes)
    -  First proprietary knowledge graphs for social networks or page rank 
        - Consume JSON schema or custom schema serializations with one exception)
    -  Second open labeled property graphs (LPG) (GraphQL) for social networks
        - Consume JSON schema or custom schema serializations)
    -  Third repurpose SQL or NoSQl to act like labeled property graph
        - Consume JSON schema or custom schema serializations)
    -  Fourth machine learning custom or proprietary weighted edge knowledge graphs
        - Consume JSON schema or custom schema serializations or mine other knowledg graphs)
    -  Fifth (distant last) RDF JSON-LD
        -  Consume JSON-LD or other RDF serialization


### Semantic Inference and Automated Reasoning

Machine learning and automated reasoning are types of inference mechanisms that may be applied to knowledge graphs when those graphs support uncertainty measures. The primary mechanism for representing uncertainty in a knowledge gragh is to use weighted edges, aka,
weighted directed edge graphs. A generalization of a weighted directed edge graph is a property graph that supports as edge properties directed weighted edges. In such a generalized LPG, each edge type may have multiple properties and may have one or more weights to represent different forms of uncertainty. Similarity (ambiguity, distance), Probability (liklihood), Possiblity (precision, plausibility, etc). Mathematically its measure theory. Whereas RDF triples provide no comparable mechanism. Its just not reasonable. IMHO this is one of the main reasons the semantic web died. Because the main consumers of semantic web information, the data aggregators need a more powerful graph model that supports uncertainty measures in order to do machine learning type inference. In other words RDF is considered legacy technology largely because it does not support the most useful and powerful forms of inference. For example one problem in mapping knowledge is called polysemy.In the IT world mapping one corpus of knowledge to another encounters the problem of polysemy. One way to address this is to use similarity measures to measure the degree of polysemy.  [Termediator](https://dl.acm.org/doi/10.1145/2656434.2656443). The measures of polysemy are similarity measures (uncertainty). The process is called terminology mediation. I often find myself in discussions with RDF advocates where they assume that polysemy is not a thing and hence don't appreciate any attempt at terminology mediation =). They believe erroneously that a single ontology of terms with zero uncertainty is practically attainable in the real world for anything but extremely narrow isolated contexts. This ignores the inescapable uncertainty that polysemy measures on the set of all contexts.

The only application where RDF is well suited for inference is the semantic web which is constrained by
the semantic limitation of mapping to URI/IRIs. For every other application of semantic inference, labeled property graphs are more convenient and more powerful (social networks) and for machine learning weighted directed edge graphs (including labeled property graphs with weighted edges) are essential. Its much much harder to reason with RDF triples than pretty much any other type of semantic graph. Read any tutorial on modeling and inference that compares LPG to RDF and its much more intuitive to model and infer with LPG. Labeled property graphs have properties on both the nodes and the edges of the graph. This allows easy modeling of complex relationships. RDF allows no properties on either nodes or edges. One must jump through hoops to model complex relationships with many many more nodes and edges and do things like reification or other workarounds. Querying RDF becomes complicated very quickly. Absent the need to represent all edges and nodes as URLs I doubt that anyone would have though that using triples as a starting point for semantic inference was the best way to go. They would start with LPG instead. Recall that the original whitepaper [Identity System Essentials](https://github.com/SmithSamuelM/Papers/blob/master/whitepapers/Identity-System-Essentials.pdf) defines an identity as an identifier plus attributes. This exactly a node in an LPG. Whereas in RDF its many triples, one for each attribute. Also recall that this whitepaper describes an identity graph that maps all the relationships between identities. This is also exactly modeled with an LPG. The edges have properties to distinguish different types of relationships. But in RDF there are no properties on edges. Different types of relationships require cumbersome workarounds. RDF is plain cumbersome and ill suited to decentralized identity relationship modeling. Why use an old archaic tool built for a different purpose when there are many, more widely adopted, more widely supported, more performant, more suitable, open tools based on LPGs?

To elaborate, standardizing on RDF forces one to express semantics in triples which only supports the most cumbersome and least expressive of all the inference types. As a result it is also the least adoptable choice among all the types of semantic inference. The most adoptable would be labeled property graphs. I used to teach graduate classes in automated reasoning. A property we explore and apply to each technique is its ability to represent and reason about knowledge with uncertainty. Real world practical knowledge has uncertainty in it. The expressive and reasoning power (of which inference is a type of reasoning) of a given technique includes how well it handles uncertainty.  To illustrate consider this simple example. Lets start with an extremely simple triple:  
`Joe is Tall`.  (or more correctly as a triple but more awkwardly `Joe is a TallPerson`)
How do I represent Tall? The adjective tall is inherently uncertain. One way to represent this uncertainty is with a fuzzy set. For example I can get a reasonable fuzzy set to approximate `Tall` expressed of the range of heights by taking the histogram of heights of the general population, extracting the half of the histogram that is above the mean, normalizing it on a scale of zero to one and inverting it. 

How do I represent the inherent uncertainty in the concept `Tall` if all I have are Boolean truth valued nodes and edges in an RDF graph? Where each node and edge must be expressed as a URI?  I would have to do complex backflips at the very least. Now let’s add to our corpus on Joe by adding the triple. 
`Joe is a BasketballPlayer`.   
Now given this added information, does my original fuzzy set taken from the general population histogram of heights still accurately convey the uncertainty of the tallness of Joe?  Not so much because basketball players on average are much taller than the general population.  So if a have a frozen definition of `Tall` then I wouldn’t have said `Joe is Tall` in the first place I would have said `Joe is Very Tall`.  So how do I model `Very Tall` with a triple? Do I create another node that is the `VeryTallPerson` node. I would rather have an edge that is the modifier `Very`. The latter means that I can use `Very` for any number of objects that are uncertain because `Very` may be an operation that shifts my fuzzy set along its domain.   But I can’t have edges that are operations when I am limited to the is-a has-a triples of RDF. So I am stuck with doing backflips again to represent a very simple idea, `Joe is Very Tall`. 
Now lets add some more information with the triples:
`Eva is Tall`. 
`Eva is Female`.  
Is the `Tall` for Eva that same as Joe (who I have assumed is Male). Well the histogram for heights is different for females and males. Uncertainty is a permanent inherent property of knowledge and is-a has-a triples have no way of representing it. Now lets add to this the triple:
`Bob is not Tall`. In RDF there is no negation operation on an edge. One can’t make statements like this without doing backflips. I could say instead, `Bob is Short`. But `Short` is not the same as `Not Tall`.  So I could create a new node to represent `NotTall`,  or I could just have an edge with an operator that is the negation operation, `Not` (if only my graph language allowed it).

As this simple example illustrates, for any practical knowledge representation with uncertainty, bare triples are woefully inadequate. In my classes we would spend a couple of days using triples as a basic introduction to knowledge representation and then move one to more powerful representations. There are so many richer more expressive more powerful ways to represent knowledge and reason with it than is-a has-a triples (of which RDF is one representation) especially for any practical real world knowledge imbued with uncertainty. Given that understanding it is hard for me to appreciate why anyone would select RDF instead. Legacy support? Impractical ideology? Uninformed? Misinformed? Free Government Money?


### RDF Impracticality Presaged Its Own Demise

Frankly it was naive to propose that the semantic web was practical with only RDF triples as the knowledge representation format (The Semantic Web)[https://web.archive.org/web/20171010210556/https://pdfs.semanticscholar.org/566c/1c6bd366b4c9e07fc37eb372771690d5ba31.pdf]. In 1999 it was already well known in the automated reasoning community that practical inference was problematic with triples. Anyone who had a solid foundation in the field of automated reasoning at that time would have preferred something better that included at least some way to handle uncertainty. Oddly enough LPG can be implemented with URIs if one uses the query string to provide the edge properties and also uses JSON at the resource end point to provide the node properties. But who I am to suggest such a thing =).

This does not include all the myriad other types of semantic inference that are not based on knowledge graphs at all. Moreover all these other types of semantic inference would only consume schema at the API level not any higher level. Examples of semantic inference techniques that are not graph based include deep learning, case-based reasoning, fuzzy logic, Bayesian belief networks, non-graph rule base systems, etc. By limiting ourselves to the edge case that is JSON-LD/RDF we impose extreme friction on everyone else.

But this is just not my opinion. I have read several articles from those in the RDF world that also conclude that RDF is a dead end technology. Here is an article from a world class expert in data management who fielded some renowned applications at scale (which is important) using RDF technology who now has abandoned RDF not only because of its practical limitations (both semantic and performance) but because there are better solutions such as property graphs.  [Why-I-dont-use-semantic-web-technologies-anymore-even-if-they-still-influence-me](https://www.lespetitescases.net/why-I-dont-use-semantic-web-technologies-anymore-even-if-they-still-influence-me)

I find the lessons learned telling. I paraphrase: The ideal of RDF is belied by its practical limitations. The main limitation being that RDF is too hard to use. If no one will use it then it is practically useless. If one really truly needs a knowledge graph then use a labeled property graph which has broad industry adoption.

Quoting: “So currently, Semantic Web technologies are only used to retrieve and process the data from Wikidata so as to enrich our reference vocabularies for people and places. If the need arises and / or if there is a political intent, we can still consider exposing all or part of our data as Linked Data and through a sparql endpoint, but otherwise, we will prefer dumps in simple formats, to ensure greater data reuse. Thus, while being impregnated with all the reflection and the contributions resulting from the Semantic Web technologies, our project does not use them any more.” 

My survey of other attempts to implement RDF at scale end this same way. Great ideal but fails in practical use. In short RDF is a dead end. Its a siren song that ends up drowning its adherents. Time to move on. Great ideal that never realized its imagined potential. More importantly there are now better technologies to use instead.  Given this strong conclusion I am surprised that anyone besides overly idealogical RDF diehards or the uninformed or misinformed would select RDF for any new standards. I always operated on the assumption that as long as I was not forced to use RDF, I didn’t really need to care if someone else preferred it. But that assumption has been proven wrong. It seems that there is an unreasonable bias that threatens to force me to use RDF for VCs and that would be an adoption dead end and a security disaster. 


### One narrow use case for JSON-LD: Google SEO

According to the JSON-LD site, 10 million sites use JSON-LD. However in my survey of the uses, I could find  one and only one narrow use case, that is, to provide JSON-LD/schema.org encoded content for Google SEO of their site. Not for any semantic inference on the sites themselves. Most if not all of those sites also use either JSON Schema or custom schema for every other schematic purpose on their site (not schema.org). And most if not all do not use an RDF graph database backend for any purpose. Many either use a different graph database backend or no graph database backend at all.
In stark contrast to schema.org, JSON Schema is part of the OpenAPI spec (Swagger). JSON Schema is already the defacto web standard for interoperable schema. JSON-LD/schema.org will never ever displace it. JSON Schema has already won that battle.

This is comparable to the XML vs JSON battle. in almost every technical way except simplicity and compactness, XML is better than JSON as a serialization format. But JSON is good enough for the 99% of use cases so simplicity became the deciding feature.  JSON won the battle. 
Likewise json-ld/RDF/schema.org is superior in many ways to JSON Schema but JSON Schema is already good enough for the 99% use case which it already serves and is much simpler. JSON schema has already won that battle. The decentralized identity community just hasn't realized it yet.

The proper use of RDF and JSON-LD is the semantic web. The only significant consumer of semantic web data is Google for their proprietary Google Knowledge Database (which is not RDF under the hood). One of its APIs allows it to consume RDF data (but not the only API so its optional). 

In general, any knowledge graph database may be dummed down into an RDF representation in order to fit the extreme constraints of the semantic web. But that dumming down removes much of the expressive power of the the not dummed down graph. JSON-LD/RF is relatively poorly suited for any other purpose but the semantic web. As it now stands almost nobody is actually using the open semantic web in any meaningful way.  Most if not all of the other open standard graph database formats are more powerful, more performant, and more popular than RDF. They all enable semantic interoperability by consuming JSON schema and more importantly provide more powerful, more convenient, and more performant semantic inference than RDF. 

Nonetheless, the ultimate goal is not semantic interoperabiity nor is it security interoperability. Both of these are just means to another end.  That end is automated reasoning (of which semantic inference is one type of automated reasoning). We want securely attributed data (VCs) so we can do automated reasoning  (automate workflows) on data that crosses trust domain boundaries. The value capture is in automating the workflows. Which starts with verifying the authenticity of the data but does not end there.

### Selection Bias

Based on my survey the decentralized identity community is suffering from a severe case of selection bias induced by the grossly disproportionate participation of members of the JSON-LD/RDF community.

This bias has been exacerbated by the infusion of money from a single DHS program manger. Government money when ill placed distorts the market and when that market is already small, a little government money creates a large distortion. Many in the JSON-LD camp in the decentralized identity community are only there because they are funded by this one DHS program that has selected JSON-LD/RDF. They would not be there otherwise. Certainly not for any commercially driven market demand. They are leveraging the government money to bootstrap to commercial markets.

We are allowing ourselves to be stampeded by the JSON-LD community into an adoption box canyon because we have been blinded to other better options by the selection bias of the community.

### Designing Semantic Interoperability with the Ultimate End in Mind

As mentioned above semantic interoperabilty is just a means to an end. The more proximate end is automated reasoning. But automated reasoning is just a means to another end. The ultimate end in my opinion at least with respect to Verifiable Credentials may expressed succinctly as lowering trust transaction costs. This costs are lowered by automating the process of establishing and conveying sufficient trust to enable interactions (transactions). When one starts at the end and works backwards and also recognizing the inescapable pervasiveness of uncertainty (in all forms) and how to automated reasoning with that uncertainty then one realizes how limited the role of semantic interoperability becomes. Automated reasoning systems in complex uncertain environments (aka the real world) are layered and hierarchical and must isolate and properly manage domains of uncertainty not mash them together by stripping off that uncertainty. This allows partitioning to reduce the drag of inter-contextual uncertainty (ambiguity, polysemy). So one does not need to or would even want to attempt to have a flat sea of knowledge using one barren over simplified representation (like is-a has-a triples), but instead one wants to build a layered hierarchy of reasoning with knowledge.  With different reasoning methods and different knowledge representations that are best suited to the reasoning method not the other way around. This constrains and simplifies knowledge domains and contexts and makes them manageable. This is how the human brain works. It allows us to reason in an environment of extreme complexity and uncertainty. Automated reasoning systems design starts with end decision and works backwards to the source data. An automated reasoning system may have many layers such as Learning, Adaptive, Supervisory within each decision making process or construct. The data representation is driven by the decision making process not the other way around.

Lets take a simple example:  A given eco-system of transactions involves decisions that enable those transactions such as do I create for a purported person a bank account. If we oversimplify our knowledge representation to early in the process by removing uncertainty then we will build a fragile decision making process. One reason real world practical automated decision making systems (aka automated reasoning) are layered is because anti-fragility comes from supervisory, adaptive, and learning safety jackets.  

Returning to our example. Suppose one of the requirements for creating a bank account is proof of identity such as proof of the assertion that Joe is a US Citizen. Suppose that a passport may serve as proof of Joe's citizenship. However in an automated computer system there is no way to establish with complete certainty the Truth of the statement Joe is a US citizen. It may only ever be established as TRUE to some degree. For example suppose Joe presents his passport by sending a scan of it. An image processor scans the image of the passport and based on a set of features returns a likelihood measure that the passport is valid (its only statistically valid). The passport includes a photo. But maybe the passport was stolen or someone made a scanned copy. As a result we decide we can't rely on the passport alone. So we require another form of photo ID such as a drivers license. We have another process to compare photos and do a match. But you might say that this example is a motivating example for why we want a verifiable credential. We erroneously believe that A VC that states Joe is a US Citizen is now cryptographically verifying the truth of that statement. Is not. A VC does nothing to establish the veracity of the statements is makes. It merely establishes the authenticity. The veracity comes from the measurement and decision process used by that issuer. And real world processes are uncertain to some degree. Hence the veracity is uncertain. We must trust the issuer but the degree of uncertainty in either the original decision process or our trust in the issuer is not provided in the VC. So any subsequent decision process that uses that VC has lost the inherent uncertainty that qualifies the statement, Joe is a US Citizen and has replaced it with a reliance on trust in the reputation of the issuer of that statement. 

The idea of semantic interoperability is built on the poorly founded idea that one can combine statements whose veracity is purely established by reputation as if they are TRUE statements. Once one accepts this poorly founded idea, one is tempted to believe that given any statement so established as TRUE then all one needs for semantic interoperability is a universal schema and representation that allows one to match up as identical, True statements from different issuers and then combine them into derived True statements that are every bit as TRUE as the source statements. Anyone doing so has removed the uncertainty in the inference process at their peril and it presents a false sense of accomplishment. They have misled themselves into believing that if one only had a universal schema then one could reason semantically in an unbounded way about all such TRUE statements. But truthfully one's ability to infer is always limited by the uncertainty of the underlying decision processes. No statement is ever completely TRUE and when one stacks or chains a set of almost True statements in an inference, the TRUTH of that inference may decrease dramatically with the length of the chain, or the height of the stack. For example consider the case of probabilistic uncertainty. Suppose Joe's presented passport was validated by the VC issuer's image processing algorithm with probability 0.95.  And suppose that Joe's drivers license was validated by the image processing algorithm with probability 0.9 and that the process for matching photos was evaluated with a match probability of 0.8.  If we say Joe is a US citizen if Joe presents a valid passport and Joe presents a valid drivers license and the photos of the two match. This is a conjunction whose truth (probabalistic conjuction) is 0.95 * 0.9 * 0.8 = 0.684. Clearly much less TRUE. Now one may quibble and note that the combination of those three items is not a simple AND conjunction but must be normalized by the conditional probability of both a drivers license and a passport with the same name and matching photo (making the probability higher than 0.684). This is besides the point. Once we have removed the uncertainty in our model, whatever it may be, we have no way of knowing the actual TRUTH value of our chain of reasoning. So we would be foolish to attempt any long chains of inference on big flat knowledge graph. So we there is little benefit to a big flat knowledge graph.

To elaborate, when the issuer of the VC issues a credential that states Joe is a US citizen. It is not truly TRUE only more or less TRUE. But a consumer of that VC does not know that. And any downstream decision process may be made more fragile by that lack of knowledge. Recall, that any real world data has uncertainty to some degree and may have multiple forms of uncertainty. As a result, any inference chains on that real world data must be bounded in length because the inferred TRUTH decreases with the length of the inference chain. This means that any corpus of knowledge that excludes uncertainty in its representation and thereby allows inference chains of unbounded length is a completely unrealistic approach to modeling and reasoning about the real world. Its a theoretical fantasy and a practical nightmare. This aptly describes RDF. Therefore given that unbounded inference chains are problematic to say the least in a real world environment, then semantic interoperability for semantic inference in real world applications only makes sense within limited domains and contexts that are bounded by the length of practical inference chains. 

This means that knowledge is best represented as a hierarchy of combinations of knowledge from bounded domains or contexts. The associated decision processes look more like decision trees. We only need semantic interoperability (and common schema) at each layer of the decision tree. Information from lower layers on different branches may each use different semantics and schema. This allows us to use sets of semantics. Interoperabilty is limited to the size of each set. Not the set of all sets. This simplifies the task of interoperability in general because it relaxes dependencies between application domains. For example KYC semantics (and schema) may be different than a given supply chain semantics (and schema) thus simplifying the design of each. A given decision process that needs to combine KYC and Supply chain knowledge to reason about both may use a simplified aggregation semantic and (schema). This follows best practices for complexity management in systems design. For example if in my supply chain I have a requirement that I will only extend credit to an entity that has been KYC'd I only need  to include in my supply chain semantic and schema a way of expressing  "has been KYC'd" but do not have to include semantics and schema for every minutia of the KYC decision process.

Semantic interoperability is based on the premise that we can make TRUTHful decisions on complex chains and aggregations of information from multiple VCs from different sources that in turn are from widely different contexts where those VCs may be combined without including the underlying uncertainty. But as we have established that premise is False and the practical value of such unbounded semantic interoperability without uncertainty is very limited. So its a futile task to strive for universal semantic interoperability and extensibility. Don't need it, because we can't use it.

### Systems Design and Complexity Management

One of my fields of research is called Complexity Management. The lab I used to run was called the Advanced Marine Systems Lab. With the emphasis on Systems. In my research, the key to complex system design starts with managing complexity. The most powerful tool we have for managing complexity is to isolate dependencies. This is usually and almost exclusively performed via hierarchical composition. If I can collapse layers in my hierarchy then they were bad layers in the first place. The point of layers is to reduce the number of dependencies that one has to manage. 

The terminology I use is `real` versus `apparent` complexity. The `real` complexity is a measure of the number of dependencies in the system. The `apparent` complexity is a measure of the the number of dependencies one has to care about or manage to make meaningful changes to the system and/or to meaningfully use the system. When one collapses layers there is typically a combinatorial explosion in the number of dependencies that one must manage as a result. This increases apparent complexity not reduces it. 

An optimal design from a complexity management perspective is one that minimizes the ratio of `apparent` complexity to `real` complexity. In its simplest form good layering acts to reduce the combinatorics of dependencies without reducing the expressive or functional power of the system. It's trivial to show how this may happen. A tree of dependencies scales better than a mesh for combinatorial reasons.  

One reason RDF was idealogically so attractive but practically so difficult to use is that its triples with `is-a`, `has-a` predicates appear simple. But when combined for anything but simple applications, the resultant combinatorial explosion of triples makes the resulting graph practically infeasible to use or understand or reason with or otherwise manage. RDF's ideal is that there are no layers, but one big semantic soup of triples with the weakest form of inference possible (`is-a`, `has-a`). Thus RDF's idealistic simplicity of expression actually prevents dependency management layering in any meaningful way. Its ideal ends up maximizing the ratio of apparent complexity to real complexity not minimizing it. So its the wrong kind of simplicity. Tt's **too** simple.  As Einstein is reputed to have said: "Everything should be made as **simple** as possible, **but no simpler**." Better yet is Oliver Wendell Holmes reputed quote "I would **not** give a **fig** for the **simplicity** ***this*** **side of complexity**, but I would give my **life** for the **simplicity** on the ***other*** **side of complexity**." RDF/JSON-LD or more commonly known as *linked-data* ***is simplicity on the wrong side of complexity***.

Whereas a labeled property graph allows dependency isolation via layering. Each edge may have different complex types with multiple properties this allows layering of semantic operations at the edges. Nodes may have multiple properties of different types. This allows layering using distinct combinations of node and edge types. When one looks under the hood at a practical implementation of an LPG, one sees that the graph is usually structured hierarchically thereby better following  the principle of minimizing apparent complexity.  So although it is one graph syntactically, semantically, however it is a hierarchy of layered graphs. And the different combinations of edge and node types create different semantic classes for each of the layers.  So we have interoperable syntax (i.e. interoperability at the semantics of the syntax of the LPG) but not collapsed layers.  Likewise JSON Schema allows one to have layered schema and isolated contextual ontologies of schema. In contrast [schema.org](http://schema.org/) org does not because [schema.org](http://schema.org/) forces the collapse of schematic layering into one universal ontology. 

One of the most common criticisms of RDF is that it requires an A.I. to use it but there is no A.I. yet powerful enough to use it. And humans are incapable of using it but for very simple applications. So if its only good for very simple applications then why use it all? Especially if there are more powerful, easier to use, and already more broadly adopted alternatives?


 
### Proposal Adopt a JSON Schema VC Specification as the Primary Path to Adoption 

What if a small group of the decentralized identity community created a JSON schema VC standard
with immutable content addressable schema definitions? How fast would we get adoption outside the
narrow confines of the existing community?

Questions:
How easy would it be to make secure?
How easy would it be to express VC equivalents?
How easy would it be to create a tooling stack to support it? 
How easy would it be to understand and explain?
How easy would it be to gain adoption of such a standard? 

Answer:
Everything would be easy! Much easier that JSON-LD.

So why are we using a JSON-LD based standard if we care at all about adoption? 

Do we really need to support the edge cases of JSON-LD versus JSON schema as first class properties instead of the special edge cases that they are?

Did we learn nothing from the failure of XML vs. JSON, XHTML vs. HTML5 or the failure of RDF versus any other Graph database? Most if not all of which have higher core application work flow adoption than RDF graphs. (Google SEO is a proprietary edge case)



For content addressable immutable schema, think decentralized resource identifiers (DRI)[https://engagedri.ca/assets/documents/whitepapers/The-opportunities-of-Decentralized-Resource-Identifiers-in-the-research-landscape.pdf] or
(ISCC Content Identifiers)[https://iscc.codes] or compact composable content addressable derivation codes from KERI (KID001)[https://github.com/decentralized-identity/keri/blob/master/kids/kid0001.md]
(KID001 Comments)[https://github.com/decentralized-identity/keri/blob/master/kids/kid0001Comment.md]
applied to JSON schema.


### Tactics: Carrot and Stick

#### Stick is security. 

Correct layering for security means that authenticity (secure attribution)
Happens before one looks at the payload. It happens on the over-the-wire serialization of any payload before you open the payload.
The end state of JSON-LD RDF is antithetical to this concept. Indeed I am disappointed that anyone is seriously considering using JSONLDSignatures on the serialized expanded RDF for security. This approach violates fundamental principles secure protocol layering. Alternatively signing the over the wire JSON-LD before it has been expanded where that JSON-LD includes a reference to mutable schema.org based contexts is not secure do to that mutability. We have to fix up that context to make it immutable and we can do that much easier with JSON Schema which buys us much more adoption than fixing up JSON-LD.

#### Carrot is LPG (Layered Property Graphs) and JSON Schema

Layered property graphs (LPG) are better almost every way for semantic inference than RDF. This is a huge advantage to any standard that wants to convert the payload to some format for inference. One can create a LPG in several open standard graph database that support it and (GraphQl) is an open standard query language for LPGs. Secondly existing SQL and NoSQL databases can be designed support the semantics of LPGs. Further reducing adoption barriers. But in all cases one uses immutable schema for over the white transmission and this schema is JSON Schema which everyone is already using anyway for over the wire schema. So adding immutability is a small extra effort.

## End State for JSON-LD/RDF Security is not scalable nor widely adoptable

When I dug down into Linked Data Proofs specification  I found confirmation of what I was told about the 
end state for securing JSON-LD/RDF documents.

[https://w3c-ccg.github.io/ld-proofs/#dfn-canonicalization-algorithm](https://w3c-ccg.github.io/ld-proofs/#dfn-canonicalization-algorithm) which points to

[https://w3c-ccg.github.io/security-vocab/#URDNA2015](https://w3c-ccg.github.io/security-vocab/#URDNA2015)

 
The proof types are signatures on RDF graphs.  A proper json-ld proof requires that the signed json-ld be derived from a canonicalization of the over-the-wire json-ld. The canonicalized json-ld is derived from a fully normalized RDF graph that is used to produce a fully expanded json-ld serialization (not the over the wire json-ld).

[https://www.w3.org/TR/json-ld11-api/#serialize-rdf-as-json-ld-algorithm](https://www.w3.org/TR/json-ld11-api/#serialize-rdf-as-json-ld-algorithm)

  
Quoting, "The default canonicalization mechanism is specified in the RDF Dataset Normalization specification, which deterministically names all unnamed nodes. "

One way to serialize an RDF graph is to convert it to a normalized fully expanded JSON-LD serialization.

This is not the over the wire JSON-ld but the JSON that has been fully expanded. So a signature on the over the wire JSON-LD (AKA the verifiable credential itself) is not verifiable on the RDF serialization JSON-LD and vice-versa.

  
Indeed The following are all signatures on serialized RDF graphs.: ([https://w3c-ccg.github.io/security-vocab/#URDNA2015](https://w3c-ccg.github.io/security-vocab/#URDNA2015)) 

  
- GraphSignature2012
- LinkedDataSignature2015/LinkedDataSignature2016
- MerkleProof2019
- Ed25519Signature2018
- JsonWebSignature2020/JsonWebKey2020
- BbsBlsSignature2020/BbsBlsSignatureProof2020

  
I always thought that JsonWebSigantures were signing the over the wire  JSON-LD document itself (AKA the Verifiable Credential) not the RDF serialization (AKA a fully expanded JSON-LD derived from the RDF which in turn was derived from the VC)

Moreover the expansion process is a performance hog. I talked to a couple of developers who are using the expansion library and they volunteered that its extremely non-performant. This by itself makes
it non-scalable and makes it an adoption show stopper when it must happen before signature verification.

There appears to be one special case exception that allows signatures on the over the wire JSON-LD.
In other words, JWS signatures on the VC itself are a special case i.e. the detached and unencoded payload variant but are not secure due to the mutability of the @context in the over-the-wire json-ld.
Immutability is not guaranteed by JSON-LD until after full normalization, expansion and re-serialization. Then an only then is there any guarantee of immutability according to the standard.
Any thing else is a special case exception that is a variance with the end state security design of the JSON-LD Signature specifications.
Quoting:

"
The JWS used with Linked Data Signatures is the detached and unencoded payload variant. See the links below for details:

-   [JSON Web Signature (JWS) Unencoded Payload Option](https://tools.ietf.org/html/rfc7797)
-   [JSON Web Signature (JWS) and JWS Detached for a five-year-old.](https://medium.com/swlh/json-web-signature-jws-and-jws-detached-for-a-five-year-old-88729b7b1a68)
"



Certainly we could define a new standard for VCs that required immutable schema (i.e. not schema.org) but that is such a fundamental misalignment with the JSON-LD community that it serves no purpose. We might as well use the already much more widely adopted JSON schema instead and impose an immutability constraint or our use of JSON schema.

My contention is that we want security on the over-the-wire data not some eventually expanded canonicalization. Otherwise we limit adoption dramatically because we impose an insurmountable interoperability barrier by requiring RDF canonicalization/expansion as a hard dependency on any security (signature verification).

## Immutability Requirements on JSON Schema

All schema definitions must be securely immutable. Identifying schema with versioned identifiers does not guarantee immutability. Secure immutability comes from either embedding the schema definition in the VC embedding a digest of the schema definition in the VC. In either case the signature on the VC will not verify if the schema is changed. A digest is a verifiable cryptographic commitment to the schema stored elsewhere. When embedding a content digest of the schema definition, the schema definition itself must be stored elsewhere in a highly available repository or registry or database. Any change to schema results in a new content addressable instance of that schema stored in that repository. To clarify, the signature on the VC makes a verifiable commitment to the digest which in turn makes a verifiable commitment to the immutable schema. The VC will not be verifiable if the actual schema definition is not found in the repository. This is why it must be highly available.

Immutable schema definition require a content addressable identifier name space. One will have to be designed for JSON schema in order that they be known and registered as immutable. Any issuer is responsible for providing or designating a highly available source or repository for the schema they reference via such identifiers in VCs. Otherwise the issuer may use an embedded copy of the schema in the VC making it self contained and removing the requirement for a highly available schema repository.

This level of flexibility will make adoption must easier as high risk high performance applications that want to avoid the performance hit of an external repository can always embed schema definitions (they may still include the identifier allowing semantic interoperability) or attach the schema definition to the VC when transmitted or cache the schema. Because the schema is immutable all three approaches to performance optimization and risk mitigation are secure.

## Natural Canonicalization

One of the touted reasons to use JSON-LD RDF is that it provides a canonical serialization for key:value mappings (hash tables, dictionaries).  By canonical we mean reproducible or stable. A key:value mapping is a self-describing data structure in that each value is labeled by a key. One can infer the purpose of the associated value from a descriptive or well known key (label). Historically these key:value mappings for performance reasons used something called a hash table. A simple hash table does not provide a stable ordering of its keys when serialized. This means that when serializing and deserializing one may not have a stable serialization for a verifiable digital signature. 

For example a JSON document is a serialization of a JavaScript Object. Because under the hood Javascript objects are based on a hash table structure which does not guarantee a stable order of appearance, a signed a serialization may not guarantee its later verifiability. One reason put forth for using JSON-LD with an expanded RDF is that expanded RDF has a canonical form that when re-serialized is stable. In other words JSON-LD/RDF provides a standard canonical serialization whereas JSON does not.

This may have been true years ago when JSON-LD was first conceived but it has not been true with respect to ECMA standard JSON serializtion for some time.
The EcmaScript standard Javascript (since ES6) supports stable property creation order serialization and deserialization of JSON via the stringify method and Reflect.ownPropertyKeys(). More importantly, as of ES11 (2020) all JavaScript objects preserve property creation order in stringify(). In other words as of ES11 JSON objects are canonically ordered by default by property creation order. This means serialization and deserialization natively preserves this canonical ordering. This means that a trivial natural canonicalization standard is to use property creation order.

The JSON-LD/RDF ordering is alphabetic. This is a much more cumbersome ordering. It does not allow one to structure documents with the most logical understandable ordering of fields. Property creation order makes any logical ordering possible merely by creating the properties in that logical order when creating the document.

This now puts JSON on equal footing with the predominant languages that have for some time already adopted property creation ordering for mappings (hash tables). For example all dictionaries in Python since version 3.6 are ordered by property creation ordering. (A python dictionary is the Python equivalent of a JavaScript object for serialization purposes). Python's standard library prior to 3.6 also supported an ordereddict type. Languages such as C or Rust that use fixed data structures instead of hash tables have always supported ordered serialization.

To restate, for some time (ES6 or later) it has been trivial for a JavaScript Object ser/deser to reproduce and preserve property creation order.  This makes property creation (logical) ordering the natural canonical serialization ordering. Although, Javascript was a late comer to the ordered map party there is no longer a need to use JSON-LD/RDF's much more cumbersome approach.


This property creation order canonical serialization is part of the KERI spec.

[KERI KID003](https://github.com/decentralized-identity/keri/blob/master/kids/kid0003.md)

For more rationale on serialization approach and JSON stringification, see [KERI KID003 Commentary](https://github.com/decentralized-identity/keri/blob/master/kids/KID0003Commentary.md).


Here are the relevant details.


### JavaScript Ordered Object Property Serialization (stringify())

The following code snippet shows how to enforce object property creation order preserving serialization using JSON.stringify for ES6-ES10. Not needed for ES11 or later.

### Code Snippet

```js
"use strict"

// Spec http://www.ecma-international.org/ecma-262/6.0/#sec-json.stringify
const replacer = (key, value) =>
  value instanceof Object && !(value instanceof Array) ?
    Reflect.ownKeys(value)
    .reduce((ordered, key) => {
      ordered[key] = value[key];
      return ordered
    }, {}) :
    value;

// Usage

// JSON.stringify({c: 1, a: { d: 0, c: 1, e: {a: 0, 1: 4}}}, replacer);


var main = function()
{
    console.log("Running Main.");

    let x = { zip: 3, apple: 1, bulk: 2, _dog: 4 };
    let y = JSON.stringify(x);
    console.log(y);
    console.log(Object.keys(x));
    console.log(Object.getOwnPropertyNames(x));
    console.log(Reflect.ownKeys(x));
    console.log(replacer);
    let w  = {w: 1, a: x, c: {e: 2, b: 3}};
    let z = JSON.stringify(x, replacer);
    console.log(z);
    let v = JSON.stringify(w, replacer);
    console.log(v);

}

if (require.main === module)
{
    main();
}


//nodejs % node play.js
//Running Main.
//{"zip":3,"apple":1,"bulk":2,"_dog":4}
//[ 'zip', 'apple', 'bulk', '_dog' ]
//[ 'zip', 'apple', 'bulk', '_dog' ]
//[ 'zip', 'apple', 'bulk', '_dog' ]
//[Function: replacer]
//{"zip":3,"apple":1,"bulk":2,"_dog":4}
//{"w":1,"a":{"zip":3,"apple":1,"bulk":2,"_dog":4},"c":{"e":2,"b":3}}
```

### Background

We may impose ordered serialization with the JSON.stringify method by using the replacer option and the \[\[ownPropertyKeys\]\] internal method added in Javascript ES6 exposed as Reflect.ownKeys(). This method is provided in JavaScript ES6 (2015). \[\[ownPropertyKeys\]\] uses the property creation order. At the time of ES6, JSON.stringify was not required to use the \[\[ownPropertyKeys\]\] iteration order (ie property creation order). This appears to have been fixed in later versions of ES. (See the discussion below). As of this writing nodejs v14.2.0 and the latest versions of Chrome, Safari, and Firefox all appear to preserve property creation order in JSON.stringify() without having to use  Reflect.ownKeys(). This means that we can depend on and use that ordering in the specification. For earlier versions but still with support ES6, a custom replacer function for JSON.stringify(x, replacer) will ensure that property creation order is used. Earlier versions that do not support ES6 may still work with an ES6 polyfill if that includes support for \[\[ownPropertyKeys\]\].

[Does-es6-introduce-a-well-defined-order-of-enumeration-for-object-properties/30919039](https://stackoverflow.com/questions/30076219/does-es6-introduce-a-well-defined-order-of-enumeration-for-object-properties/30919039)

In ES6 the stringify order is not required to follow the ownPropertyKeys (property creation order) ordering but this will be fixed in ES 2020 == ES11. It appears that the Babel polyfill as of 2020 already supports ES2020 already.

[Metaprogramming in ES6: Part 2 - Reflect](https://l.antigena.com/l/DX6b1wizs71Ijc-eLjrl_reEdyDi0rb~jWMlu~s-bedYFHABJnPkRhFRynVLeqwmfFJqdN_xpMxwt6Sh_bfOpYN3ayk2ahUCyfNYGmg-ZPEW697WtcmWK199PVqsVhieOEj7z7oiXplT1qiP75eLC-5VBVVXmEpHXZp99X73)

Reflect.ownKeys ( target )

This has already been discussed a tiny bit in this article, you see Reflect.ownKeys implements \[\[OwnPropertyKeys\]\] which if you recall above is a combination of Object.getOwnPropertyNames and Object.getOwnPropertySymbols. This makes Reflect.ownKeys uniquely useful. Lets see shall we:

```js
var myObject = {
  foo: 1,
  bar: 2,
  [Symbol.for('baz')]: 3,
  [Symbol.for('bing')]: 4,
};

assert.deepEqual(Object.getOwnPropertyNames(myObject), ['foo', 'bar']);
assert.deepEqual(Object.getOwnPropertySymbols(myObject), [Symbol.for('baz'), Symbol.for('bing')]);

// Without Reflect.ownKeys:
var keys = Object.getOwnPropertyNames(myObject).concat(Object.getOwnPropertySymbols(myObject));
assert.deepEqual(keys, ['foo', 'bar', Symbol.for('baz'), Symbol.for('bing')]);

// With Reflect.ownKeys:
assert.deepEqual(Reflect.ownKeys(myObject), ['foo', 'bar', Symbol.for('baz'), Symbol.for('bing')]);

```


### Stringify References

-   [https://github.com/tc39/proposal-for-in-order](https://github.com/tc39/proposal-for-in-order)
-   [https://l.antigena.com/l/pXlB8VCSaq1IkjIbrXeODcb~8JQ2a0cOtATBCdIUQW6bwZKwzTDf5PGozlLBjaNc-BFxrD9wNjXhTJgjhTZ2ONynSib-uIylaaVcVtqacyqGdLIU6Xff0qkXda2nEv1FSW0wHqP6LsRstrLOkLaXqk9~qI](https://l.antigena.com/l/pXlB8VCSaq1IkjIbrXeODcb~8JQ2a0cOtATBCdIUQW6bwZKwzTDf5PGozlLBjaNc-BFxrD9wNjXhTJgjhTZ2ONynSib-uIylaaVcVtqacyqGdLIU6Xff0qkXda2nEv1FSW0wHqP6LsRstrLOkLaXqk9~qI)
-   [https://2ality.com/2015/10/property-traversal-order-es6.html#traversing-the-own-keys-of-an-object](https://2ality.com/2015/10/property-traversal-order-es6.html#traversing-the-own-keys-of-an-object)
-   [https://www.stefanjudis.com/today-i-learned/property-order-is-predictable-in-javascript-objects-since-es2015/](https://www.stefanjudis.com/today-i-learned/property-order-is-predictable-in-javascript-objects-since-es2015/)
-   [https://medium.com/@robertgrosse/how-es6-classes-really-work-and-how-to-build-your-own-fd6085eb326a](https://medium.com/@robertgrosse/how-es6-classes-really-work-and-how-to-build-your-own-fd6085eb326a)
