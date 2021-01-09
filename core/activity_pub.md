# Activity Pub Compliance

This document specifies the extensions to Fora required for a community to be compliant with the [Activity Pub](https://www.w3.org/TR/activitypub/) specification. Without this extension, the Fora kernel will not be capable of interacting with other Activity Pub implementations.

## Introduction

"it's just activity pub with governance"

If we examine the Fora kernel through the lens of Activity Pub we find numerous similarities. Each member is a Client and each host is a Server, as defined by the Activity Pub specification. In fact, the only differences are a layer of governance settled on top of Activity Pub, along with additional properties between clients and servers, and servers and servers. The primary property added is the right to collective identification as a single community. Different servers with different sets of clients can now collectively agree to be identified as the same community thereby sharing their state and community membership.

The usefulness of this conceptual nesting is addressing the **right to individual exit** and the **right to collective exit**. 

The former can be relatively easily addressed in Activity Pub by using flexible URI's such as DID's. This would give any member the power to exit a community without losing its account information (content may have to be manually ported, but followers/following lists would automatically follow the user).

The latter requires the concept of communal ownership, which Activity Pub lacks. Suppose an Activity Pub implementation added support for DID's and thus fulfilled the requirements for the right to individual exit. In the presence of a malicious operator, we can reasonably predict that even in the case of an organized exit, the size and network effect of the forked community will be less than the original community. The right to collective exit and implicitly the right to partial ownership of a community helps mitigates the risk of malicious operators while aligning the incentives of operators, producers, moderators, and consumers.


## URI and DID

URIs (Uniform Resource Identifier) are used within Activity Pub to identify different servers and clients.  

Fora will use [decentralized identifiers](https://www.w3.org/TR/did-core/), to identify both members and communities. 

Please refer to the DID specification for further details, but it is essential to understand that:
- a DID is a URI (it can be a URL and it has properties such as Query, Fragment etc)
- a DID identifies a DID Subject, and that subject must never change
- a DID resolves to a DID Document which describes the DID subject
- a DID Document may be modified by one or more DID Controllers
- a DID Subject is not necessarily also the DID Controller
- there exists a verification method for each DID controller, which is specified in the DID Document
- there exists an authentication method to be associated with a DID Subject, which is specified in the DID Document
- All information may be modified in a DID Document except the id of the subject

The DIDs of a client must be recorded on a verifiable data registry, typically this will be a public blockchain or ledger. All communities will rely on client DIDs to resolve the authentication method used by a particular client (public key the client signs ActivityPub messages with). Communities may choose not to rely on DIDs for authentication of governance module interaction. If the communities choose to use on-chain DID verification, the client DID controllers must be verifiable via a communication protocol such as InterBlockchain Communication. It is recommended that governance modules support using a third verification relationship known as `voting authentication` to authenticate any votes casted by a DID controller on behalf of a DID subject. If the `voting authentication` verification relationship does not exist, governance modules should default to using the authroization method of DID controllers to verify casted votes. If communities do not want to require on-chain verifiable DIDs for each member, members must register with the governance module a public key that they will use to cast votes.

Client DIDs are intentionally isolated from the communities they are members of in order to allow for individual exit to different communities as well as improved operational security. DIDs allow for clients to rotate private/public key pairs without notifiying all the relevant parties.  The authentication method specified by a client DID must be a form of public/private key authentication.

Communities will also have DIDs that identify them to their members. This DID will also be recorded on a blockchain as a government. The DID Document for this DID will be deterministically generated based on the community parameters that exist at the block height the DID resolving function is being executed.  All governance decisions will be recorded on the DID Document. The DID controller will be the community members and the verification method to update the DID Document will be the passing of a governance proposal. The DID may include a default set of "seed" hosts to allow members to conveniently find and connect to the community which will be set under the service endpoint section of a DID Document. Additional parameters will be added to the DID Document which indicate a list of admittors, enforcers, members, governance decisions and any other relevant information. 

Note: There may be many valid, honest hosts that exist outside of this community-curated set.

## Public/Private Key Authentication

ActivityPub does not specify a particular authentication scheme, however most ActivityPub communities use OAuth2.0 or something similar which involves a traditional username/password scheme. Since all of the community state must be public and shareable between hosts, we cannot rely on username/password schemes or any scheme that gives servers privileged information on a client’s authentication info. Thus, clients must authenticate by proving ownership of a secret private key that corresponds to the public key that is publicly registered on a blockchain.

## Server

In addition to maintaining the full state of the community, servers will listen to events on the ActivityPub network to record and propagate new interactions. Servers must be connected to the appropriate blockchains or DID resolution services in order to track and verify the DID of members in the community.

## Limitations

Since any content posted to a Fora community must contend with the possibility of byzantine host behaviour, the Fora protocol cannot make any guarantees regarding private posts that are only readable to a subset of members. Thus, Fora will only support Public posts and direct messages. Public posts will be sent in plaintext and signed by the sender, while direct messages to members can be end-to-end encrypted given that all members have a Public/Private key as mentioned above (though a different pair would be used for DMs). This ensures that hosts have no ability to attack the confidentiality or integrity of member-sent messages. They can only refuse to relay the messages, though the message may simply get relayed through a different path.
