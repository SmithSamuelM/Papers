---
tags: KERI
email: sam@samuelsmith.org
comment: Local Copy
---


# KERI Request Authentication Mechanism (KRAM)


Originally I solved this decentralized end-to-end non-interactive authorization problem as part of a proof-of-concept for a privacy preserving lost and find registry and peer-to-peer messaging service. This proof-of-concept was implemented in python in an open source Apache2 project called Indigo-BluePea which may be found here. [Indigo BluePea](https://github.com/reputage/bluepea)  

### Overview

KRAM provides an efficient, low-overhead way to authenticate requests between two hosts using signed, timestamped requests and a single-message, timed request cache. The requestor signs and timestamps each request and the requestee stores only the last received message from the requestor in the single-message, timed request cache. The requestee compares the timestamp of any new signed request to the timestamp of the message in the single-message cache and if the timestamp of the new message is greater than the cached message then the new message is accepted, processed, and then placed in the single-message cache. This monotonically increasing cache timestamp completely obviates the need for nonces as it provides the same protection against replay attacks. Avoiding the overhead of nonces is a major benefit of KRAM as it enables scalability and simplifies implementation.

It bears pointing out that KRAM effectively synchronizes message processing between two hosts due to the monotonic message processing guarantee provided by the timestamp provided in the signed message and the single-message cache.

# Exposition

## Interactive vs. Non-interactive Authentication Design

Authentication mechanism broadly may be grouped into two different approaches, these are: interactive and non-interactive approaches. An interactive mechanism requires a set of requests and reponses or challenge responses with challenge response replies for secure authentication. Non-interactive approaches on the other hand pose unique problems because they do not allow a challenge response reply handshake. A request is submitted that is self-authenticating without additional interaction.  The main benefits of non-interactive authentication are scalabilty and path independent end-to-end verifiability. These benefits become more important in decentralized applications that employ zero-trust architectures. (By zero-trust we mean never trust always verify, this means every request is independently authenticated, there are no trusted pre-authenticated communication channels.)

For non-interactive authentication of some request for access, the most accessible core non-interactive mechanism is a non-repudiable digital signature of the request itself made with asymmetric key-pairs. The hard problem for asymmetric digital signatures is key management. The requester must manage private keys. Indeed this problem of requiring the user to manage cryptographic keys, at least historically, was deemed too hard for users which meant that only federated, token based, authentication mechanisms were acceptable. But given that it's now commonly accepted that users are able to manage private keys, which is a core assumption for KERI in general, then the remaining problem for non-interactive authentication using non-repudiable digital signatures is simply replay attack protection. Indeed, with KERI the hardest problem of key management, that is, determining current key state given rotations, is already solved. 

The closest authentication mechanism to what KERI enables is the FIDO2/WebAuthn standard [FIDO2/WebAuthn](https://developers.yubico.com/WebAuthn/WebAuthn_Developer_Guide/Overview.html). The major difference between FIDO2/WebAuthn and KERI is that there is no built-in, automated, verifiable key rotation mechanism in FID02/WebAuthn. FIDO2/WebAuthn consists of two ceremonies, a registration ceremony and then one or more authentication ceremonies. Upon creation of a key-pair, the user engages in a registration ceremony to register that key pair with a host. This usually involves some MFA procedure that associates the entity controlling the key pair with the public key from the host's perspective. Once registered then individual access to a resource protected with FIDO2/WebAuthn may be obtained through an authentication ceremony that typically involves signing the access request with the registered private key.  Unfortunately FIDO2/WebAuthn has no in-stride verifiable key rotation mechanism. Should a user ever need to rotate keys, that user must start over with a new registration ceremony to register the new key pair for that user entity. Whereas, with KERI rotation happens automatically with a rotation event that is verified with the pre-rotated keys.
So given one already has KERI verified key state, using FIDO2/WebAuthn to authenticate replay requests would be going backwards from a security capabilities standpoint.

Any access control or authorization mechanism based on KERI must still have some additional registration mechansim in order to link or associate an actual entity as the controller of a given identifier. However, in this case where we are enabling access for the purpose of replaying key event logs, it may not be necessary to link to any external entity. We merely need authenticate to the controller of the identifier regardless of the actual external entity. Thus authentication can be simply restricted to authenticating as the controller of the identifier via its current key state without requiring a separate registraion step to link to an external controlling entity. With this we can build a base level private authorization policy for a given identifier using a simple authentication mechanism based only on proof of control over the authorized identifier by signing a request or query with the current controlling key pairs of that identifier. This does not preclude an implementation from layering on additional authentication and authorization mechanisms such as external entity associated registrations or MFA. But the minimal simple authentication mechanism proposed here is meant to enable this layering without requiring it. 

## Replay Attack Protection in Non-interactive Authentication

A digital signature made with an asymmetric key pair(s) on a request provides non-repudiable authentication of the requester as the controller of that key pair(s). This assumes that the receipient of the request (requestee) can securely map the identifier of the requestor to the controlling key pair(s). With KERI derivation codes, we have a simple mechanism for attaching and resolving both the identifier of the requestor and the requestor's signature of that request to that request. Given that the requestee has the current key event log for the requestor, then the requestee may securely verify the mapping from the requestor identifier to its associated current key state. Indeed, we could view the publication of the key event log of a given requestor to a given requestee as an ersatz registration ceremony that then enables later authentication of signed request messages that include both the requestor's identifier and signature as attachments to the request. Presentation of the identifier and request signature along with a request is used as an authentication of that specific request.

The problem with such authentication is that an attacker may capture the signed request and then resubmit it or replay it from its own host, thereby effectively fooling the requestee into redirecting a copy of the response to the attacker's host in addition to or instead of the original requestor's host. In other words the attacker acts as an imposter. This requires that the attacker intercept the original request and then replay it from a different connection. This is called a replay attack. There are several mechanisms one can employ to protect against this form of replay attack. However the practical choices for non-interactive protection are limited. In general, replay attack protection imposes some form of timeliness or uniqueness to any signed request.

In KRAM, we assume that only the current keys from the current key state may be used for authentication. This requirement imposes one form of timeliness, in that any requests signed with stale keys are automatically invalid. The requestor can ensure that the requestee has its, (the requestor's) latest key state so that it is protected from replay attacks using old, stale key signed requests.

The simplest and most practical non-interactive mechanism for ensuring timeliness and uniqueness is to insert a date-time stamp in the request body and have the requestee check this date-time stamp, assuming synchronization of clocks between requestor and requestee. The date time stamp of the signed request is relative to the date time of the requestee not requestor. There are two protection mechanisms, one loose and one tight. In the loose mechanism the requestee imposes a time window with respect to its date time for any request. The time window is usually some small multiple of network latency of the connection to the requestor. An imposter must intercept and resubmit the replay attack within that window in order to successfully redirect a response to itself. This is loose protection because the requestor is not guaranteed to be able to detect the attack. The requestee may respond to multiple copies of the same signed request but from different source addresses. In addition to the loose mechanism of timeliness, the tight mechanism imposes a uniqueness constraint on the request timestamps. The requestee does this by keeping a cache of all requests from the same requestor identifier (not host address). All requests in the cache must have monotonically increasing timestamps. The requestee only responds once to any request with a given time stamp and any subsequent requests must have later timestamps. This means that a  successful replay attack may be detected because the only way that the attacker gets a reply response is if it redirects the one and only one response to that request first to itself before possibly forwarding it on to the requestor (if at all). This redirection is detectable because either the requestor does not get the response at all, if it is not forwarded, or it gets it via a later redirection from some host that is not the originating requestee. Given that the requestor detects the attack it may then take appropriate measures to avoid future interception of its traffic by the attacker.

The monotonic cache even protects against a retrograde attack on the system clock of the requestee. The monotonicity requirement means that even if the system clock is retrograded to an earlier time such that an attacker could replay stale requests, the cache would have later timestamps and therefore prevent response until the system clock was restored to normal time.

The datetime stamp should have fine enough resolution that the monotonicity constraint does not limit the rate of requests nor cause it to run past the end of the timeliness window. Microsecond resolution is adequate for most internet connections but may not be for 40 Gbps LAN connections. An ISO-8601 timestamp with microsecond resolution is a common variant. Most operating system now also support nano or atto time variants of ISO-8601 timestamps.

## Authentication Request Timeliness and Caching

Generally speaking all authentication mechanisms, both interactive and not, depend on some explicit or implicit datetime stamp for timeliness. Interactive mechanisms that use tokens typically embed the datetime stamp or its hash in the token itself. This is because tokens are bearer authentication mechanisms and need to expire quickly to prevent replay. A common, non-token based interactive authentication mechanism requires that the requestee send a challenge response with a nonce to the requestor. The requestor then includes the nonce in its signed challenge reply. This ensures uniqueness of the signed request (as challenge reply). But even then its still best practice to have a timeout on the authentication because an attacker that intercepts the traffic can still replay the challenge response unless the requestee enforces one and only one reply per request. This is especially problematic in asynchronous networks since the routes by which the set of request, challenge, response, reply messages are exchanged could all be different and each subject to MITM attacks.

One way to mitigate MITM attacks on async interactive authentications using a nonce (challenge response) is for the requestee (host) to keep a cache of challenge responses and only reply once to any nonced signed response. The nonce must be a salty nonce generated by a CSPRNG instead of merely a nonce. The salty nonce must have sufficient cryptographic entropy that its universally unique, i.e. the host can never generates the same nonce for any two interactive requests. At scale this means either a database of all requests, which will grow without bound, or there must be a timeout mechansim on a cache to allow old entries to be deleted. 

Absent such a database or timeliness cache, an attacker might capture the reply from the requestee to the signed challenge response and replay the response as a signed bearer token. The attacker can forward the reply to the requestor to hide that fact that it has intercepted the response and reply as a MITM. To reiterate, any intermediary that intercepts the signed response now has a bearer token it can replay unless the requestee enforces one and only one request per nonce. Thus either a database that grows without bound or some timeliness mechanism must be used to enforce the one and only one reply per request. 

One timeliness mechanism could be that the requestee imposes a timeout on nonced reply requests that invalidates any stale, replayed nonced requests. This approach enables the requestor to detect an attack because there is one and only one response to any request. Given a successful attack, either the requestor does not get the response or it sees that the response has been redirected from some other host that is not the requestee.

Consequently, from a replay attack protection perspective, a nonce with a time window in an interactive authentication is functionally equivalent to a monotonic time stamp in a non-interactive authentication but the latter avoids the overhead of the interaction. Both employ a timed cache (timeliness) tied to each requestor and both employ a mechanism that ensures one and only one response is sent for any signed request (uniqueness). 

The typical use case for interactive authentication is not on an asynchronous channel but to use a synchronous channel such that the host can pair each request and response and replay without needing a database. In a sense, a synchronous channel provides a weak form of timeliness given the ordering of messages from the time of the establishment of the synchronous channel. This sort of works if the host enforces that there may be one and only one outstanding request-challenge-response-reply interaction at a time by caching the latest request. So a replay must be the one outstanding request and it is assumed that a prior request could not be replayed without re-establishing the connection. This means the channel is blocked until either the interaction times out or succeeds. It does provide replay attack protection since any message must either be part of the current interaction or start a new interaction with a request, never a signed response.

However, this only gives the illusion of protection because even on a synchronous channel there may be a MITM that can replay requests out of order. The nonce by itself does not order the requests which means the MITM can reorder requests and so change the transaction state. This is technically not a replay attack; it is a first play attack but can create similar exploits. Therefore, notwithstanding a synchronous channel, in order to guarantee secure, sequential message processing all requests must still be cached so that the host can ensure one reply per request is sent in the originally intended order. 

Therefore, except for a couple of extreme corner cases, there is little advantage to interactive authentication over KRAM's non-interactive authentication. KRAM, with a timed cache, monotonically orders the requests, ensuring no replays and making retrograde attacks detectable. For strict monotonic ordering use a sequence number in addition to the timestamp. KRAM would not be feasible without KERI becasue it depends on key state and signed requests relative to that key state. But given one has KERI there is good reason to leverage it to build a truly scalable and secure request authentication method, e.g. KRAM.

This analysis just reinforces the observation that all practical secure authentication replay attack protection mechanisms include both timeliness and uniqueness properties.

### Caching by intermediaries

Because, generally, replay attack protection mechanisms use timeliness, caching of authentication requests by the requestee or some intermediate load balancer may be an anti-pattern. This applies to both interactive and non-interactive authentication requests. Thus, in general, request caching of authentication requests must be disabled. Replies, however, may still be cached. 

In a zero-trust, non-interactive environment where every request is self-contained and self authenticating with an embedded date time stamp and attached identifier and signature couple, request caching would be problematic. This may be viewed as a trade-off where the scalability advantages of non-interactivity exceed the scalability disadvantages of not being able to cache requests.


## Examples

There is some terminological ambiguity becuase of the extra two messages that are the nonced challenge and nonced response to that challenge.

So to better remove ambiquity lets not use request but query. This uses KERI parlance.
```
Querier                                    Replier
------------------------------------------
Query  ->
                                                       <- Nonced Challenge
Signed Nonced Response ->
                                                        <-   Reply
```
A replay attack is the succesfful reuse of a prior  Signed Response. 
A First-play attack uses Signed Responses out of order.

This approach only works if the channel is synchronous and the Replier caches the current/last interaction (at least the Signed Nonced Response.  This means only one interaction per channel is allowed at any time. This does not work on an asynchronous channel unless a database of all interactions is kept which grows without bound unless a monotonic ordering is placed on the interactions  so that the database may be pruned of stale intereraction which effectively creates a synchronized set of channels. This means at the least putting time stamps in the Query messages and then keeping the latest Interaction for each Querier and dropping any interactions that are earler.
Given this timed ordered cache all the querier has to do differently is sign the query and now you have exactly  KRAM. There is no need anymore for the Nonce challenge response set of messages.


With KRAM you have

```
Querier                                    Replier
------------------------------------------
Signed Query  ->
                                                     <-   Signed Reply
```
But the Query MUST have a datetime and should have a SAID
The Signed Reply should include the Query's SAID
The Replier has a sliding timed window cache that enforces that any Query's datetime must be later than any so far previously received query.

KRAM works just as well on asynchronous channels and peer-to-peer exchanges not simply query-reply between client and server. So KRAM can be applied to exchange messages

```
Peer Initiater                                    Peer Correspondent
------------------------------------------
Signed EXN  ->
                                                     <-   Signed EXN
```

Both sides keep a timeliness cache of EXNs from the other party to protect from replay attacks.

Given that routes in the exns define an interactive protocol that is a transaction of a given type. Then the timed cache can be per transaction type per Peer  not just per Peer.  The timeout on the timed cache can then be of any size bounded only by the memory of the cache across all simulataneous live transactions of a given type for all peers.

Because eaxch EXN includes the SAID of the prior EXN in the transaction, And the first EXN message in the transaction has a datetime stamp  it is not vulnerable to First Play attack.

The transaction itself is ordered and self synchronizing.



## Summary



1. Simple kram uses a short time window relative to the host (i.e the host clock is determinative). All signed client requests must include a timestamp field that lies within the window else they are dropped. This does not require any host cache. A replay attack is only possible inside the time window synchronized to the host clock. A short time window minimizes the opportunity for replay attack. But simple kram may be problematic for multi-sig where the time to collect signatures is longer than the time window. This is what is currently implemented in keripy.

2. Full kram is a monotonic timed cache. The last request from any client is cached inside a sliding time window. A new request must both have a later datetime than the cached request and must be inside the host time window. Requests that move outside the sliding time window may be safely deleted thereby pruning the cache. The timed cache can be made more granular by caching a request per message type for a given client or even more granular by caching messages with a specific time window for a given transaction itself not merely the message type or the transactions type. Essentially, creating an expiration datetime as the request datetime plus the host configured timewindow for all instances of a given message at a given stage in a transaction. The monotonicity of the cached requests protects against replay attacks. The time window bounds the size of the cache. That has not been implemented yet in keripy. The timewindow needs to be controlled by the host relative to the host's clock otherwise the client can attack it.

With a host-controlled sliding window timed monotonic cache, because the monotonicity of the cache protects against a replay attack and the time window merely bounds the memory requirements, one can tune the window per message type (and per aid) and/or per transaction type or per transaction itself, so that messages associated with certain types of transactions or certain transactions themselves can have relatively long time windows (days or weeks) which windows are only bounded by memory and the traffic load for those message from given clients.

## Afterword

A little bit of historical perspective may help. In the early days of computing, there were no network time servers. Consequently timestamps had no meaning. Each party could just lie about time so time was untrustable. Historically a nonce was just a sequence number. To be a nonce it only had the property of being used once. There are other ways to create a nonce. A datetime can be a nonce. A hash of a datetime or the hash of a sequence number etc. In the early days CSPRNGs (cryptographic strenght pseudo random number generator) were both rare and computationally expensive. So a Salty Nonce was hard. What was common were private networks (the internet did not exist yet). Private networks could have really strong synchronization properties. So given these conditions, using extra messages that do not depend on a time stamp but use a simple nonce (not a Salty Nonce) was reasonable. Moreover,  In those days (and for many protocols still used in the wild) signing did not mean digital signatures with asymmetric crypto but merely an HMAC (a hash using a shared secret key). There using an interaction with an extra set of messages i.e. a nonced challenge response for replay attack protection made some sense. Often some CS professor puts a simple mechanism in a text book as an academic exercise to teach a principle and then it gets adopted in the real world by former students not because it was well thought out but because it was familiar.

But today where we assume a KERI context, we can already assume CSPRNGs, assemetric digital signatures tied to keystate of AIDs, cryptographic strength hashes, salty nonces,  robust ubiquitous access to network time servers and a requirement to work over asynchronous public networks. The cryptographic operations have gotten relatively cheap given multiple exponential Moore's law increases in computation performance per cost. What has not scaled relatively exponentially is bandwith per cost. Synchronous networks do not scale as well as asynchronous networks and almost all communication now happens over public networks. So refactoring the solution to support asynchronous networks at scale and eliminate 1/2 of the packets is a huge design win.
