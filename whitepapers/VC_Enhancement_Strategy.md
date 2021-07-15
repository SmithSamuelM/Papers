---
tags: ACDC, KERI, JSON-LD, RDF
email: sam@samuelsmith.org
version: 1.00
---

# VC Spec Enhancement Proposal

[![hackmd-github-sync-badge](https://hackmd.io/rnY5xiXESWqWBlyDPHDorA/badge)](https://hackmd.io/rnY5xiXESWqWBlyDPHDorA)


Strategic Technology Choices vis-a-vis the Linked Data (JSON-LD/RDF) End State.


## Barriers to Adoption of Linked Data VCs

The purpose of this paper is to capture and convey to a braoder audience my increasingly worrisome concerns about the adoption path for Verifiable Credentials (VCs). My concerns began with the security limitations of VCs that use *Linked Data* (otherwise known as JSON-LD/RDF). My concerns have since extended to the semantic inference limitations of *Linked Data*. My concerns may be expressed succinctly as, ***the VC standard appears to be an adoption vector for Linked Data not the other way around***. My overriding interest is that the concept of a VC as a securely attributable statement is very powerful and attractive one and therefore should be widely adopted. We should therefore be picking the best technologies that best support broad VC adoption not the other way around.

A VC is a member of a more general class of data that may be described as *securely attributed data* or *securely provenanced data* or more simply ***authentic data***. My overriding interest for authentic data is to reach the end state where we have reached universal adoption. This would finally fix the broken internet. Therefore the underlying technology and adoption strategy must be compatible with that end state. We may never reach that end state but if so, it must not be due to any technical limitation nor a bad adoption strategy.

In my view the primary role of any identifier system is to provide secure attribution. Another way of stating this is that the identifier system solves the secure attribution problem. The core of such a solution is the establishment of control authority over an identifier where such control authority consists of a set of asymmetric (public, private) key pairs for non-repudiable digital signature(s) (PKI). Other cryptographic operations may serve a similar role but the simplest most universal method of establishing provable control authority is an *asymmetric key pair based* digital signature on some statement attributed to an identifier. Only the controller of the private key(s) that control an identifier may make a verifiable non-repudiable commitment to some statement via such a signature. Any second party may verify that signature given the public key(s). The role of any identifier system is to securely map identifiers to the authoritative public key(s)s in order to cryptographically verify the signatures and thereby make secure attribution. To elaborate, secure attribution via digital signatures is a cryptographic way of establishing the authenticity of any signed statement.

KERI is one such identifier system. DID identifiers that belong to the name space of some DID method are another such system. 

Once one can make secure attribution then one can build a secure transactional system on top of that secure attribution system. One such transactional system is a Verifiable Credential (VC) but that is only one. The end state for decentralized identifier systems is to enable all transactional systems to be securely attributed using that decentralized identifier system.

This goal of universal secure attribution is very bold and ambitious. It therefore requires a realistic and sane approach to adoption. Any anticipated adoption barrier that would or could reasonably and significantly prevent reaching that end goal must be carefully considered. More specifically, any required technology choice that induces an adoption barrier that may reasonably prevent such broad adoption should be contra-indicated unless there is no other more viable alternative.

### Adoption Strategies for New Technology

Often a new technology may choose as its adoption strategy to leverage some other pre-existing technology, especially pre-existing open standard technology. The reputation of the pre-existing standard may reduce the perceived risk of the new technology enough to gain more rapid adoption of the new tech. Often the new tech, by virtue of its new features is not an exact fit for the pre-existing standard. However the much larger adoption status of the pre-existing standard means that adapting it or modifying it even if not the most elegant approach pays off with accelerated adoption of the new tech. Over time the new tech may grow big enough to incentivize the pre-existing standard to change to better accommodate the new tech.

However,if the pre-existing tech is at best a limited adoption standard that fits a narrow niche where its technology is either too special case or is not scalable enough that it will likely never be widely adopted or its market penetration has already peaked, then it may be a poor strategic choice for the new tech to leverage. Moreover if the old standard and the new tech communities are not fundamentally aligned in terms of purpose and approach then there may be too much friction to ever attain first class status and thereby receive first class support for that new tech from the pre-existing community. This misalignment friction would make the adoption journey net-counter-productive for the new tech.

#### Relation to Linked Data (JSON-LD/RDF)

RDF (upon which Linked Data depends) is old tech that long ago peaked in its adoption curve. JSON-LD is an attempt to refresh that old tech and bring new life to its adoptability. Based on my investigation into the end state for JSON-LD and where the JSON-LD community intends to go, I have concluded that its adoptability will never scale to the level needed nor be net advantageous to the goal of supporting the majority of use cases for securely attributed (authentic, provenanced) data, of which VCs are a seminal example. The purpose of this paper is to better understand and support the validity of that conclusion. 

***In short, the fundamental flaws in using Linked Data (JSON-LD/RDF) as an adoption vector for securely attributed data are as follows:***
 - Linked Data emphasizes *interoperable extensible semantics* at the cost of *interoperable security and trust*. 
 - Linked Data uses the weakest form of semantic inference that has long since been relegated to legacy status, i.e. it does not represent the state of the art for semantic inference.


These two fundamental technical flaws are why I believe Linked Data (JSON-LD/RDF) has a hard adoption ceiling that is much too low to satisfy the full potential of broad VC adoption. I also see strong evidence that adoption barriers induced by these technical flaws will become harder to overcome going forward because growth is happening outside of JSON-LD/RDF at such a rapid pace that there is no way for JSON-LD/RDf to ever catch up. It's already lost the adoption battle.

The best case that we could hope for given the current VC spec and the bias of the JSON-LD community is to change the spec to support non schema.org (non RDF) schema. This would be, at best, an uneasy compromise equivalent to the shotgun wedding compromise reached by the DID spec where JSON-LD is not required. A second-best case would be an exception where a special form of immutable schema (i.e. not schema.org) is used but with JSON-LD syntax. But such an exception is antithetical to the core values of the JSON-LD community and would be a source of never ending friction. Such friction by itself poses a significant adoption barrier as discussion would quickly degrade into unproductive disagreement and political posturing as it often does in the DID WG for the same reasons.

To better understand the core problem mentioned above, I'll first review the prioritization of system design principles associated with solving interoperability for both the secure attribution problem and the semantic normalization problem (i.e. interoperable security and interoperable semantics).


### System Design Principles

Interoperability layers:
- Security first always.  
- Semantics second always.

This layering may be phased as:  
***Interoperable secure attribution of data first; interoperable semantics of data second.***  
  
If we want to move data across trust domain boundaries then we need an interoperable protocol that enables the secure attribution (authenticity) of the data at each boundary.  In other words, *secure attribution must be established whenever or wherever data crosses a trust domain boundary. This secure attribution must be maintained across any subsequent trust domain boundaries (i.e. end-to-end provenance)*. If we have a universal decentralized protocol for establishing secure attribution then those trust domain boundaries can belong to decentralized entities with no loss of security.

Following best practices for security system design means we want to discard any data
that is not securely attributed as soon as possible in our processing stack. Pushing secure attribution further up in the stack exponentially expands the attack surface which includes DDOS attacks and transaction malleability attacks. It also exponentially expands the vulnerability surface for secure code delivery and execution.
  
The principle may be summarized as: 
***Only after we have proven secure attribution may we move data across a trust domain boundary, then and only then should we address the secondary problem of moving the data semantics across that trust domain boundary.*** 
There is no point in moving the semantics across a trust domain boundary for data that will be discarded because it was not securely attributed. Doing so puts the proverbial *semantic cart in front of the security horse*.

Correct design always places interoperable security in the primary position and interoperable semantics in the secondary position.   

To reiterate this layering of security first semantics second follows best principles for protocol design layering. Session and Presentation layers come before the Application Layer.
Payload semantic verification and integration happens at the Application layer not at the Session and Presentation layers where secure attribution happens. Any good IT protocol security person would be surprised at any suggestion that secure attribution of data should happen after the semantic integration of that data into some backend database (aka an RDF graph).

***Linked Data inverts this prioritization by putting security behind semantics*** (aka RDF expansion and normalization first, signature verification second). Given the number of proposed expanded RDF serialization signature standards, the JSON-LD community is blithely unashamed to propose such an inverted security layering. See the section below on the Linked Data community's planned end state for JSON-LD/RDF security.

Attempting to repair this by using hash links on the schema.org schema does not guarantee the proper form of immutability that ensures an issued VC is verifiable. It only enables detection of mutability after the fact. This means that when the signatures are on the over-the-wire JSON-LD, some entity besides the issuer is enabled to *effectively revoke* any Linked Data VC.  The VC is thereby *effectively revoked* because it is not verifiable due to the mutation of the schema. The issuer must then revoke and reissue the VC, not for any valid reason on the issuer's behalf, but merely due to the mutation of the schema which is not under the control of the issuer. This makes schema.org an attack vulnerability. It creates a treasure trove for attacks. ***Any attacker who hacks the schema.org registry could force all issued VCs everywhere to be unverifiable***.

As a result we lose interoperable security in any reasonably scalable way which means we can't solve the problem of universally secure attribution of data across and between trust domains.  
  
Instead we have Balkanized our trust domains into, at the very least, RDF and non-RDF. And RDF will always be dwarfed by non-RDF for many reasons not the least of which is the inverted (bad) security design described above. 

Web 3.0 ten years ago meant semantic-web-enabled (Linked Data).  
Web 3.0 now means crypto-enabled for trust and security.

*Old legacy Web 3.0 tech was semantics first. New Web 3.0 tech is security first.*
The reason should be obvious. 

## Survey of the Market for Semantic Interoperability

Given we place interoperable semantics in the secondary position to interoperable security, what technology choices does the market provide for interoperable semantics?

See these references for terminology: 
[Knowledge Graphs 2021 survey](https://arxiv.org/pdf/2003.02320.pdf)  
[Knowledge Graph Wikipedia](https://en.wikipedia.org/wiki/Knowledge_graph)  

### Semantic interoperability levels:

- API level 
    - Consume data using OpenAPI JSON schema, custom schema  
    - Single backend database SQL, NoSQl, Graph  
- Data Lake level  
    - - Consume JSON schema or custom schema. Map to data lake   
    - Multi backend databases, cross database schema enforcement  
- Knowledge Graph level  
    - Depending on the graph database: consume JSON Schema, custom schema or schema.org schema  
    - Graph databases (Property Graphs, Weighted Directed Edge Graphs, Is-a has-a triples)  
        - Open property graphs (Neo4j, Arrango, various Cypher supporting or GraphQL wrapped databases  
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
        - mine legacy open RDF graphs and convert to directed edge property graphs to perform   inference.



### Market Segments for Semantic Interoperability

Below is a ranking of of relative adoption levels of semantic interoperability technology. Highest is first, lowest is last. Notice that JSON-LD is a distant last. I will explain why below.

- API level: 90%+ json schema or custom schema (relative to the other two)
- Data Lake level:  (relative to knowledge graphs)
- Knowledge Graph level ranking by market adoption: (relative to data lakes)
    -  Proprietary knowledge graphs for social networks or page rank 
        - Consume JSON schema or custom schema serializations with one exception)
    -  Open labeled property graphs (LPG) (such as Cypher) for social networks
        - Consume JSON schema or custom schema serializations)
    -  Repurposed SQL or NoSQl to act like a labeled property graph
        - Consume JSON schema or custom schema serializations)
    -  Machine learning custom or proprietary weighted edge knowledge graphs
        - Consume JSON schema or custom schema serializations or mine other knowledge graphs)
    -  Linked Data RDF/JSON-LD (distant last)
        -  Consume JSON-LD or other RDF serialization


### Semantic Inference and Automated Reasoning

#### Semantic Inference

Machine learning and automated reasoning are types of inference mechanisms that may be applied to knowledge graphs when those graphs support uncertainty measures. The primary mechanism for representing uncertainty in a knowledge graph are weighted edges, aka,
weighted directed edge graphs. A generalization of a weighted directed edge graph is a property graph that supports as edge properties, directed weights. In such a generalized labeled property graph (LPG), each edge type may have multiple properties and may have one or more weights to represent different forms of uncertainty. These include, similarity (ambiguity, distance), probability (likelihood), and possibility (precision, plausibility, etc). The associated branch of mathematics is called *measure theory*. RDF triples provide no comparable mechanism for representing uncertainty. This may be one of the main reasons the semantic web died. The main consumers of semantic web information, the data aggregators, use machine learning on their graph models. They all use much more powerful graph models than RDF. These more powerful graph models support uncertainty measures which are necessary to machine learning derived inference. In other words RDF may have become legacy technology largely because it does not support the most useful and powerful forms of inference. 

For example one problem in the semantic mapping of knowledge is called ***polysemy***. In the IT world the ability to map one corpus of knowledge is severely constrained by *polysemy*. One way to address this is to use similarity measures to characterize the degree of polysemy.  [Termediator](https://dl.acm.org/doi/10.1145/2656434.2656443). To clarify, the measures of polysemy are similarity measures (a type of uncertainty measure). The process is called terminology mediation. I often find myself in discussions with RDF advocates where they assume that polysemy is non-existent and hence don't appreciate any need for terminology mediation. They believe erroneously that a single ontology of terms with zero uncertainty is practically attainable in the real world for anything but extremely narrow isolated contexts. They ignore the inescapable uncertainty that polysemy measures over the set of all contexts.

Labeled property graphs (LPGs) may have properties on both the nodes and the edges of the graph. This allows easy modeling of complex relationships. In contrast RDF does not allow properties on either nodes or edges. Each node has only one property and all edges must be of the same type. One must jump through hoops to model complex relationships in RDF. RDF one property per node results in many many more nodes and edges for even simple graph constructs. One must employ workarounds like reification. Querying RDF graphs becomes complicated very quickly. Absent the need to represent all edges and nodes as URIs/IRIs, I doubt that anyone would have thought that using triples was a good starting point for semantic inference. They would have started with an LPG instead. 

To be fair, the only application where RDF (Linked Data) may be well suited for inference is the semantic web which is constrained by the semantic limitations of graph edges that are URI/IRIs. For every other application of semantic inference, labeled property graphs (LPGs) are more convenient and more powerful (social networks). For machine learning especially, weighted directed edge graphs (LPGs with weighted edges) are essential. It's far more difficult to reason with RDF triples than almost any other type of semantic graph. Read any tutorial on modeling and inference that compares LPG to RDF to see how much simpler and intuitive it is to model and infer with LPGs vs RDF graphs.

As an example from the SSI (Self Sovereign Identity) space, recall that the original white paper [Identity System Essentials](https://github.com/SmithSamuelM/Papers/blob/master/whitepapers/Identity-System-Essentials.pdf) defines an identity as an identifier plus attributes. An identity of this type may be represented succinctly and clearly as one node in an LPG. Whereas in an RDF graph, one identity, an identifier plus attributes, requires many nodes and edges, with one triple per attribute. Also recall that the white paper also describes the concept of an identity graph that maps the different types of relationships between identities. This may also be conveniently modeled with an LPG. The edges have properties of different types to distinguish different types of relationships. But in RDF there are no properties on edges. Different types of relationships require cumbersome workarounds. Thus RDF is poorly suited to decentralized identity relationship modeling. Why use an old archaic tool built for a different purpose when there are several, more widely adopted, more widely supported, more performant, more suitable, non-proprietary tools based on LPGs? It's no wonder that JSON-LD/RDF adoption is in a distant last place in terms of market adoption when compared to the other knowledge graph technologies.

To elaborate, standardizing on RDF forces one to express semantics in triples which only support the most cumbersome and least expressive of all the inference types. ***As a result, RDF is the least adoptable choice among all the types of semantic inference***. The most adoptable would be open labeled property graphs (LPGs). 

#### Automated Reasoning

I used to teach graduate classes in automated reasoning. In those classes, a property we examined for each automated reasoning technique was its ability to both represent (express) and reason about knowledge with uncertainty. Real world practical knowledge has uncertainty in it. Both the expressive and reasoning power (of which inference is a type of reasoning) of a given technique is measured by how well it handles uncertainty.  

To better illustrate, consider this triple:    
`Joe is Tall`  
(or more correctly as a triple but more awkwardly)  
`Joe is a TallPerson`  

How does one represent `Tall` (TallPerson)? The adjective `Tall` is inherently uncertain. One way to represent this uncertainty is with a fuzzy set. For example one can generate a fuzzy set that reasonably approximates `Tall` expressed over the domain of heights by taking the histogram of heights of the general population, extracting the half of the histogram that is above the mean, normalizing it on a scale of zero to one and inverting it. 

Now suppose that one only has Boolean truth valued nodes and edges in an RDF graph? How then does one represent the inherent uncertainty in the concept `Tall`. Especially where each node and edge must be expressed as a URI?  One would have to do complex backflips at the very least. 

Now let’s add to our corpus of knowledge on Joe by adding the triple. 
`Joe is a BasketballPlayer`  
Given this added information, does the original fuzzy set taken from the general population histogram of heights still accurately convey the uncertainty of the `Tallness` of Joe?  Not so much because basketball players on average are much taller than the general population.  So if one has a frozen definition of `Tall` then one wouldn’t have said `Joe is Tall` in the first place. One would have said `Joe is Very Tall`.  

So how does one model `Very Tall` with a triple? Does one create another node that is the `VeryTallPerson` node. Better would be to use a graph model that allowed an edge with a property that represents the modifier `Very`. The latter means that one can use `Very` as an edge for any number of objects that are uncertain because `Very` may be an operation that shifts a fuzzy set along its domain.   But with RDF one can’t have edges that are operations.  RDF is limited to `is-a` or `has-a` edges for its triples not `very is-a` or `very has-a`. So one is stuck with doing backflips again to represent even extremely simple concepts like, `Joe is Very Tall`. 


Now let's add some more information to our corpus with the triples:
`Joe is Male`  
`Eva is Tall`  
`Eva is Female`   
Is the `Tall` for Eva the same as for Joe. The histogram for heights is different for females and males. Recall that uncertainty is a permanent inherent property of knowledge and `is-a` or `has-a` triples have no practical way of representing it. 

Now lets add to to our corpus the triple:
`Bob is not Tall`  
You may be surprised to learn that in RDF there is no edge negation operation, i.e. `not`. One can’t make statements like this without doing backflips. One could say instead, 
`Bob is Short`  
But `Short` is not the same as `Not Tall`.  So one could create a new node to represent `NotTall`. Better would be to use a graph type (LPG anyone?) that allowed edges with an operator that is the negation operation, `Not`, (sorry RDF!).

As this simple example illustrates, for any ***practical*** knowledge representation with uncertainty, bare triples are woefully inadequate. In my graduate classes, we would spend a couple of days using triples as a basic introduction to knowledge representation and then move one to more powerful representations. There are so many richer. more expressive, more powerful ways to represent knowledge and reason with it than the painfully limiting `is-a` or `has-a` triples of RDF. Any general real world knowledge application will be infused with uncertainty. Given that understanding, it is hard for me to appreciate why anyone would select RDF over LPG. Legacy support? Impractical ideology? Uninformed? Misinformed? ...


### RDF Impracticality Presaged Its Own Demise

Frankly it was naive to propose RDF triples as the knowledge representation format for the [The Semantic Web] (https://web.archive.org/web/20171010210556/https://pdfs.semanticscholar.org/566c/1c6bd366b4c9e07fc37eb372771690d5ba31.pdf) in the first place. By 1999 it was already well known in the automated reasoning community that using triples for semantic inference was problematic in any real world application. Anyone with a solid foundation in the field of automated reasoning at that time would have preferred any of several other techniques that at the very least provided some way to better handle uncertainty. 

As a side note, an LPG can be implemented with URIs if one uses the query string to provide the edge properties and also uses JSON at the resource end-point to provide the node properties. Imagine if Tim Berners Lee picked an LPG instead of RDF in the first place. Maybe the semantic web would still be a thing.

This analysis does not include all the myriad other types of semantic inference that are not based on knowledge graphs at all. Moreover all these other types of semantic inference would only consume schema at the API level. Examples of semantic inference techniques that are not graph-based include deep learning, case-based reasoning, fuzzy logic, Bayesian belief networks, non-graph rule base systems, etc. By limiting ourselves to standards that insist on the edge case that is Linked Data (JSON-LD/RDF) we impose extreme friction on everyone else.

But this is not just my opinion. I have read several articles from those in the RDF world that also conclude that RDF is a dead-end technology choice. The article below from a world class expert in data management who fielded some renowned applications at scale (which is important) using RDF technology, describes why he abandoned RDF. The reasons include its practical limitations (both semantic and performance) and the fact that there are now much better solutions such as LPGs.  [Why-I-dont-use-semantic-web-technologies-anymore-even-if-they-still-influence-me](https://www.lespetitescases.net/why-I-dont-use-semantic-web-technologies-anymore-even-if-they-still-influence-me)

I find the lessons learned telling. Paraphrasing: *The ideal of RDF is belied by its practical limitations. The main limitation being that RDF is too hard to use. If no one will use it then it is practically useless. If one really truly needs a knowledge graph then use a labeled property graph which has broad industry adoption.*

Quoting:
> So currently, Semantic Web technologies are only used to retrieve and process the data from Wikidata so as to enrich our reference vocabularies for people and places. If the need arises and / or if there is a political intent, we can still consider exposing all or part of our data as Linked Data and through a sparql endpoint, but otherwise, we will prefer dumps in simple formats, to ensure greater data reuse. Thus, while being impregnated with all the reflection and the contributions resulting from the Semantic Web technologies, our project does not use them any more.

My survey of other attempts to implement RDF at scale have ended this same way. In summary, Linked Data was a great ***ideal*** but ***fails in practical use***. In short it's a dead end. It's a siren song that ends up drowning its adherents. Time to move on. More importantly there are now better technologies to use instead.  Given this strong conclusion, I am surprised that anyone besides ideological RDF diehards, or the uninformed, or misinformed would select RDF for any new standards meant for broad adoption. 

***I always operated on the assumption that as long as I was not forced to use Linked Data (RDF), I didn’t need to care if someone else preferred it, but that assumption has been proven wrong.*** It seems there is an unreasonable partisan bias within the standards community that threatens to force me and everyone else to use Linked Data for VCs. ***That would be both an adoption dead end and a security disaster for VCs.***   


### There is only one narrow market adoption use case for JSON-LD: Google SEO

According to the JSON-LD site, 10 million sites use JSON-LD. However upon closer examination I found that these are focused in only one narrow use case: to provide JSON-LD with schema.org encoded content for Google SEO of a given web site. Linked data is not for any semantic inference on the sites themselves. **Most, if not all, of those sites also use either JSON Schema or custom schema (not schema.org) for every other schematic purpose on their site. Most, if not all, do not use an RDF graph database backend for any purpose! Many either use a different graph database backend or no graph database backend at all.**
In stark contrast to schema.org, JSON Schema is part of the OpenAPI spec [Swagger Open API](https://swagger.io/docs/specification/data-models/keywords/).
As a result, JSON Schema is already the de facto web standard for interoperable schema. Given its dominate position it is unreasonable to believe that JSON-LD with schema.org would ever displace it. JSON Schema has already won the adoption battle.

This situation is comparable to the XML vs JSON battle. In almost every technical way except simplicity and compactness, XML is better than JSON as a serialization format. But because JSON is *good enough* for 99% of use cases its simplicity became the deciding feature.  JSON won the battle. 
Likewise Linked Data (JSON-LD/RDF/schema.org) may superior in many ways to JSON plus JSON Schema but JSON Schema is already good enough for 99% of the use cases which it already serves and is also much simpler. ***JSON Schema has already won that battle. The decentralized identity community just hasn't realized it yet.***

Given its technical limitations, the only proper application of Linked Data was the semantic web. The only significant current consumer of Linked Data is Google which they ingest for their proprietary Google Knowledge Database (which is not RDF under the hood). Even in this case it is optional. One of Googles APIs allows it to consume JSON-LD data but other APIs do not. 

In general, any knowledge graph database must be dumbed down into an RDF representation in order to fit the extreme constraints of the semantic web. But that dumbing down removes much of the expressive power of the the original graph. Linked Data (JSON-LD/RDF) is poorly suited for any other purpose but the now defunct semantic web. As it now stands almost nobody is actually using the open semantic web in any meaningful way.  Most if not all of the other open standard graph database formats are more powerful, more performant, and more popular than RDF. They all enable semantic interoperability by consuming JSON schema and more importantly they provide more powerful, more convenient, and more performant semantic inference than RDF. 

Nonetheless, I strongly believe that the ultimate goal is not semantic interoperability nor is it security interoperability. Both of these are just means to another end, automated reasoning (of which semantic inference is one type of automated reasoning). Securely attributed data via VCs is necessary so one can use automated reasoning technology to automate workflows on data that crosses trust domain boundaries. 

### Selection Bias

Based on my survey the decentralized identity community is suffering from a severe case of selection bias induced by the disproportionate participation of members of the more vocal Linked Data (JSON-LD/RDF) community.

The broader decentralized identity community majority is allowing itself to be stampeded by the minority Linked Data community into an adoption box canyon because it has been blinded to other better options by the selection bias of that more vocal minority. 

### Designing Semantic Interoperability with the ***Ultimate*** End in Mind

#### Reducing Trust Transation Costs

As mentioned above, semantic interoperability is just a means to an end, automated reasoning. But automated reasoning is also just a means to another end. The ultimate end in my opinion, at least with respect to verifiable credentials (VCs), may be expressed succinctly as lowering trust transaction costs. 

Significant trust transaction cost reduction comes from automating the process of establishing and conveying sufficient trust to enable better or entirely new interactions (transactions). When one starts at this end and works backwards while also recognizing the inescapable pervasiveness of uncertainty (in all forms) and how to deal with that uncertainty then one realizes how limited the role of semantic interoperability becomes. Automated reasoning systems in complex uncertain environments (aka the real world) are layered and hierarchical and must isolate and properly manage domains of uncertainty not mash them together by stripping off that uncertainty. Layering and partitioning reduces the drag of inter-contextual uncertainty (ambiguity, polysemy). Therefore one does not need or want a flat sea of knowledge using one barren over-simplified representation (like `is-a` or `has-a` triples), but instead one wants a layered hierarchy of reasoning with knowledge under uncertainty.  One employs a mixture of different reasoning methods and different knowledge representations, each best suited to the data and application not the other way around. This constrains and simplifies knowledge domains and contexts and makes them manageable. This is how the human brain works; it allows us to reason effectively in an environment of extreme complexity and uncertainty. The design of an automated reasoning systems starts with end decision in mind and works backwards to the source data. An automated reasoning system may have many layers including learning, adaptive, supervisory within each decision making process or construct. The data representation is driven by the decision making process not the other way around.

#### Dealing with Uncertainty

Let's take a simple example:  A given eco-system of transactions involves decisions that enable those transactions, such as should one open a bank account for a particular person? If we oversimplify our knowledge representation too early in the process by removing uncertainty then we will build a fragile decision-making process. One reason that practical real world automated decision making systems (aka automated reasoning) are layered is because anti-fragility comes from supervisory, adaptive, and learning safety jackets.  

Suppose one of the requirements for opening a bank account is proof of identity such as proof of the assertion that Joe is a US Citizen. Suppose that a passport may serve as proof of Joe's citizenship. In an automated computer system, however, there is no way to establish with ***complete*** certainty the Truth of the statement `Joe is a US citizen`. It may only ever be established as `TRUE` to some degree. 

Now suppose Joe presents his passport by sending a scan of it. An image processor scans the image of the passport and based on a set of features returns a likelihood measure that the passport is valid (statistically valid). The passport includes a photo. But maybe the passport was stolen or someone made a scanned copy so we decide not to rely on the passport alone. So we require another form of photo ID such as a drivers license and employ another process to compare photos and do a match. You might believe that this example is a motivating example for using a verifiable credential based on the erroneous assumption that A VC that states `Joe is a US Citizen` cryptographically verifies the truth of that statement. It's not. A VC does nothing to establish the veracity (accuracy) of any statements it makes; it merely establishes the authenticity (who made it). The veracity (accuracy) of a VC comes from real world measurement and decision processes used by its issuer prior to issuing the VC. Real world processes are always uncertain to some degree. Hence the veracity of the VC is always uncertain to some degree. We must trust the issuer but the degree of uncertainty in either the original decision process or our trust in the issuer is not provided in the VC. So any subsequent decision process that uses that VC has lost the inherent uncertainty that qualifies the statement, `Joe is a US Citizen` and has replaced it with an uncertain reliance on trust in the uncertain reputation of the issuer of that statement. 

#### Compounding Uncertainty

***The idea of semantic interoperability is built on the poorly founded idea that one can combine statements whose veracity is purely established by reputation as if they are TRUE statements.*** Once one accepts this poorly founded idea, one is tempted to believe that given any statement so established as `TRUE` then all one needs for semantic interoperability is a universal schema and representation that allows one to treat as identical, other `TRUE` statements from different issuers about the same subjects and then derive other `TRUE` statements about those subjects that one then erroneously concludes must be every bit as `TRUE` as the original source statements. Anyone doing so has removed the inherent uncertainty from the inference process at their peril and because it provides a false sense of accomplishment there are misled into believing that if they only had a universal schema then they could reason semantically in an unbounded way about all such `TRUE` statements. 

Instead one must recognize, that one's ability to infer is always limited by the uncertainty of the underlying decision processes. No statement is ever completely `TRUE` and when one chains together a set of `almost TRUE` statements via an inference process, the `TRUTH` of that chain of inference decreases dramatically with the length of the chain. 

#### Probabalistic Uncertainty

Consider the case of probabilistic uncertainty. Suppose Joe's presented passport was validated by the VC issuer's image processing algorithm with a probability of 0.95.  Now suppose that Joe's drivers license was validated by the image processing algorithm with a probability of 0.9 and that the process for matching photos was evaluated with a match probability of 0.8.  Let's say `Joe is a US citizen` is `TRUE` `IF` Joe presents a valid passport `AND` Joe presents a valid drivers license `AND` the photos of the two match. This is a conjunction whose truth (probability conjunction) is usually calcuated as `0.95 * 0.9 * 0.8 = 0.684`. Clearly much less `TRUE`. Now one may quibble and note that the combination of those three items is not a simple `AND` conjunction but must be normalized by the conditional probability of both a drivers license and a passport with the same name and matching photo (making the probability higher than 0.684). This is beside the point. Once we have removed the uncertainty in our model, whatever it may be, we have no way of knowing the actual `TRUTH` value of our chain of reasoning. ***So we would be foolish to attempt any long chains of inference on a big flat knowledge graph. Thus there is little practical benefit of a big flat knowledge graph.***

To elaborate, when a VC asserts that `Joe is a US citizen`. The veracity of that assertion is not verifiably `TRUE`. At best it may be more-or-less `TRUE`. But a consumer of that VC does not know that. And any downstream decision process may be made more fragile by that lack of knowledge. Recall, that any real-world data has uncertainty to some degree and may have multiple forms of uncertainty to varying degrees. ***As a result, any inference chains on that real-world data must be bounded in length because the inferred `TRUTH` decreases with the length of the inference chain***. ***This means that any corpus of knowledge that excludes uncertainty in its representation and thereby allows inference chains of unbounded length is a completely unrealistic approach to modeling and reasoning about the real world. It's a theoretical fantasy and a practical nightmare.*** ***This aptly describes Linked Data (RDF)***. Given that unbounded inference chains are so problematic a real-world environment, then semantic interoperability for semantic inference in real-world applications only makes sense within limited domains and contexts that are bounded by the length of practical inference chains as constrained by the varying degrees and types of uncertainty of the associated data. 

#### Proper Knowledge Representation

This means that knowledge is best represented as a hierarchy of combinations of knowledge from bounded domains or contexts. The associated decision processes look more like decision trees with semantic interoperability (and common schema) at each layer of each branch of the decision tree. Information from lower layers on different branches may each use different semantics and schema. This means using sets of semantics each within a given application domains. Interoperability is limited to the size of each set (application domain), not the set of all sets. This greatly simplifies the task of interoperability because it relaxes dependencies between application domains. 

For example a given financial industry's KYC semantics (and schema) may be different from a given supply chain's semantics (and schema) thus simplifying the design of each. A given decision process that needs to combine KYC and Supply chain knowledge to reason about both may use a simplified aggregation semantic and (schema). This follows best practices for complexity management in systems design. Suppose the supply chain has a requirement that only KYCed entities may be extended credit for purchases. The supply chain semantic and schema only needs a way of expressing `ACME has been approved for credit` but do not have to include intermingled semantics and schema for every minutia of the KYC decision process.

Universal semantic interoperability on a flat knowledge graph is based on the faulty premise that we can make `TRUTHful` decisions on complex inference chains that aggregate information from multiple VCs from different sources from wildly different contexts where those VCs may be combined without including the underlying uncertainty. To reiterate, that premise is dangerously `FALSE` and the practical value of such unbounded semantic interoperability without uncertainty is extremely limited. Therefor It's a futile effort to even contemplate much less strive for universal semantic interoperability and extensibility. We don't need it, because we can't use it.

### Systems Design and Complexity Management

#### Complexity Management

One of my fields of research is called Complexity Management. The lab I used to run was called the Advanced Marine Systems Lab, with an emphasis on *Systems*. In my research, the key to complex system design starts with managing complexity and the most powerful tool we have for managing complexity is to isolate dependencies. This is usually and almost exclusively performed via hierarchical composition/decomposition of the system. If I can collapse layers in my system hierarchy then they were bad layers in the first place. The point of layers is to reduce the number of dependencies that one must manage. 

The terminology I use is `real` versus `apparent` complexity. The `real` complexity is a measure of the number of dependencies in the system. The `apparent` complexity is a measure of the number of dependencies one must manage to make meaningful changes to and/or to meaningfully use the system. When one collapses layers there is typically a combinatorial explosion in the number of dependencies that one must manage. This increases apparent complexity not reduces it. 

An optimal design from a complexity management perspective minimizes the ratio of `apparent` complexity to `real` complexity. In its simplest form, good layering acts to reduce the combinatorics of dependencies without reducing the expressive or functional power of the system. It is trivial to show how this may happen; a tree of dependencies scales better than a mesh for combinatorial reasons.  

#### The False Allure of an RDF Graph "Soup"

One reason RDF was ideologically so attractive but practically so difficult to use is that its triples with `is-a`, `has-a` predicates appear simple. But when combined for anything but simple applications, the resultant combinatorial explosion of triples makes the resulting graph practically infeasible to understand, reason with, manage or otherwise use. RDF's ideal is that there are no layers, but one big semantic soup of triples with the weakest form of inference possible (`is-a`, `has-a`). Thus RDF's idealistic simplicity of expression actually prevents dependency management layering in any meaningful way. Its ideal ends up maximizing the ratio of `apparent complexity` to `real complexity` not minimizing it. So it's the wrong kind of simplicity. It's **too** simple.  As Einstein is reputed to have said: "Everything should be made as **simple** as possible, **but no simpler**." Better yet is Oliver Wendell Holmes  "I would **not** give a **fig** for the **simplicity** ***this*** **side of complexity**, but I would give my **life** for the **simplicity** on the ***other*** **side of complexity**." 

Linked Data (RDF/JSON-LD)  ***is simplicity on the wrong side of complexity***.

#### The Superiority of Labeled Property Graphs

In contrast, a labeled property graph (LPG) allows dependency isolation via layering. Edges may have different complex types with multiple properties which allows layering of semantic operations at the edges. Likewise, nodes may have different complex types with multiple properties.  This allows layering using combinations of different node and edge types. When one looks under the hood at a practical implementation of an LPG, one sees that the graph is usually structured hierarchically and thereby better following the principle of minimizing apparent complexity.  So although it is one graph syntactically, it is a hierarchy of layered graphs semantically. And the different combinations of edge and node types create different semantic classes for each of the layers.  We can have interoperable syntax (i.e. interoperability at the semantics of the syntax of the LPG) without being forced to have collapsed layers.  Likewise JSON Schema allows layered schema and isolated contextual ontologies of schema. In contrast [schema.org](http://schema.org/) org does not because [schema.org](http://schema.org/) forces the collapse of schematic layering into one universal ontology. 

One of the most common criticisms of RDF is that: *it requires an A.I. to use it, but there is no A.I. yet powerful enough to use it*. Humans are incapable of using it but for very simple applications. So if it's good only for very simple applications why use it all? Especially if there are more powerful, easier to use, and already far more broadly adopted alternatives like LPGs?


 
### Proposal: Adopt a JSON Schema VC Specification as the Primary Path to Adoption 

What if a small group of the decentralized identity community created a JSON schema VC standard
with ***immutable*** content-addressable schema definitions? How fast would we get adoption outside the
narrow confines of the existing community?

Questions:
How easy would it be to make secure?
How easy would it be to express VC equivalents?
How easy would it be to create a tooling stack to support it? 
How easy would it be to understand and explain?
How easy would it be to gain adoption of such a standard? 

Answer:
Everything would be easier! Much easier than JSON-LD.

So why are we using a JSON-LD based standard if we care about adoption? 

Do we really need to support the edge cases of JSON-LD versus JSON schema as first class properties instead of the special edge cases that they are?

Did we learn nothing from the failure of XML vs. JSON, XHTML vs. HTML5 or the failure of RDF versus any other Graph database? Most if not all have higher core application work flow adoption than RDF graphs. (Google SEO is a proprietary narrow edge case)



For content addressable immutable schema, think of the compact composable content addressable derivation codes from KERI (KID001)[https://github.com/decentralized-identity/keri/blob/master/kids/kid0001.md]
(KID001 Comments)[https://github.com/decentralized-identity/keri/blob/master/kids/kid0001Comment.md]
but applied to JSON schema.

Related are decentralized resource identifiers (DRI)[https://engagedri.ca/assets/documents/whitepapers/The-opportunities-of-Decentralized-Resource-Identifiers-in-the-research-landscape.pdf] or
(ISCC Content Identifiers)[https://iscc.codes]


### Tactics: A Carrot and a Stick

#### The Stick is Security. 

Correct layering for security means that authenticity (secure attribution)
happens before considering the payload. Authenticity of the over-the-wire serialization of any payload is established before opening the payload.
The end state of JSON-LD RDF is antithetical to this concept. It is disappointing that anyone is seriously considering using JSONLDSignatures for security on the serialized expanded RDF. It violates fundamental principles of secure protocol layering. Alternatively signing the over the wire JSON-LD before it has been expanded where that JSON-LD includes a reference to mutable schema.org based contexts is not secure because of the mutability and vulnerability of schema.org. One would need to fix up that context to make it immutable which can be done much easier with  content addressable JSON Schema which gains more adoptability than fixing JSON-LD. Caching mutable schema is a hazardous fragile fix. Some VCs must survive for decades which means the caches must survive for decades as well.

#### The Carrot is LPG (Labeled Property Graphs) and JSON Schema

Labeled property graphs (LPG) are better then RDF for semantic inference in almost every waywould be a huge advantage to any standard that wants to convert the payload to a graph for inference. One can create LPGs using several open standard graph databases. [Cypher](https://en.wikipedia.org/wiki/Cypher_query_language)  is an open standard query language for LPGs and [GraphQl](https://graphql.org/learn/) is an open standard wrapper library for queries that enable a standard query interface for custom LPG APIs. Existing SQL and NoSQL databases can be designed to support the semantics of LPGs further reducing adoption barriers. In all cases one would use immutable JSON schema for over-the-wire transmission. JSON Schema is already universally adopted for over-the-wire transmission when communicating with ReST APIs.  Adding immutability to JSON Schema is a minor extra effort.

## End State for JSON-LD/RDF Security is Not Scalable nor Widely Adoptable

When I dug down into the Linked Data Proofs specification I unfortunately found confirmation of what I had been told about the  end state for securing JSON-LD/RDF documents.

[https://w3c-ccg.github.io/ld-proofs/#dfn-canonicalization-algorithm](https://w3c-ccg.github.io/ld-proofs/#dfn-canonicalization-algorithm) points to

[https://w3c-ccg.github.io/security-vocab/#URDNA2015](https://w3c-ccg.github.io/security-vocab/#URDNA2015) which provides proof types.

 
The proof types are signatures on fully expanded reserialized RDF graphs not on over-the-wire JSON-LD documents.  A proper JSON-LD proof according to the spec requires that the signed JSON-LD be derived from a fully expanded, canonicalization of the over-the-wire json-ld. The canonicalized fully expanded JSON-LD is derived from a fully normalized RDF graph that is used to produce a fully expanded JSON-LD serialization (not the original over the wire JSON-LD).

[https://www.w3.org/TR/json-ld11-api/#serialize-rdf-as-json-ld-algorithm](https://www.w3.org/TR/json-ld11-api/#serialize-rdf-as-json-ld-algorithm)

  
Quoting, "The default canonicalization mechanism is specified in the RDF Dataset Normalization specification, which deterministically names all unnamed nodes. "

One way to serialize an RDF graph is to convert it to a normalized fully expanded JSON-LD serialization.

This is not the over the wire JSON-LD but the JSON-LD that has been fully expanded. So a signature on the over the wire JSON-LD (AKA the verifiable credential itself) is not verifiable on the RDF serialization JSON-LD and vice-versa.

  
Indeed The following are all signatures on serialized RDF graphs.: ([https://w3c-ccg.github.io/security-vocab/#URDNA2015](https://w3c-ccg.github.io/security-vocab/#URDNA2015)) 

  
- GraphSignature2012
- LinkedDataSignature2015/LinkedDataSignature2016
- MerkleProof2019
- Ed25519Signature2018
- JsonWebSignature2020/JsonWebKey2020
- BbsBlsSignature2020/BbsBlsSignatureProof2020

  
I always thought that JsonWebSigantures were signing the over-the-wire  JSON-LD document itself (AKA the Verifiable Credential) not the RDF serialization (AKA a fully expanded JSON-LD derived from the RDF which in turn was derived from the VC)

Moreover the expansion process is a performance hog. I talked to a couple of developers who are using the expansion library and they volunteered that it's extremely non-performant. This alone makes it non-scalable and an adoption show stopper when it must occur before signature verification.

There does appear to be one special case exception that allows signatures on the over-the-wire JSON-LD.
In other words, JWS signatures on the VC itself are a special case i.e. "the detached and unencoded payload variant"  (see below). But even those are not secure due to the mutability of the @context in the over-the-wire JSON-LD.
Immutability is not guaranteed by JSON-LD until after full normalization, expansion and re-serialization, then and only then is there any guarantee of immutability according to the standard.
Anything else is a special case exception that is a variance with the end state security design of the JSON-LD Signature specifications.

Quoting:


> The JWS used with Linked Data Signatures is the detached and unencoded payload variant. See the links below for details:  
> [JSON Web Signature (JWS) Unencoded Payload Option](https://tools.ietf.org/html/rfc7797)  
> [JSON Web Signature (JWS) and JWS Detached for a five-year-old.](https://medium.com/swlh/json-web-signature-jws-and-jws-detached-for-a-five-year-old-88729b7b1a68)  




Certainly we could define a new standard for VCs that required immutable schema (i.e. not schema.org) but that is such a fundamental misalignment with the JSON-LD community that it serves no purpose. We might as well use the already much more widely adopted JSON schema instead and impose an immutability constraint on our use of JSON schema.

***My contention is that we must have security on the over-the-wire data not on some eventually expanded canonicalization. Otherwise we limit adoption dramatically because we impose an insurmountable interoperability barrier by requiring RDF canonicalization/expansion as a hard dependency on any security (signature verification).***

## Immutability Requirements on JSON Schema

All schema definitions must be securely immutable, full stop. Identifying schema with versioned identifiers does not guarantee immutability. Secure immutability comes from either embedding the schema definition in the VC or embedding a digest of the schema definition in the VC with the schema definition stored elseqhere. In either case the signature on the VC will not verify if the schema is changed. A digest is a verifiable cryptographic commitment to the schema stored elsewhere. When embedding a content digest of the schema definition in the VC, the schema definition itself must be stored elsewhere in a highly available repository or registry or database. Any change to schema results in a new content addressable instance of that schema stored in that repository. To clarify, the signature on the VC makes a verifiable commitment to the digest which in turn makes a verifiable commitment to the immutable schema. The VC will not be verifiable if the actual schema definition is not found in the repository which is why it must be highly available. But availability is a much easier problem to solve than security.

Immutable schema definitions require a content-addressable identifier name space. One will need to be designed for JSON schema so that they be known as immutable. Issuers are responsible for designating a highly available repository for the immutable schema they reference in VCs. Alternatively the issuer may embed a copy of the schema in the VC itself, thereby making the VC self contained with respect to the schema and removing the requirement for a highly available schema repository.

This level of flexibility will make adoption much easier as high risk high performance applications that may avoid the performance hit of an external repository with embedded schema definitions (they may still include the identifier allowing semantic interoperability) or attach the schema definition to the VC when transmitted or cache the schema. Because the schema is immutable all three approaches to performance optimization and risk mitigation are secure.

## Self-Addressing Identifiers (SAIDs)

One of the innovative concepts underlying the KERI are self-certifying identifiers (SCIDs). These are identifiers that are cryptographically bound to a set of (public, private) signing key pairs. A type of SCID is a self addressing identifier (SAID) that is bound to more than just the key pairs, it is also bound to other configuration data that supports the identifier. KERI uses SAIDs to bind the identifier to the inception event of the Key Event Log that provides proof of the identifier key state. For schema and VCs we may use non-SCID SAIDs or simply SAIDS. An SAID is self-referential and is embedded in the data it addresses. Generally one may not embed a content addressable identifier in the content that it is derived from unless one uses a special derivation protocol. KERI employs such a protocol for its SAIDs. We can use this protocol to generate SAIDs both for a VC and its schema. Schemas or VCs with embedded SAIDs are provably immutable in a self-contained way. The SAID with its derivation protocol makes any content tamper-proof i.e. provides proof of immutability. To clarify, any valid (integral) copy of the VC or schema will validate against its embedded unique SAID. Therefore a cryptographic commitment to any SAID is equivalent from a security perspective to a cryptographic committment to the actual contents of the data that the SAID references (was derived from). We can therefore reason about VCs and their schema through their SAIDs. ***This simplifies the interoperable security of schemas and VCs without needing schema or VC identifier registries.  This fully decentralizes VCs and their Schema.
***

###  Self-Contained Secure Schema

Unlike schema.org and JSON-LD, JSON Schema provides convenient support for completely self contained custom schema and embedded custom schema via schema metadata fields with emphasis on the word convenient. Any field in a JSON schema that is not included in the properties field is metadata. This allows a given schema definition to be self-describing. More importantly this allows a schema definition to include meta data that may be cryptographically committed to via a cryptographic digest of the whole schema including its meta-data fields. When the schema identifier used externally is a digest of all or part of the schema then that identifier provides a means to verify the integrity of any copy of the schema thereby ensuring immutability.  


A version field in the meta data allows an immutable commitment to a given semantic version but the actual version is the digest. A different digest means a different version.

For example

```JSON

{
  "$id": "did-of-schema-itself",
  "$schema": "did-of-schema-repository",
  "title": "Human Friendly title of schema",
  "version": "1.0.0",
  "type": "object",
  "properties": 
  {
  }
}

```


### Self Addressing Identifier (SAID) Derivation Protocol

The advantage of a content addressable identifier is that it is cryptographically bound to the contents providing a secure root-of-trust. Any cryptographic commitment to a content addressable identifier is functionally equivalent (given comparable cryptographic strength) to a cryptographic commitment to the content itself. 

A `self-addressing identifier (SAID)` is a special class of content-addressable identifier that is also self-referential. The special class is distinguished by a special derivation method or process to generate the `SAID`. This derivation method is determined  by the combination of both a derivation code prefix included in the identifier and the context in which the identifier appears. The reason for a special derivation method is that a naive cryptographic content addressable identifier must not be self-referential, i.e. the identifier must not appear within the content that it is identifying. This is because the naive cryptographic derivation process of a content addressable identifier is a cryptographic digest of the serialized content. Changing one bit of the serialization content will result in a different digest.  Therefore self-referential content addressable identifiers require a special derivation protocol, method, or process. 

The `SAID` derivation protocol assumes that the fields in a JSON serialization are ordered in stable round trippable reproducible order, i.e. canonical. The natural canonical ordering for JSON is called property creation order or field insertion order. This logical ordering has been supported by the `stringify()` method in JavaScript with a custom reordering function using Reflect.ownPropertyKeys() since ES6 and natively without a reordering function by `stringify()` since ES11 (2020). Property creation order is a natural canonicalization as it supports any predefined logical ordering that one chooses by creating or inserting the properties (fields) in that predefined order. Other languages have long supported ordered dictionaries or mappings that support insertion order of the fields when serializing the mapping to/from JSON. ***The canonical serialization problem for JSON is now a solved problem.***  

The self-addressing identifier derivation process is as follows:

- replace the value of the `id` field in the content that will hold the self-addressing identifier with a dummy string of the same length as the eventually derived self-addressing identifier
- compute the digest of the content with the dummy value for the `id` field
- prepend the derivation code to the digest and encode appropriately to create the final derived self-addressing identifier of the same total length including the prepended code as the dummy.
- replace the dummy with the self-addressing identifier

As long as any verifier recognizes the derivation code, the `SAID` is a cryptographically secure commitment to the contents in which it is embedded. It is a cryptographically verifiable self-referential content addressable identifier.

Because an `SAID` is both self-referential and cryptographically bound to the contents it identifies, anyone can validate this binding if they follow the _derivation protocol_ outlined above.


To elaborate, this approach of deriving self-referential identifiers from the contents they identify,  is called `self-addressing`. It allows any validator to verify or re-derive the self-referential self-addressing identifier given the contents it identifies.  To clarify, a `SAID` is different from a standard content address or content addressable identifier in that a standard content addressable identifier may not be included inside the contents it addresses. Moreover a standard content addressable identifier is computed on the finished immutable contents and therefore is not self-referential.  


### Self-Addressing VC and Schema Identifiers (SAID for $id)

VCs and their Schema typically include within their contents respectively, a self-referential identifier that is not derived from their contents. This poses a problem because there now may be two different identifiers for the same content. The first identifier is self-referential but not cryptographically bound although included in the content and the second identifier is cryptographically bound but not included in the content. When reasoning about the content, the existence of a non-cryptographically bound identifier is a security vulnerability. Certainly the non-cryptographically bound but self-referential identifier may not be used on its own to securely reason about the content because it is not bound to the content. Anyone can place such an identifier inside a different schema or VC and claim that the different schema or VC is the correct one for that identifier.  On the other hand, a standard content addressable identifier may not be included in the content itself. It must be tracked independently so it is not self-contained which may pose other security problems. Whereas a SAID is included in the content, is self-referential, and is cryptographically bound to that content using the prescribed standard derivation method. SAIDs make VCs and their schema fully self-contained with self-referential cryptographically bound and verifiable content addressable identifiers.

To elaborate, we may apply the SAID derivation protocol defined above to generate the `$id` field in a JSON Schema as well as the `id` field in a VC. Furthermore, nested blocks in both schema and VC content may have block specific SAIDs. Specifically any given schema identifier or nested schema block identifier may be derived from all the contents of the schema or nested block except for the `$id` field itself. Likewise any given VC identifier or nested VC block identifier may be derived from all the contents of the VC or nested block except for the `id` field itself. 

To clarify we restate the derivation process as applied to the schema `$id` value:   

- First replace the value of the $id field with a string filled with dummy characters of the same length as the eventual derived value for $id. 
- Second make a digest of the serialized schema contents that include the dummy value for the $id. 
- Third replace the dummy identifier value with the the derived identifier value in the schema contents. 

Thus the `$id` of the schema or the `id` of the VC will be strongly bound to the contents of everything except for the identifier itself in a universally verifiable way. This derivation approach allows a self-referential identifier embedded in any content to also be cryptographically bound to that content.

Thus only one identifier need ever be used to securely reason about any content, that is, the SAID. This also makes the content immutable with respect to its SAID. Any change in the content and the SAID will no longer be derivable from that content. This provides a secure root-of-trust for reasoning about any content in a self-contained way, not merely VCs and their schema but especially VCs and their schema.
Moreover because a cryptographic commitment to a SAID (such as digital signature) is equivalent (from a cryptographic security perspective) to a commitment to the content that the SAID is derived from, we may create very compact VCs using only SAIDs for the contents of the VCs and and any nested blocks as well as the schema and any nested schema blocks. The actual content referenced by the SAIDs my be provided elsewhere such as with a cache or database. This allows the over the wire version of a VC to be extremely compact while not losing any security. The actual content must still be provided in a highly available way, but any copy of the content will do because the SAID is universally verifiable with respect to its content. This converts a security problem (mutability) into an availability problem. Availability problems are easily solved with redundant highly available data stores (i.e. NOT total ordering shared distributed consensus ledgers). This allows a given application or eco-system to tune their VCs for performance. Yet another advantage over the poorly though out inverted security model of JSON-LD/RDF VCs.



## Natural Canonicalization

As mentioned above, the SAID derivation protocol requires that JSON have a canonical field ordering. One of the touted reasons to use JSON-LD/RDF has been that it provides a canonical serialization for key:value mappings (hash tables, dictionaries).  By canonical we mean reproducible or stable. A key:value mapping is a self-describing data structure in that each value is labeled by a key. One can infer the purpose of the associated value from a descriptive or well known key (label). Historically these key:value mappings used something called a hash table. A simple hash table does not provide a stable ordering of its keys when serialized. This means that when serializing and deserializing one may not have a stable serialization for a verifiable digital signature. 

For example a JSON document is a serialization of a JavaScript Object. Because under the hood Javascript objects are based on a hash table structure which does not guarantee a stable order of appearance, a signed serialization may not guarantee its later verifiability. One reason put forth for using JSON-LD with an expanded RDF is that expanded RDF has a canonical form that when re-serialized is stable. In other words JSON-LD/RDF provides a standard canonical serialization whereas JSON does not.

This may have been true years ago when JSON-LD was first conceived but it has not been true with respect to ECMA standard JSON serialization for some time.
The EcmaScript standard Javascript (since ES6) supports stable property creation order serialization and deserialization of JSON via the stringify method and Reflect.ownPropertyKeys(). More importantly, as of ES11 (2020) all JavaScript objects preserve property creation order in stringify(). In other words as of ES11 JSON objects are canonically ordered by default by property creation order. This means serialization and deserialization natively preserves this canonical ordering. This means that a trivial natural canonicalization standard is to use property creation order.

The JSON-LD/RDF ordering is alphabetic. This is a much more cumbersome ordering. It does not allow one to structure documents with the most logical understandable ordering of fields. Property creation order makes any logical ordering possible merely by creating the properties in that logical order when creating the document.

This now puts JSON on equal footing with the predominant languages that have for some time already adopted property creation ordering for mappings (hash tables). For example all dictionaries in Python since version 3.6 are ordered by property creation ordering. (A python dictionary is the Python equivalent of a JavaScript object for serialization purposes). Python's standard library prior to 3.6 also supported an ordereddict type. Languages such as C or Rust that use fixed data structures instead of hash tables have always supported ordered serialization.

To restate, for some time (ES6 or later) it has been trivial for a JavaScript Object ser/deser to reproduce and preserve property creation order.  This makes property creation (logical) ordering the natural canonical serialization ordering. Although Javascript was a late comer to the ordered map party there is no longer a need to use JSON-LD/RDF's much more cumbersome approach to canonicalization.


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

We may impose ordered serialization with the JSON.stringify method by using the replacer option and the \[\[ownPropertyKeys\]\] internal method added in Javascript ES6 exposed as Reflect.ownKeys(). This method is provided in JavaScript ES6 (2015). \[\[ownPropertyKeys\]\] uses the property creation order. At the time of ES6, JSON.stringify was not required to use the \[\[ownPropertyKeys\]\] iteration order (ie property creation order). This appears to have been fixed in later versions of ES. (See the discussion below). As of this writing nodejs v14.2.0 and the latest versions of Chrome, Safari, and Firefox all appear to preserve property creation order in JSON.stringify() without having to use  Reflect.ownKeys(). This means that we can depend on and use that ordering in the specification. For versions that support at least ES6, a custom replacer function for JSON.stringify(x, replacer) will ensure that property creation order is used. Earlier versions that do not support ES6 may still work with an ES6 polyfill if that includes support for \[\[ownPropertyKeys\]\].

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
