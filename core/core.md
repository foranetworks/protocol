# Fora Protocol

## Introduction and Definition of Terms

As mentioned in the README, the Fora protocol relies on ActivityPub run over a decentralized gossip network to propogate the state of the community to all its members, and it relies on a blockchain as a censorship-resistant ledger of community votes and moderator decisions.

In order to understand the Fora protocol, it is useful to define the entities involved in a Fora community.

**Member**: This is a member of the Fora community, they may or may not host the Fora community. A member that hosts the Fora community will specifically be termed **Host** and a member isn't a host and relies on a Host to connect to the community will be specifically termed **Light User**. (NOTE: This is analogous to the Full Node/Light Client distinction in blockchains). Members have a unique name in the community and an associated public key.

**Host**: As mentioned above, a host is a member that maintains the full state of the community. They will receive messages from other members of the community over a gossip network and update their own state, as well as gossip any updates to the rest of the network. They may or may not serve as an access point for Light Users. Hosts will record and propogate community interactions, and they must also listen to events on the blockchain in order to abide by any decisions made there. An honest Host will delete any banned content from their local copy of the community and ignore any banned members by refusing to accept and propogate their messages.

**Admittors**: Communities may optionally elect a trusted member to the role of admittor. Admittors are responsible for admitting new members into the community. They may also assign voting power to members (if this is allowed by the community). They are appointed and replacable by governance.
Admittors record their decisions on the blockchain, at which point all honest Hosts will start accepting messages from the newly admitted members.
It is up to the community how admittor(s) add members to the community. It may require 1-of-n, k-of-n, or a unanimous approval from the admittors to add a new member to the community. This should be specified in community rules and enforced by blockchain.

**Enforcers**: Communities may optionally elect a trusted member to the role of enforcer. Enforcers are responsible for enforcing the rules of the community by banning content that violates the rules, and banning members that consistently violate the rules. They are appointed and replacable by governance.
Enforcers record their decisions on the blockchain, at which point all honest Hosts will no longer propogate or serve banned content, and will ignore any incoming messages from banned members. Note, all previously sent messages from banned members are by definition banned as well.
It is up to the community how enforcers coordinate with each other to enforce the rules. It may require 1-of-n, k-of-n, or a unanimous approval from the admittors to ban contents or members. This should be specified in community rules and enforced by blockchain.

**Blockchain Validator**: Blockchain validators run the blockchain on which the governance decisions are recorded. As a set, they must be sufficiently distinct from the set of community authorities. If not, then malicious community authorities can influence the blockchain to censor votes and retain power. Ideally, the blockchain validators are very far removed from the community and are ambivalent to its internal politics.

## Community Interaction

Now that we've defined the terms of the Fora protocol, we can describe how the members of the community are meant to interact with each other.

Members send messages to the community by sending an ActivityPub message signed with the private key that is associated with their public key. Members must also include a sequence in their message along with the hash of the previous message to ensure consistent ordering and to detect any dropped messages.

If the member is a Light User, they must then send this message to a Host of their choice. If the member is a Host, they simply propagate their message to the rest of their peers.

Any Host receiving a message must first check that the member it is coming from is a valid member of the community (ie accepted and not banned), and then verify that the signature is valid, and that the content is valid and not banned. If all these checks pass, the Host will apply the message to modify its own internal state and then gossip the message to the rest of its peers.

Hosts themselves will have access to the full state of the community, subject to liveness constraints. However, Light Users must query a Host in order to get the relevant parts of the community state. At any given point, a Host may not have access to the latest community state. If this is the case, the Light User may query other Hosts to fill in the missing information, or they may wait for the Host to query this information itself and update the Light User's feed.

Missing information from other members will be sometimes be detectable if there are missing sequences in the member's message history. If this is the case, then Hosts can query for the missing information if it indeed exists. Since malicious members may intentionally create missing sequences, Hosts and Light User implementations should not make the assumption that this message actually exists.

Messages cannot be forged since all messages accepted by Hosts and Light Users must include the signature that comes from the sending member.

Thus, Hosts and Light Users can send ActivityPub messages to the community and receive ActivityPub feeds over a decentralized gossip network with no loss of integrity and eventual consistency subject to liveness constraints.

## Community Governance

### Instantiating a Community and Community-wide Governance

As mentioned above, all governance decisions are recorded on the blockchain. The community must first be instantiated with some parameters that define how blockchain governance works. These include parameters like voting period, delay period (for the delayed effect of certain governance decisions explained later), the max number of Admittors and enforcers, and the algorithm for Admittor/enforcer decisions (eg 1-of-n, k-of-n, simple majority, unanimous consent).

The community must also be instantiated with how the community vote itself will work. These parameters are the various voting thresholds necessary to change governance parameters, change text rules, and electing Admittors/enforcers.

While the community must start with an initial choice of parameters, these parameters can always be changed through a Parameter Change Proposal via a vote that is subject to the parameters currently specified.

Once these parameters have been instantiated, the community may choose to vote for text-based rules that govern the community. These cannot be understood and enforced by the blockchain itself, but it will be enforced by the elected enforcers.

Every part of governance (parameters, rules, and appointees) are subject to change via a vote by the community.

The community must also be instantiated with an initial set of members and an initial distribution of voting power. Once this has been instantiated, an Admittor may be elected who can add new members (and perhaps assign them a voting power).

**Community Governance Security Model**:

Note that the governance protocol relies on the people who are running it. Thus, any instantiation of a community can be taken over by bad actors that the community originally wanted to shun. The security model of any community hinges on the thresholds specified in the Community-wide governance parameters. The higher these thresholds are, the more bad actors there must be as a proportion in order to take over the network. However, thresholds that are too high might make it impossible to make any community-wide decision. Thus, communities must strike a balance between making their thresholds low enough to be dynamic and flexible while also high enough to prevent a hostile takeover of the community.

Thus the Fora protocol must assume that malicious actors will never reach the threshold proportion of total voting power necessary to control community-wide governance votes.

### Admittor Decisions

While it is important that new members are able to join in order to keep the community dynamic, malicious admittors can use their power to flood the community with new members that are controlled by the admittor and thus take over the governance of the community. Thus, to prevent this we enforce a delay period (parameterized by governance) on the voting power of the new members. Thus, while newly admitted members can immediately start interacting in the community; the delay period must pass first before their voting power becomes active. The delay period **must** be greater than the voting period necessary to change the admittor.

This gives the community enough time to detect the attack and replace the Admittor before the Admittor takes over community governance, while also allowing new members to immediately participate in the community and giving them voting power later on.

If the Admittor gets replaced for malicious activity, then any member that is still in the delay period will have their voting power remain 0. The new Admittor must **explicitly** choose to give back the new members their voting power, at which point the delay period gets reset and must pass again from that point. However, these members can still interact with the community unless the admittor or enforcer **explicitly** kicks them out of the community.

Admittors may remove members who are still in the delay period, but cannot remove established members. This gives established members protection from admittors but allows admittors to kick out new members who were maliciously added by an old admittor or to simply cull inactive new members.

Note: This can be inconvenient for honest members that are in the delay period when the admittor gets removed for malicious activity, since they must wait even longer for their voting power to get activated. However, this will only happen if voters vote to explicity remove the admittor for malicious activity which should be a very rare event. If the admittor gets replaced for benevolent reasons (eg term limits, retirement, etc), then the new members will have their voting power activated as soon as the original delay period passes.

### Enforcer decisions

Similarly it is important for enforcer decisions to take effect as soon as possible. If a bad actor is pestering the community, their ability to interact with the community must be stopped as soon as possible. Thus, Hosts will act on Enforcer decisions immediately and stop serving the content of banned actors.

However, the power to expel actors/content can also be abused by malicious Enforcers. An Enforcer can simply ban any member that is standing up to them and thus consolidate power and suppress dissent.

In order to prevent this, we enforce a similar delay period in which the voting power of a banned actor does not get deactivated until the delay period is met. Since the delay period is larger than the voting period, this gives time for the community to retaliate by replacing an Enforcer who is unjustly banning members while still ensuring that Enforcer decisions immediately take effect at the Community Interaction level.

If the Enforcer gets replaced for malicious activity, then all banned members will retain their voting power. The newly elected Enforcer must **explicitly** choose to remove their voting power as well, which again will only take affect after a new delay period has passed from that point. However, the banned members will remain unable to interact in the community unless the newly elected Enforcer **explicitly** unbans them, at which point they may start interacting again immediately.

Enforcers may also choose to unban members who have already passed the delay period. This will be like adding a new member. The previously banned member will be able to interact immediately, but must wait for the delay period to pass before their voting power gets activated.

Note: The delay period will allow bad actors to hold onto voting power after getting banned temporarily. They may even have voting power for an extended amount of time if the enforcer happened to be malicious as well. However, so long as the community has not reached the threshold number of bad actors with active voting power; the security model will hold and the bad actors will eventually be removed and disenfranchised.
Also just as in the Admittor case, Enforcers may be removed for non-malicious reasons. In this case, the banned actors will be disenfranchised as soon as the original delay period is reached.

### An Aside on the Delay Period

The delay period is a key part of the Fora protocol that prevents malicious authorities from taking over the system, while also allowing honest authorities to effectively and efficiently steward the community.

Its basic effect is to delay any changes to community governance at the blockchain level while immediately enacting any changes to community interaction at the ActivityPub level. This is because community governance acts as a final backstop against malicious authorities. If a malicious authority successfully takes over the governance of the community, then there is no protocol-level recovery from the situation. Thus any changes an authority wants to make to voting power on the blockchain must be delayed so that the community can protect against malicious attacks that seek to consolidate power in the hands of a malicious authority.

At the same time, we do not want to delay changes to community interaction because we want to give honest authorities effective and immediate tools to steward the community and enforce its rules. Note that if these changes were enacted by malicious authorities, they can be easily reversed by a new authority once the community chooses a replacement using voting power that cannot be easily usurped by the malicious party.

## Dishonest Hosts

So far, we've only been talking about honest Hosts who will monitor decisions made by Admittors and Enforcers and act accordingly; for example allowing new members to participate or refusing to serve bad content to end-users.

However, Hosts are not accountable to community governance and thus Fora does not assume that Hosts will be honestly following the protocol. So how does Fora intend to protect honest members from dishonest Hosts who refuse to comply with community decisions?

The following will illustrate scenarios in which a host can act dishonestly. While the solutions to these scenarios seem manual, light client code should always have a lists of hosts to connect to, similar to the options provided by a DNS query. This way, light clients can completely automate the detection and mitigation of dishonest hosts illustrated in the following scenarios away from the user providing a seamless experience.

**Scenario 1: Host ignores Admittor Decision**

First we can look at a Host who does not accept a new member admitted by the Admittors. Such a Host may refuse to accept messages coming from the new member and may refuse to serve queries about the new member from other members.

This is not an issue since the new member may simply choose a different Host to send their messages to, and if this Host is honest then they will gossip the message to all peers. Other members who are trying to get messages from the new member may simply direct their queries to other Hosts if they do not get a response from the dishonest Host. Note that a dishonest Host can only refuse to serve queries. They cannot maliciously forge messages since all messages are signed by the member.

So long as there are enough Honest nodes in the network such that no member gets completely eclipsed by Dishonest nodes, then all members will be able to send and receive messages as soon as they are admitted into the community.

**Scenario 2: Host ignores Enforcer Decision**

The more serious issue is when a Host ignores an Enforcer's Decision about banned members or banned content. Here a Host might choose to serve an honest member content that has already been banned by an enforcer. The honest member may then choose to report this content, only to find out that this content has already been banned by the Enforcer. The honest member then has a choice of switching to a different Host if they believe this was the cause of malicious hosting rather than a mere liveness issue.

**Honest Members Gravitate to Honest, Competent Hosts**

As noted above, in each case dishonest nodes can choose not to comply with community decisions; however honest members can always detect this and move to an honest host so long as they aren't completely eclipsed by dishonest hosts. (See eclipse attacks in decentralized networks [here](https://www.radixdlt.com/post/what-is-an-eclipse-attack/)). Thus, dishonest Hosts only threaten to be a temporary nuisance to honest members that rely on them and can no longer affect members once they've switched hosts.

Eventually, dishonest hosts will develop a reputation in the community and members will simply choose honest hosts that comply with community decisions. Of course, hosts must also be competent in order to keep track of the latest decisions recorded on the blockchain. Even the most reliable host will occassionally serve banned content because of synchrony issues in which a member gets served content before it gets deleted by the host. However, repeated and consistent failure to keep track of governance decisions will alienate members from incompetent hosts. Thus, honest members will eventually gravitate towards a trusted set of honest and competent hosts.

Thus even without assuming all Hosts are honest and baking in accountability in the protocol, we can accomplish Goal 6 in the list of [goals](../dogma/goals.md) since the community will be able to effectively enforce its rules by relying on honest Hosts to carry out the decisions made by community-appointed authorities.

Dishonest hosts may still serve dishonest members by continuing to serve banned content and banned members. However, they cannot interfere with the honest parts of the community since the Honest hosts will not accept any banned messages. In fact, persistently dishonest Hosts may simply be shunned from the gossip network if they consistently send banned messages to Honest peers. Of course, dishonest hosts can form their own network of banned members that send banned messages and interact in ways that are in violation of the original community rules, however one should note that this constitutes an **entirely separate** community altogether that cannot interfere with the interactions of the original community.

As noted in Non-Goal #3 in the [goals doc](../dogma/goals.md), this is not something Fora expects to make impossible. So long as the original community can effectively safeguard its own interaction space, other communities may form with different rulesets without any issues.

**Community Interaction Security Model**

As mentioned above, for an honest members to successfully connect to the rest of the honest community; they must not be eclipsed by dishonest Hosts. If an honest member has no choice but to access the community through a dishonest Host, then Fora can make no guarantee that the member can be protected from banned members and banned content.

Thus the Fora protocol assumes that the peer-to-peer network is sufficiently distributed and decentralized such that no member can be fully eclipsed by dishonest Hosts.
