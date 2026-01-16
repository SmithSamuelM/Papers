# ACDC Affordances for Confidential (Sensitive) Data Protection and Management

The novel graduated disclosure mechanisms in ACDCs enable best practices for confidential (sensitive) data protection and management of the recipients of data disclosed via ACDCs. We intentionally use the term "confidential" data protection, not "private" data protection. Best practices for confidential, sensitive, or classified data have benefited from extensive development and refinement over time. They are well established. In contrast, the concepts of private data rights and private data management are relatively recent and are not so well established. This distinction motivates the question: What is the difference? Frankly, regarding data shared over the internet, the difference is in name only. For all real practical purposes, data shared over the internet, whether labeled private and confidential, is the same thing. Yet "private" data disclosure and data management best practices often overlook the better-established confidential data disclosure and management best practices. Indeed, in many cases, "private" data disclosure mechanisms defeat best practices for confidential data management. 

The main difference between the two is that the term "privacy" or "private" carries an emotional bonus compared to "confidentiality" or "confidential". The root meaning of private is intimate, so using the term private to refer to data shared over the internet implies that the data is somehow better protected merely by using a different term. Data confidentiality is boring, but it more accurately reflects how any shared data can be protected. This misuse of emotion to convey a false sense of protection interferes with clear-headed design. Using the term "data privacy protection" imbues the activity with a feel-good virtue rather than the real virtuosity of protecting confidential data.

## Three-Party Model

We use a three-party model for data sharing. The first party is the source of some data. The second party is the recipient of some data shared by a first party. A third party is a recipient of first-party data that is not an intended recipient. 

Given this model, once the second party has received first-party data, there is no technical way for the first party to restrict how the second party uses that data, including with whom the second party chooses to share it. This bears restating: once shared, there is no technology that by itself protects the first party's interests. Therefore, there is no such thing as truly private data once it is shared with a second party. The only mechanisms that enable a first party to protect its interests in data that it has shared with a second party are economic, legal, and regulatory. These mechanisms are well known for their confidentiality protection. These include well-known non-disclosure agreements (NDAs) that contractually bind the second party to protect that confidential data. Similarly, regulations that impose on the second party a fiduciary duty of loyalty to the first party with respect to the use of that disclosed data provide another form of legal protection. The risk of enforcement of either contractual or regulatory protections provides an economic disincentive to violate either legal protection. Finally, the loss of reputation incurred when a second party uses shared data to exploit the first party against the first party's expressed or implied interests can provide additional economic disincentives.

The three-party model can be extended to cases in which the second party shares data with downstream parties. When this sharing is permitted by the terms of either contractual protection or regulatory protection, then the downstream parties become effectively second parties. One way to facilitate such downstream sharing is to use chain-link confidentiality and/or chain-link data loyalty agreements that define the terms of confidentiality and use for downstream parties.

The recognition that the problem is about the confidentiality and permitted uses of shared data, not about the so-called "privacy" of that data, allows us to address the real issues more effectively. It also exposes the legerdemain of de-identifying shared first-party data, which supposedly converts it into data that is no longer private and therefore grants a safe harbor to the second party to share that data without restriction. Which then may be used by any third party in an exploitative manner against the interests of the first party.


## Best Practices for Confidential (Sensitive) Data Protection and Management

Clearly, one of the challenges in confidential data protection and management is how best to prevent confidential data from leaking to unintended recipients, thereby violating confidentiality or data loyalty terms. A leak may be inadvertent due to mismanagement or may be the result of a breach by a malicious adversary. In either case, a leak could result in legal, regulatory, or economic costs. Consequently, confidential data protection is an exercise in risk management for both parties (first and second). The first party must trust the second party's management practices to minimize leaks and breaches. The threat of enforcement with recourse bulwarks that trust. But ultimately, some or all of the shared data may be breached. This reinforces the idea that data shared in this way is merely confidential and ceases to be truly private the moment it is shared.




Bulk Issuance

Sparse Merkle Tree of Independant AID Bulk Issued  set. Root of SMT is internal account ID. 
  Protected service to query SMT for membership of AID in bulk issuance set.  So bulk issued ACDC state updates for bulk  issued AIDs are executed by protected service.
  
  Leaves are blinded, so even if database is breached attacker can't discover AID without blinding factor. Blinding factor can be sharded or multiply encrypted so no single point of failure.
