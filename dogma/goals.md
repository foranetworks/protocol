# Fora: Goals and Non-Goals

**NOTE: This doc to state the intended goals of the Fora protocol and explicitly state the problems Fora is not intended to solve. All readers are encouraged to read this doc and evaluate the proposed protocol against the stated goals. Any comments that show how the protocol does not accomplish the intended goals or proposals for how to accomplish the intended goals in a better fashion are greatly appreciated. Comments intending to change the stated goals themselves are also welcome.**

## Goals

1. All code that is part of the Fora protocol must be FOSS (free and open source software).
2. Communities can decide for themselves how they will interact. (ie through text messages only, images only, text and images, etc).
3. Communities may decide who gets to be a part of the community, or they may choose to have a permissionless community (open enrollment).
4. Communities decide the rules by which individual members get voting power. This may either be determined statically (eg each member gets a single vote), or it may be determined dynamically (eg trusted authority awards variable voting power to members).
5. Members of the community can vote on the rules of the community and may change them anytime they wish, subject to a vote that passes the required threshold.
6. Communities must be able to efficiently and effectively enforce their rules by having the ability to ban content as well as members from their communities.
7. In order to efficiently and effectively enforce community rules, members of the community may appoint trusted members to specific roles that help steward the community (eg admitting new members, enforcing rules, etc). However, they may also replace these appointees at any time by voting them out and replacing with a new candidate. 
8. Any malicious authority must not be able to use their power to suppress the ability of the community to overthrow them. For example, a moderator should not be able to ban all members that vote against them before they can vote them out.
9. *A member who wishes to individually exit the community should be able to transfer their established identity to a different community.*
10. *Community governance should be a liquid democracy where members can delegate their voting power to other members who can vote on their behalf*

All italicized goals are low-priority and may not be accomplished on the first version of the protocol.

## Non-Goals

1. Anonymity is not a goal of the Fora network. While anonymity may be appropriate in many contexts, complete anonymity cannot coexist with a regulated community since there would be no way to ban bad actors and prevent them from re-entering the community.
2. Censorship-resistance is again not a goal of Fora since the goal of Fora is to create communities that are capable of regulating and thus censoring bad actors and bad content. While the Fora protocol cannot prevent bad actors from simply creating their own community to propogate their content, Fora does intend to create tools to prevent them from propogating on a community where their content is against the rules.
3. Complete prevention of bad actors is also not a goal of Fora. As mentioned in the previous point, there is nothing that can be done at the protocol-level that will prevent bad actors from simply running their own community with a bad set of rules. The protocol itself is neutral with respect to the type of rules it enforces, and thus any free open-source content distribution protocol cannot prevent bad actors from using it (a governance protocol is as good as the people who run it). Thus Fora does not try to accomplish an impossible goal. Instead, the main goal of Fora is to give communities the tools to efficiently and effectively regulate their spaces as they see fit so that bad actors and bad content cannot pester good communities. (Note: The definition of "good" and "bad" is left intentionally vague since they are entirely subjective. Though the general point holds for any coherent definition of both.)