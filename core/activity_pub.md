# Activity Pub Compliance

This document specifies the extensions to Fora required for a community to be compliant with the [Activity Pub](https://www.w3.org/TR/activitypub/) specification. Without this extension, the Fora kernel will not be capable of interacting with other Activity Pub implementations.

## Introduction

"it's just activity pub with governance"

If we examine the Fora kernel through the lens of Activity Pub we find numerous similarities. Each user is a Client and each host is a Server, as defined by the Activity Pub specification. In fact, the only differences are a layer of governance settled on top of Activity Pub, along with additional properties between clients and servers, and servers and servers. The primary property added is the right to collective identification as a single community. Different servers with different sets of users can now collectively agree to be identified as the same community thereby sharing their state and community membership.

The usefulness of this conceptual nesting is addressing the **right to individual exit** and the **right to collective exit**. 

The former can be relatively easily addressed in Activity Pub by using flexible URI's such as DID's. This would give any user power to exit a community without losing it's account information (content may be lost, but followers/following lists would not).

The latter requires the concept of communal ownership, which Activity Pub lacks. Suppose an Activity Pub implementation added support for DID's and thus fulfilled the requirements for the right to individual exit. In the presence of a malicious operator, we can reasonably predict that even in the case of an organized exit, the size and network effect of the forked community will be less than the original community. The right to collective exit and implicitly the right to partial ownership of a community helps mitigates the risk of malicious operators while aligning the incentives of operators, producers, moderators, and consumers.


## Public/Private Key Authentication

ActivityPub does not specify a specific authentication scheme, however most ActivityPub communities use OAuth2.0 or something similar which involves a traditional username/password scheme. Since all of the community state must be public and shareable between hosts, we cannot rely on username/password schemes or any scheme that gives hosts privileged information on a user’s authentication info. Thus, users must authenticate by proving ownership of a secret private key that corresponds to the public key that is publicly registered on a blockchain.

## URI and DID

URIs (Uniform Resource Identifier) are used within Activity Pub to identify different servers. 

Fora will use [decentralized identifiers](https://www.w3.org/TR/did-core/), to identify both users and platforms. These DIDs must be registered on a blockchain that the platform can query identities from. Users will have a DID so they can self-attest to their identity on the blockchain and then use that DID in any Fora community they wish. They can also retain this identity even as they move to different communities. The DID document for each user must contain the current public key they will use sign messages sent from their Fora user account. 

Fora implementations which rely on the public key registered with a DID instead of registering a public key with a governance module, will benefit from dynamic swapping of public keys. 

Meanwhile communities will also have DIDs that identify them to their users. This DID will also be registered on the blockchain as a government. The DID will resolve to the current hosts of the platform. Any governance decisions which affects the set of hosts will update the DID resolving result to output the new set of hosts.

## Server

In addition to maintaining the full state of the community, servers will listen to events on the ActivityPub network to record new interactions. Servers must be connected to the appropriate blockchains or DID resolution services in order to track and verify the DID of users in the community.

## Limitations

Since any content posted to a Fora community must contend with the possibility of byzantine host behaviour, the Fora protocol cannot make any guarantees regarding private posts that are only readable to a subset of users. Thus, Fora will only support Public posts and direct messages. Public posts will be sent in plaintext and signed by the sender, while direct messages to users can be end-to-end encrypted given that all users have a Public/Private key as mentioned above (though a different pair would be used for DMs). This ensures that hosts have no ability to attack the confidentiality or integrity of user-sent messages. They can only refuse to relay the messages, though this is grounds for enforcers to remove a host.


