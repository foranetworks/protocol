# Fora Overview

Fora is a protocol for building self-hosted, self-regulating Internet communities built on top of ActivityPub and blockchain governance.

In the Fora sense, a **community** is a set of actors and a shared understanding of who belongs in the community, a shared understanding of what constitutes valid interaction within the community, a shared understanding of the community rules, and a shared understanding of who enforces the rules.

A **self-hosted** community is a community in which no member must necessarily rely on a third-party to access the community. In a self-hosted community, any member is able to host the state of the community themselves and communicate directly with other members to keep track of the latest updates.

A **self-regulating** community is a community in which the members themselves collectively decide on the rules and who will enforce them. These rules and their enforcers must also be changable at any time by the community. This allows communities to decide their own rules of conduct and appoint specific members to enforce these rules on behalf of the community. However, these enforcers cannot abuse their power without risking getting overthrown and replaced by the community.

Virtually all major internet platforms today do not fit the description above. Take Facebook as an example community. The Facebook community is the set of Facebook users who all understand that every Facebook user is also part of the Facebook community. The Facebook community interacts through Facebook posts, messages, comments, etc. The community rules are the Facebook content policy and Facebook, Inc is ultimately responsible for enforcing them. Thus Facebook is a community as defined above, yet it is not self-hosted since one must rely on Facebook servers to access the community. It is also not self-regulating since Facebook, Inc unilaterally sets the content policy and unilaterally enforces it, without any mechanism for Facebook users to have input on the rules or their enforcement. This is a problem since the incentives of the central operator can become grossly misaligned with the welfare of the community without an effective way for the community to respond.

A possible reaction to the current centralized Internet platforms might be to build a completely decentralized, censorship-resistant, anonymous social media platform. Such a platform certainly doesn't suffer from a central authority abusing its power, however it is not self-regulating since there is no way to enforce any rules on such a system. In fact it does not necessarily fit the definition of a community above. Such a system can easily devolve into chaotic, uncivil pandemonium of the sort we see in less-scrupulously moderated social media platforms already existing today.

Fora seeks to strike a balance between these two extremes by offering Internet communities the ability to enforce civil and moral norms without being beholden to the whims of a misaligned central operator.

For more information on why Fora is better than the status quo, see [dogma](./dogma).

Fora will use a blockchain to record governance decisions by the community. This includes votes on community rules and enforcers, as well as the unilateral decisions made by enforcers (e.g. new member acceptance, banning members/content). This ensures that this information is censorship-resistant and available to all community members.

The actual community interactions will be propogated through the community using [ActivityPub](https://www.w3.org/TR/activitypub/), although some modifications may need to be made in order to allow an ActivityPub community to form on a decentralized gossip network.

For more technical information on how the Fora protocol works, see [core](./core).
