---
tags: TSP, KERI 
email: sam@samuelsmith.org
version: 1.00
---

# Privacy Given Strongest Authenticy and Confidentiality (part 1)

[![hackmd-github-sync-badge](https://hackmd.io/2D6Ld2vISf2M8UE0XqbGng/badge)](https://hackmd.io/2D6Ld2vISf2M8UE0XqbGng)


## PAC Trilemma

In my opinion, the three properties, authenticity, confidentiality, and privacy inhabit a trade-space. I have posited the PAC Theorem or PAC Trilemma to qualify this trade-space. The PAC theorem/trilemma states:

> One can have any two of the three (privacy, authenticity, confidentialty) at the highest level but not all three. 

The trilemma insists that one must make a trade-off by prioritizing one or two properties over a third.

The ToIP [design goals](https://github.com/trustoverip/TechArch/blob/main/spec.md#61-design-goals) reflect that trade-off and provide an order of importance. The design goals indicate that one should start with high authenticity, then high confidentiality and then as high as possible privacy given there is no trade off with respect to the other two. The properties are defined as follows:  


> With regard to the first design goal, establishing trust between parties requires that each party develop confidence in the following properties of their relationship:
> 
> 1. Authenticity: is the receiver of a communication able to verify that it originated from the sender and has not been tampered with?
> 2. Confidentiality: is the contents of a communication protected so only authorized parties have access?
> 3. Privacy: will the expectations of each party with respect to usage of shared information be honored by the other parties?
> 
> Note that, in some trust relationships, confidentiality and privacy may be optional. Thus our design goal with the ToIP stack is to achieve these three properties in the order listed.
> 

The prioritization of authenticity first, confidentiality second, privacy third does not mean that when constructing a message that the order of layering of mechanisms in the message that contribute to authenticity, confidentiality, and privacy protection must or should follow that same order. 

Indeed, it may be best in some protocols if they don't share the same ordering. To be clear, the ordering of the ToIP design goals w.r.t. authenticity, confidentiality, and privacy may be different from the ordering of layers that employ these three in a given protocol. 

This, distinction, I believe is a source of much confusion in prior discussions and one of the reasons I provided the overlay spiral diagrams. The purpose of the overlay spiral is to illustrate that multiple layerings of  authenticity, confidentiality, and privacy protection mechanisms may appear  in different orders both in-band (IB) and out-of-band (OOB) with respect to the setup for and the eventual transmission of a message over a given communication channel or channels.  

From a practical perspective I use the following elaborations to help refine how I think about the three properties.

* Authentic and Authenticity:  
The origin and content of any statement by a party to a conversation is provable to any other party. Authenticity is about control over the key state needed to prove who said what in the conversation via digital signatures. In other words is about secure attribution.
* Confidential and Confidentiality: 
All statements in a conversation are only known by the parties to that conversation.
Confidentiality is about control over the disclosure  of what (content data) was said in the conversation and to whom it was said (partitioning). Confidentiality is about control over the key state needed to hide content via encryption vis-a-vis the intended parties.
* Private  and Privacy:  
The parties to a conversation are only known by the parties to that conversation.
Privacy is about control over the disclosure of who participated in the conversation (non-content meta-data = identifiers). Privacy is about managing exploitably correlatable identifiers



## Privacy and Confidentiality

There are also two fundamentally different but complementary definitions of privacy that are relevant to protocol design. 

The ToIP definition is about the recipient respecting the data privacy concerns of the sender with respect to the content disclosed to the receiver. The protection comes from the recipient protecting the data privacy rights of the sender once the recipient has received that data. 

The second defintion of privacy is about how correlatable is the publically viewable metadata that is included in the communication. This is called non-content meta-data by the survelliance community. This defintion makes a clear distinction between confidential data (meta-data or otherwise) that is not publically viewable (where public means any third party that is not a parties to the conversation. Typically confidentiality protection comes from encryption (i.e. cipher text vs plain text). But an out-of-band (OOB) exchange of information is confidential (not viewable) by a third party surveillor of an in-band (IB) channel thus making the OOB exchange confidential without encryption.

This first type of privacy protection may be achieved independently of how private the communications channel was that resulted in the receiver obtaining a disclosure of content. 

The terms confidential and private, also have distinct legal defintions. The legal definition of confidentality, however, is consistent with the surveillance definition above as well as the defintion used by the ToIP properties above. Confidentiality law is about contractual agreements governing the disclosure of data by one party to another. The legal definition of privacy, on the other hand is not so consistent. The surveillance regulatory law definition is different from the privacy rights regulatory law definition. The former is about drawing the line between non-content metadata and content data in order to be classified as searchable or not [content-metadata-line](https://www.lawfareblog.com/relative-vs-absolute-approaches-contentmetadata-line
) [surveillance](https://www.pogo.org/analysis/2019/06/the-history-and-future-of-mass-metadata-surveillance/) and the latter is mainly about protecting personal data rights by data holders, controllers, and processors (see [GDPR](https://gdpr-info.eu)).  

Of interest is the fact that the former (confidentiality law) can be used to better protect the latter (personal data rights). For example, the well-known concept called [chain-link confidentiality](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=2045818) , circa 2012, details how one can protect the data privacy concerns of the a sender via contractual confidentiality law. ACDCs, for example, use the well-known [Riccardian Contract](https://iang.org/papers/ricardian_contract.html) formalism, circa 1996, to support chain-link-confidential through what is called contractually protected disclosure.

To summarize:
* Physical Search:  (analogy to Constitutionally protected physical search)
Outside of home vs. Inside of home.  
The address is a public identifier used for routing and taxing. 
Viewing who enters the home from outside via public street is unprotected. (subpoena)
Viewing what happens behind closed doors inside the home is protected. (warrant)

* Information Search:
Relatively weak legal protection (subpoena) for identifiers, (non-content meta-data)
because identifiers are needed for public routing and billing.
Relatively strong legal protection (warrant) for content because the content is not needed for public routing and billing.


But for the purposes of protocol design the definitions of confidentiality and privacy that are most suitable are the surveillance defintions. To clarify, privacy is about the leakage of exploitably correlatable metadata about the parties to the conversation and confidentiality is about restricting the viewability of the content of the conversation to the intended parties to the conversation. In simple terms privacy is about protecting the knowledge of who is a party to a given conversation and confidentiality is about protecting  the knowlege of what was said in that conversation.

The surveillance distinction between privacy and confidentiality has a very practical basis. Recall that for the purpose of protocol design we defined privacy as about protecting knowledge of about the parties to a conversation (exchange of data).  A primary use case for identifiers in communication prototocols is to address the parties to the conversation. Indeed the practical need for identifiers to support addressing often makes the privacy protection of knowlege of identifiers as metadata to conversations particularly (in many cases pathologically) difficult.


Historically, an address of any kind (including the original postal address) exists for two main purposes. 

1. So that stuff can get routed (delivered) to that address as a destination via public roads. 
2. So that legitimate parties can tax or bill efficiently other parties at any or all addresses.

In the electronic communication's world the same applies. Addresses need to be public so that communications can be routed to destinations over multiple disparately owned, controlled, and regulated communications infra-structures (largely public regulated monopolies). Likewise, the main mechanism for billing for communication services rendered is address based. So addresses have to be public, not confidential, not only so that stuff can get routed to those addresses but also so they can be the destination for bills to pay for the communication itself. So pragmactically, the legal surveillance world respects that these two purposes (routing and billing)requires identifiers (addresses) that must needs be public and eavesdroppable.


This is merely a logical practical extension of property rights w.r.t. unreasonable search. The right to be protected from search only extends to what is not veiwable behind closed doors. Anything viewable from a public street is fair game with no protection. Likewise to some extent an automobile or delivery van. What's locked in the trunk or the back of the van (unless its making noise hearable by a bystander) may be protected. Where it goes on a public street is not protected, and its license plate is a publically viewable address proxy.  

As a result, both practically and legally, all routing and billing mechanisms assume to a large degree that addresses are not private. Anyone trying to build a house and then telling the post office, that it can't assign them a public address, or expecting that anyone will build infrastructure to support getting stuff delivered but not using public streets will likely fail. Imagine the cost of supplanting public streets with a totally private conveyance system (air-space, sea-space, and space-space are public already). If one wanted to build a totally private alternative to the public road network then maybe automobiles and vans on private roads wouldn't need license plates (oh wait these are called toll roads and they need a billing address) so not.

So the analogy holds. We can have strong authenticity and strong confidentiality (of anything but addresses) but weaker privacy w.r.t addresses (identifiers). Beyond a certain point, better privacy over addresses becomes impractical because the world needs addresses for routing and billing on public infrastructure and will always need them. So privacy, as non-correlatable identifiers (metadata) will always be difficult. Whereas, privacy as control over one's data rights is easier because its not about routing or billing. But we don't have to give up completely on privacy as non-correlatable identifiers if we better qualify it to be privacy as protection against the exploitation of correlatable identifiers (metadata). In simple terms, the goal becomes limiting the exploitation via exploitably correlatable identifiers.

## Privacy as Protection Against Exploitation

To better manage this difficult trade-off between the need to address the parties to a conversation and the need to restrict the disclosure of knowledge of those parties to the parties themselves (i.e. privacy) we focus our attention on managing exploitably correlatable non-content metadata, specifically ***exploitably correlatable identifiers***. Thus if we can protect against exploitation derived from correlation of identifiers we may achieve effective privacy. 

Protection against exploitation via correlation provides a more gentle trade-space than merely protection against correlation. The former is practical and amenable to cost-benefit analyis, the latter is largely impractical.

Let's return to our public road delivery analogy. We can achieve effective privacy over public roads if we use a delivery service. The delivery service uses the public roads but the contents of the delivery vans are not viewable by bystanders (third party observers). Although a bystander can see the van enter and leave our property then can't see the boxes the van delivered or picked up from our property. The van has a public address (its license plate), The boxes the van picks up or delivers all have public addresses. But the addresses on the boxes and the contents of the boxes are not veiwable by the bystander. The van makes stops at many houses in the neighborhood which provides herd privacy with respect to correlating van deliveries and pickups to boxes. Analogously the contents of the van may be considered content whereas the van itself is non-content metadata. The van provides a virtual private delivery network over public roads. The closest analogy in the communication's world are VPNs (virtual private networks). The limitation is that we have to trust the delivery driver of the van to not tell our neighbors the address information on the boxes delivered or picked up from our address. The driver can't tell what's in the boxes, merely who they came from or who they are being sent to. 

Let's extend the delivery driver analogy. Suppose that a malicious adversary replaced the driver with one of its own so that they could view all the addresses of all the boxes inside that delivery van and maybe even open the boxes to view their contents. Now any trust in the delivery van service is broken because of malicious impersonation of the delivery van driver. We must be careful to not mistake this as a privacy problem but to see it as an authenticity and confidentiality problem. If the delivery van driver is not impersonable because any entity at any pickup address can securely authenticate the driver then the adversary will not be able to pickup boxes much less view their addresses or contents. The exploitability of the public addresses on the boxes is derived from the exploitability of the delivery service's authenticity and confidentiality, not the inherent correlatability of the boxes' public addresses.

At first blush, it seems that the solution to protecting address privacy is not a private road network but un-impersonatable delivery van drivers with really secure delivery vans and really secure boxes inside the delivery vans. We would be better off building more secure more trustworthy delivery services (VPNs) that use public roads  than to build something more complicated and harder to adopt like replacing public roads with private roads (distributed private communication networks).


## Cold War versus Hot War

In a strong sense, given the definitions above, authenticity and confidentiality protection is in a different class or regime from privacy protection.

Authenticity and confidentiality are like a ***cold*** war.  Authenticity overlays based on public key digital signatures provide arbitrarily strong protection with minimal resources. Likewise confidentiality overlays based on public key encryption provide arbitrarily strong protection with minimal resources. To reiterate, the reason authenticity and confidentiality are analogous to a cold war is that we have existing tooling that enables cost-effective cryptographic strength authenticity (as in end-verifiable attributabillty to an AID) via digital signatures and cost-effective cryptographic strength confidentiality (as in secure end only viewability by an AID) via asymmetric en-de-cryption. This strength is such that attack is computational infeasible against the crypto and the only weakness is in the strength of the key management of the associated key pairs. So there need be no casualties in this cold war. The only looming vulnerabilty is that a secret weapon might be developed (aka a quantum computer) which would break the existing crypto. But given that we now have quantum safe, signatures and asymmetric encryption, as long as the key management system is sufficiently agile and post-quantum proof (like KERI), it will remain a cold war because one just switches to the quantum safe crypto once someone has built a quantum computer capable of breaking the existing crypto.

In comparision privacy is like a ***hot*** war. Rapidly evolving tactics and technology makes privacy protection like a resource-constrained war of attrition against vastly superior opponents. We have to be careful because decreasing the privacy attack surface may actually increase the net total harm due to lost value capture opportunities. To reiterate, the reason privacy (as exploitable correlatability of identifiers) is a hot war is that casualties are unavoidable.  The stategy, tactics, techniques, resources, and incentives to expend those resources for correlation are a function of the particular adversary. So the protection tactics and strategy are specific to the type of adversay. It becomes more about minimizing the casualities and the expended resources so as to not lose the war.

Suppose we want to build a communications channel that is not exploitable via correlation of the identifiers. Because the communications must be routed between to end-points, all of the routing identifiers, the routing tables, and every intermediary are all attack surfaces for correlation. Each of the receiver and sender are themselves attack surfaces for correlation independent of the channel. 

The channel defines the boundary between in-band and out-of-band. For some adversaries even out-of-band mechanisms are correlatable. So if our protection must account for any all all correlation mechanisms from any and all types of adversaries including those with virtually unlimited resources to correlate both in-band and out-of-band then there is no practical solution to the privacy protection problem.  One can always play the equivalent of a trump card that says, here is a way some adversary given enough resources can correlate and so your protocol is broken in terms of privacy. This is why the qualifier exploitable correlatability is used. The type and degree of exploitability depends on the adversary. 

As a result we must make risk mitigation trade-offs between exploitable harm and protection cost by focusing on exploitable correlatability of identifiers used to convey information over a communications channel.  

Let's use a related war analogy, suppose one is designing a tank and needs to select the thickness of tank's armor. One can't determine the armor thickness needed until and unless they scope the types of attacks that the tank must be able to withstand. If one of the allowed attacks is a nuclear bomb, then the tank designer has an impossible trade-off because a tank by definition is a type of mobile artillery and armor thick enough to protect against a nuclear bomb would be too heavy to move.

In the hot war of privacy, we have to pareto optimize protection mechanisms. Notwithstanding the limited resources at our disposal, we can still protect against many types of attacks from many types of adversairies but not all attacks from all adversaries. 

One way to approach this is to use a risk mitigation rating and ranking process. In critical systems one standard process rates and ranks risks by quantifying the magnitude of the harm for a given risk and then multiplies it by the likelihood of the harm. Risks with high harms and high liklihood are ranked ahead of risks with high harm but lower likelihood. 

A cost benefit analysis is then performed on the relative cost of any mitigation measures as a percentage of all available resources to mitigate all harms from all risks. A comet strinking the earth, for example, might have very very high harm but its liklihood is so small and the cost of mitigation so high that its not a risk worth mitigating.

In any resource constrained decision problems we have to make trade-offs and we need to understand the trade-space well if we want to make optimized trade-offs. In a later section we will discuss that threats and mitigations to better understand that trade-space.

On the other hand, in the cold war of authenticity and confidentiality, even with limited resources, we can protected against almost all attacks from all adversaries. In the next section we will detail specifically what strongest practical authenticity and confidentiality protection looks like.

## Three Party Exploitation Model

For this analysis we use the three party disclosure and correlation exploitation model. The 1st and 2nd party represent the parties to a conversation or exchange. The 1st party discloses data to a 2nd party. The 1st and 2nd party have knowledge of each other via thier identifiers. The 1st party as discloser has knowledge of the disclosed data (content). As a result of the disclosure the 2nd party as disclosee also has knowledge of the disclosed data (content). The identifiers are non-content meta-data and the disclosed data is content data. A 3rd party is any party that is not either a 1st or 2nd party. An intermediary may be considered a 1st, 2nd, or 3rd party depending on how and by whom it is trusted or not. 

Privacy with respect to the 3rd party is protected if the 3rd party has no knowledge of the identifiers used by the 1st and 2nd party for the conversation (disclosure). Confidentiality with respect to the 3rd party is protected if the 3rd party has no knowledge of the disclosed data (content of disclosure). A 3rd party may break privacy by directly observing messages that contain the identifiers of the 1st and 2nd party. A 3rd party may break confidentiality directly by observing the content of messages between the 1st and 2nd party. In addition the 3rd party may break both privacy and confidentiality by collusion with the 1st or 2nd party or via collusion with an intermediary. This is diagrammed below.

![](https://hackmd.io/_uploads/S1FBoB8gh.png)

## Exploiters

The three main classes of potential exploiters are as follows:

1. State Actors (3rd party): 
    The primary incentives for state actors are political but some like North Korea are also monetary. North Korea is responsible for a significant percentage of ransomware attacks (see [NK Ransomware](https://www.bleepingcomputer.com/news/security/north-korean-ransomware-attacks-on-healthcare-fund-govt-operations/), [NK Success](https://www.reuters.com/technology/record-breaking-2022-north-korea-crypto-theft-un-report-2023-02-06/)). 
    State actors may use any combination of attacks to surveil, regulate, persecute all first parties, second parties and intermediaries with virtually unlimited resources and ability to cause harm.
    Because of the virually unlimited resources and techniques that state actors can apply we will treat them as out of scope for TSP mitigation for the time being. That does not mean we will not consider state-actor resistent mitigations but to attempt state-actor proof mitigations would likely take us down rabbitholes that we can never return from.
    
2. Identity Thieves (3rd party): 
   The primary incentives for identity thieves is monetization throught asset theft  either directly via capture of access credentials to asset or indirectly through the sale of personal identifing information (PII) that may be used to capture access credentials. Another primary monetization strategy is ransomware of either access credentials or PII that may be used to capture access credentials or simply sensititive PII such as health data.

3. Aggregators (3rd party, 2nd party, 1st party): 
    The primary incentive for aggregators is to sell data to advertisers. This means profiling groups of potential customers in attractive demographics. This profiling may be entirely statistical. There are several classes of aggregators. These include:
    - Intermediaries (3rd party):
        * Search as aggregator
        * Cloud provider as aggregator
        * ISP as aggregator
    - Platforms (2nd party) 
        * Colluding 2nd parties as aggregators 
    - Inadvertent (1st party) 
        * Inadvertent colluding 1st party as aggregator via web cookies


## Identity Theft: Threats and Mitigations

### Threats

For this analysis we assume that content data is confidentially encrypted between
1st parties and 2nd parties when sent over the wire. The identity thieve cannot view this data without first compromising 1st party or 2nd party credentials in order to either decrypt the data or view it in its unencrypted state on a storage device. The ultimate target is a treasure trove of sensitive data held by a 2nd party. This may be health or other data that may captured as ransomware or it may be credentialing information that enables first party impersonation in order to either directly capture 1st party assests or may be sole to secondary thieves who then use it to capture 1st party assets. Another target are the asset access credentials of high-value 1st parties. Notable is that identity thieves do not monetize aggregated content data or metadata (anonymized or not) by selling it to advertisers. The return on investement for data aggragation is too low for identity thieves in general.

The primary attack mechanisms is labeled differently in different venues but may be described as a multi-step recursive privilege elevation attack (authentication and authorization). The attacker compromises weak access credentials on low value targets that allow it to elevate it priviledges to compromise higher value targets until it finally has the access credentials of a treasure trove. An example of this would be to compromise the VPN login of some customer or employee of some company that allows an attacker to compromise some other employee's VPN login to a connected company and so on until it captures access credentials to a connected company's treasure trove. It starts with what is called an edge attack on the weakest company. The elevation mechanism often leverages what is called a [BOLA](https://heimdalsecurity.com/blog/what-is-broken-object-level-authorization-bola/)  or  [BUA](https://github.com/OWASP/API-Security/blob/master/2019/en/src/0xa2-broken-user-authentication.md) vulnerability in the access API of a given company. These vulnerabilities singularly or in combination allow an adversary to elevate their access priviledges (see [BOLA Explained](https://www.traceable.ai/blog-post/a-deep-dive-on-the-most-critical-api-vulnerability-bola-broken-object-level-authorization). The [OWASP](https://owasp.org/www-project-api-security/) project lists in order the primary API vulnerabilities that enable a recursive priviledge elevation attack. These are not privacy weaknesses but authentication based authorization weaknesses.

This is described thusly:
>Hackers can use data stolen from companies with weak security to target employees and systems at other companies, including those with strong security protocols. ... the data ecosystem has become so vast and interconnected that people are only as safe as the least secure company that interacts with any company that has access to their data. That “least secure company” does not even need to have access to the consumer's data. … There were 5,212 confirmed breaches in 2021, which exposed 1.1 billion personal records across the globe” (see [data security](https://www.apple.com/newsroom/pdfs/The-Rising-Threat-to-Consumer-Data-in-the-Cloud.pdf))
>

To summarize, a given trust domain is only as strong as the weakest connected trust domain (which connectivity is recursively applied). 

A recursive priviledge elevation attack usually starts by a 2nd party impersonation using phishing and smishing to steal 1st party credentials to the impersonated 2nd party. Now that the attacker has 1st party access they can leverage that to discover other first parties that have access to higher value connected clients and rinse and repeat.
A related elevation attack is to attack a weak intermediary which may be used to stage a more sophisticated 2nd party impersonation attack. The classic DNS hijack is such and example. An adversary hijacks (using cache poisoning or some other mechanism) the DNS server of some 2nd party. The adversay then requests a new DNS certificate from a CA. The CA verifies control of that DNS domain by querying the DNS Zone file for the impersonated 2nd party secret provided for authentication. The hijacked DNS returns the secret thereby fooling the CA into issuing a valid certificate to the adversary. The adversary then hijacks the DNS server of a first party so that when they go to login to the 2nd party host they are redirected to a host under the control of the adversary. The 1st pary verifies the valid DNS credential misissued to the adversary and then logs in thereby allowing the adversary to steal the 1st party credentials. The adversary then logs into the 2nd party host with 1st party access. This can be repeated.
A related attack is to first compromise an intermediary using either a direct or combined attack and then leverage the surveillance of first-party to second-party traffic via the now compromised intermediary to mount attacks on the second-party in order to elevate to first-party credentials.
A more sophisticated attack is a code supply chain (intermediary) attack that creates a back door to capture first-party credentials.

### Mitigations

* Use strong authentication by second-parties of any first party at any step along a multi-step recursive priviledge elevation attack. In any multistep recursive priviledge elevation attack any step that has strong authentication with indivdually signed requests for authorizaton will stop the attack from proceeding.  
* Use strong authentication by any second-party of any first-party for any exploitable use of first-party credentials, whether those credentails be credit card, bank account or access to sensitive data. 
* Use strong authentication (zero-trust data in motion and at rest) at any party holding a treasure trove. Strong authentication is also state actor resistant.
* Get rid of treasure troves of access credentials by using decentralized trust domains (DPKI). This means passwordless security with strong key rotation mechanisms. This is also state actor resistent. 


## Aggregation: Threats and Mitigations

For this analysis we assume that content data is confidentially encrypted between
1st parties and 2nd parties when sent over the wire. The aggregator must either merely aggregate the correlatable information contained in the non-content meta-data such as identifiers or must be a 2nd or 1st party with access to 1st party data. The aggregator needs voluminous data in order to provide sufficient monetization via advertizing. This means attacks to compromise 1st or 2nd party credentials are too costly given the amount of data captured by such attacks. Moreover advertisers generally are legitimate companies that would incur a huge legal liability should they engage in direct attacks on 1st party credentials. Their primary mode of operation is to incentivize first parties to disclose exploitable information either simply as correlatable metadata or as content data. Typically, data may be anonymized and then re-correlated to demographic pool sizes appropriate for advertising campaigns. As mentioned above there are several classes of aggregators each with a different exploit mechanism.


* Intermediaries as Aggregators
    1. Search as intermediary (3rd party).  
        In this case an internet search engine is able to track the click-throughs of a successful search result. This enables the search enging to monetize the first party searcher by correlating meta-data of the incoming client browser with the second party endpoint resulting from the search.
        * Mechanisms: 
            - 1st party permissioned surveillance of first party metadata collected by  3rd party search intermediary and sold to other 3rd parties
        * Mitigations:
            - Use VPN in combination with a partitioned 1st party identifier such as a random email address
           - Use a search engine that contractually protects 1st party search metadata.
    2. Cloud data storage provider as an intermediary (3rd party).  
        In this case all 1st party content data stored in the cloud may be anonymized and re-correlated to provide monetizable advertising campaign pools.  
        * Mechanisms:	
             - First-party permissioned surveillance of first-party content data by 3rd party cloud provider intermediary and sold to other 3rd parties.
        * Mitigations:
            - Use cloud storage providers that use end-to-end encryption (first-party confidential content data)
            - Use cloud storage providers that contractually protect 1st party content data.

    3. Internet Service Provider (ISP) as an intermediary (3rd party).  
        In this case ISP has access to all traffice routed through it for 1st party client connections to any 2nd party. ISP can anonymize and then re-correlate into monetizable advertising campaign pools.
        * Mechanisms:
            - Non-content Metadata (routing and billing) identifiers. In the USA for example any anonymized data (both metadata & content) is aggregation permissible for any ISP.
            - Mobile device location tracking.
        * Mitigations:  
            * Use ISP that contractually protects user metadata
            - Use VPN that contractually protects user metadata. A VPN provides herd privacy relative to ISP tracking of source to 2nd party destinations.
            - There is no mitigation for mobile device tracking by mobile service provider except to turn off the mobile device.
            - Use distributed decentralized confidential network overlay to make metadata harder to correlate. This could include onion or mix network routing such as TOR, Elixxir, MainFrame, DIDComm Routing. In general a decentralize network overlay are difficult to incentivize and as a result have relatively low adoptability when compared to using a VPN.
            - Use information-theoretic secure routing. An example, would be arandom walk routing algorithm. Of the mitigations this one is state actor resistant. Informations theoretic secure routing has similar adoptability problems as decentralized confidential network overlays.


    4. Platform as an aggregator (2nd party).  
        In this case the platform is a 2nd party with access to all the 1st party data disclosed to it. This includes both non-content metadata and content data. The platform anonymizes the data and recorrelates it into monetizable pools for advertizing on the platform itself. A related approach is for smaller 2nd parties to collect their data and sell it to 3rd party data aggregators who then package it for third party advertisers. Statistical techniques like regression enable monetizable aggregation even with weakly correlatable data given enough 2nd party data. (see [1st party data correlation](https://www.fastcompany.com/90773053/online-marketing-privacy-tracking-apple-facebook))
        * Mechanisms:
            - First-party permissioned content data and non-content metadata is anonymized and then statistically regressed to form monetizable advertising campaign pools. Even anonymized 1st party data without correlatable identifiers has sufficent exploitable correlatabilty for monetization. Privacy protected metadata (identifiers) are insufficient to protects against exploitation. Data anonymization is not protective (fallacy of K-Anonymity).
        * Mitigations:
            - Don’t interact with 2nd parties that aggregate without permission.
            - Interact only using contractually protected disclosure alone or in combination with contingent or selective disclosure.

    5. Inadvertent 1st party as aggregator (1st party, 2nd party, 3rd party).  
    In this case the 1st party uses web browser cookies or other permissioned mechanisms that make the 1st party itself the tracker that may be exploited by either 2nd parties or 3rd parties that may glean data from the tracking mechanisms. The data is typically metadate but may also be content data depending on the invasiveness of the cookies. 
        * Mechanisms:
            - Browser cookies makes first-party browser an inadvertent permissioned intermediary that can leak both non-content metadata and content data to 2nd parties and/or 3rd party intermediaries. This enables those parties to aggregate 1st party data and monetize it for advertizing campains.
        * Mitigations:
            - Don’t use browsers in a way that supports tracking first-party data via cookies by 2nd parties or 3rd party intermediaries. All the major browsers now support a type of in-cognito mode that prevents the use of cookies for tracking. Most 2nd party websites support disabling most cookie based tracking.
            - Use browsers that contractually protect 1st party data and anonymize the 1st party source identifiers.
            - Use VPNs that contractually protect 1st party data and anonymize the 1st party source identifiers.


## Summary

As the analysis above shows the most fruitful application of TSP based protocols with regards exploitbly correlatable identifiers may be in improving VPNs to limit aggregation for advertising by 3rd parties or in providing TSP based protocols that enable VPN like trusted intermediaries for herd privacy against 3rd party aggregators. Whereas protection from 2nd party aggregators may be best provided by higher layer protocols that provide contractually protected disclosure of 1st party data to 2nd parties. Finally, protection from identity theft may be best provided with TSP based protocols that provide strong authenticity and confidentiality independent of correlatable identifiers. Proper full zero-trust (verify everything) approaches with cryptographic roots-of-trust for authentication and authorization simplifies API design and may mitigate the most common vulnerabilities (including BOLA and BUA).

