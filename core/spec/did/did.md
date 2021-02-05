# Decentralized Identifiers

### Introduction

Fora will use [Decentralized Identifiers](https://www.w3.org/TR/did-core/) (or DIDs) to represent the identities of different entities in the Fora protocol. These DIDs will define how the entity will authenticate its interaction within the community (how to authenticate ActivityPub messages coming from the entity), and define how the entity will authenticate its votes on community governance (how to authenticate blockchain governance messages).

The DID must also provide a service endpoint for modifying the DID document. This service endpoint may use the DID document to authenticate any modification request and the authentication logic used by the service endpoint should be specified in the DID document itself.

Fora uses decentralized identifiers since this allows Fora entities to retain identities that can be maintained independent of the communities they belong to. This enables the individual exit necessary to satisfy Fora's [goals](../../../dogma/goals.md) since users can retain their identity and accumulated reputation even if they leave a community. Communities in turn can use the Fora metadata (community membership and bans) associated with a particular DID to determine admission and membership status. Decentralized identifiers allow users to fully control their own Fora identities while also allowing other entities in the Fora ecosystem to update and evaluate the reputations of any Fora entity.

There may be many different implementations of DIDs and many different DID provisioners that Fora entities may want to use. Thus, the Fora protocol must support multiple DID implementations and multiple DID provisioners. This document specifies the requirements that DID implementations must satisfy in order to interoperate with the Fora protocol.

### DID Specification

DIDs are URI compliant, thus the DID must start with the particular scheme it is implementing. All Fora-compliant DID schemes must include the provisioner of the DID in its DID format.

Examples: `foradid:cosmoshub:userA` or `iid:ethereum:userB`

The provisioner **must** be able to provide a proof for the (DID, DID document) pair upon request.

### Verification Method

DID implementations must allow users to define aribitrary, self-describing verification methods in order to describe complex Fora entities like communities. Simple Fora entities, such as end users, may rely on DID implementations that only support public key verification. However, Fora communities will also be represented as a DID where the verification method may be a governance vote on the blockchain. Thus, the DID implementations Fora communities use must support arbitrary verification methods, in order to support arbitrary community governance methods.




