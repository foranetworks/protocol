### User DIDs

DID: `did:example:123`
```
"id": did:example:123

"controller": did:example:123 (typically the same as the id field, but not necessarily)

"verification_methods": {[
    {
        "id": did:example:123#PubKey1,
        "type": "Ed25519VerificationKey2018"
        "controller": did:example:123,
        "publicKeyBase58": "H3C2AVvLMv6gmMNam3uVAjZpfkcJCwDwnZn6z3wXmqPV",
    },
    {
        "id":  did:example:123#PubKey2,
        "type": "JsonWebKey2020"
        "controller": did:example:123,
        "publicKeyJwk": {
            "crv": "P-256",
            "x": "38M1FDts7Oea7urmseiugGW7tWc3mLpJh6rKe7xINZ8",
            "y": "nDQW6XZ7b_u2Sy9slofYLlG03sOEoug3I0aAPQ0exs4",
            "kty": "EC",
            "kid": "_TKzHv2jFIyvdTGF1Dsgwngfdg3SH6TpDv0Ta1aOEkw"
        }
    },
    {
        "id": did:example:123#BlockchainAccount,
        "type": "CosmosAccount",
        "controller": did:example:123,
        "blockchainAccountId": "cosmos5823...",
    },
    ...
]}

// This verification relationship is used for authenticating ActivityPub messages.
"authentication": did:example:123#PubKey1, ...

// This verification method can be used to negotiate keys for E2E communication with the DID subject.
// NOTE: OPTIONAL
"keyAgreement": did:example:123#PubKey2, ...

// This verification method is used to authenticate governance votes and for updating the DID doc
// Q: Perhaps we should create a new verification relationship for governance???
"capabilityInvocation": did:example:123#BlockchainAccount
```

### Module DIDs

Fora communities can also be represented with a DID. However, Fora communities do not have simple public-key authentication schemes. There are many actors that should be able to update a DID, from admittors who should be able to add new members, to enforcers who should be able to remove members, and community governance which should be able to arbitrarily update the DID doc. However, this is too much complex, application-specific logic to encode into a DID method. Thus, we propose a new verification method that will allow any application to define their own means of updating a DID document.

The Module Verification method will allow modules themselves to be controllers of a DID. This will require that Modules themselves have a DID associated with them and a means of authenticating that updates to the DID document are originating from the corresponding module. In the case where the DID module and the controller module are on the same blockchain, this will require some form of object capability system that ensures the update is coming from the appropriate module. In the inter-blockchain case, update-DID packets coming from the appropriate channel are proof in and of themselves that the packet is coming from the channel counterparty module.

Module DID:

DID: `did:module-dids:cosmos-hub:fora`
```
id: did:module-dids:cosmos-hub:fora

controller: did:module-dids:cosmos-hub:fora (controller may be a different module)

verification-method: {[
    {
        "id": did:module-dids:cosmos-hub:fora#SameChainAuth,
        "type": ObjectCapability,
        "controller": did:module-dids:cosmos-hub:fora,
        "capName": "capability/did/fora",
    },
    {
        "id": did:module-dids:cosmos-hub:fora#InterchainAuth,
        "type": IBCPacket,
        "controller": did:module-dids:cosmos-hub:ibc,
        "channelName": "channel1/portFora",
    },
    ...
]
}

// Used for application-specific authentication
authentication: did:module-dids:cosmos-hub:fora#SameChainAuth,

// Used for updating the DID document
capability-invocation: did:module-dids:cosmos-hub:fora#SameChainAuth,

// Used to update the capability invocation value
capability-delegation:
did:module-dids:cosmos-hub:governance#SameChainAuth
```


**Module Verification Method:**

The Module Verification Method uses an object capability model for verification.

CRUD operations for Module-owned DID:

**Create:**

The DID method MUST generate a globally unique DID that includes the module name in the DID. If necessary, the Create operation must also return a capability that allows the calling module to authorize future interactions with the DID document.

`Create(controller Optional{String}) => (DID string, cap Optional{Capability}, Error)`

A controller may be passed in if the subject of the DID is not the same as the controller. If no controller is specified, the DID is initialized with the DID subject as the DID controller.

Example DID's:

`did:cosmos-hub:fora`

`did:ethereum:0x583...` (0x583 is the smart contract address that owns this DID)

**Read:**

Anyone MUST be able to resolve the DID document given the DID method and the DID. An error should be returned if the DID does not exist.

`Read(DID string) => (DID document, Error)

**Update:**

The DID method MUST authorize that any updates to the DID document comes from the controlling module. This should be done using a capability system or an equivalent scheme. In the cosmos case, this will require passing in the dynamic capability provided during the `Create` operation. In the ethereum case, the DID smart contract simply checks that the sender of the cross-contract call is the controller of the DID.

The DID module must provide the following methods to update the DID document.

`Set(DID string, key string, value Object, cap Optional{Capability}) => Error`

`Delete(DID string, key string, cap Optional{Capability}) => Error`

The key is the property in the document that is to be updated. Nested properties should be able to be specified by using path delimiters in the string. An error should be returned if the caller is not authorized to make the update or if the DID does not exist.

**Delete**

A DID method MAY choose to allow controllers to delete the DID. If this is allowed, then the DID method MUST authenticate that the controller has sent the DELETE request. An Error must be returned if the DID does not exist or if the caller is not authorized to delete the DID or if DELETE is not a supported operation for this DID method.

`Delete(DID string, cap Optional{Capability}) => Error`

### Community DIDs

Now that modules themselves can be registered as DID's, we can use the Fora module to be the controller of a Fora community DID.

Format:

DID: `did:module-dids:cosmos-hub:fora:communityA`
```
id: did:module-dids:cosmos-hub:fora:communityA

// The Fora module is the ultimate controller of the DID document
// It will use the verification methods below to perform application-specific authentication and make the necessary changes to the DID document.
controller: did:module-dids:cosmos-hub:fora

verification-methods: {[
    // All Fora communities MUST contain the members verification method
    // And MAY contain other verification methods representing different entities in the community
    {
        "id": "did:module-dids:cosmos-hub:fora:communityA#members,
        "controller": did:module-dids:cosmos-hub:fora,
        "type": {Community-Specific-Member-Governance-type},
        "members": {[
            {
                // required fields
                member: {memberDID},
                status: {Enum{active | banned | revoked}},
                statusTimestamp: {timestamp at which status was updated}
                // there may be additional fields to assist with community governance
                ...
            },
            ...
        ]} 
    },
    {
        "id": "did:module-dids:cosmos-hub:fora:communityA#exampleEntity,
        "controller": did:module-dids:cosmos-hub:fora,
        "type": {application-specific-verification-type},
        // include verification specific fields as defined by application specific verification type
        ...
    },
    ...
]}

// The entity described in this verification method is capable of making arbitrary changes to the DID document.
// The entity will call on the Fora module to make changes. The Fora module will use the associated verification method to authenticate changes from the entity before making any changes
authentication: {verificationMethod}

services: {[
    {
        // If the admittorDID authenticates an `admitMember` action,
        // the Fora module will add the member to the member list as ACTIVE.
        id: "did:module-dids:cosmos-hub:fora:communityA#service1",
        type: ForaAdmitService,
        serviceEndpoint: "did:module-dids:cosmos-hub:fora:communityA/admitMember",
        authentication: {admittorDID}...,
    },
    {
        // If the updaterDID authenticates an `admitMember` action,
        // the Fora module will add the member to the member list as ACTIVE.
        id: "did:module-dids:cosmos-hub:fora:communityA#service1",
        type: ForaAdmitService,
        serviceEndpoint: "did:module-dids:cosmos-hub:fora:communityA/admitMember",
        authentication: {updaterDID}...,
    },
    {
        // If the revokerDID authenticates a `revokeMember` action,
        // the Fora module will update a new member to REVOKED.
        id: "did:module-dids:cosmos-hub:fora:communityA#service1",
        type: ForaAdmitService,
        serviceEndpoint: "did:module-dids:cosmos-hub:fora:communityA/admitMember",
        authentication: {revokerDID}...,
    },
    {
        // If the endforcerDID authenticates a `banMember` action,
        // the Fora module will update the member to BANNED.
        id: "did:module-dids:cosmos-hub:fora:communityA#service2",
        type: ForaBanService,
        serviceEndpoint: "did:module-dids:cosmos-hub:fora:communityA/banMember",
        authentication: {enforcerDID}...,
    },
    {
        // This endpoint will return a list of default hosts.
        // If operator DID authenticates, then the list of hosts will change.
        id: "did:module-dids:cosmos-hub:fora:communityA#service3",
        type: ForaHostDiscoveryService
        serviceEndpoint:  "did:module-dids:cosmos-hub:fora:communityA/defaultHosts",
        hosts: {hostURI}, ...
        // anyone may access this endpoint, but they must authenticate to change the hosts value
        authentication: {operatorDID}...
    },
    {
        // This endpoint will authorize metadata updates if the authority authenticates
        id: "did:module-dids:cosmos-hub:fora:communityA#service4",
        type: ForaUpdateMetadataService,
        serviceEndpoint: "did:module-dids:cosmos-hub:fora:communityA/updateMetadata",
        authentication: {metadataControllerDID}...
    }
]}
```

**Fora CRUD**:

**Create:**

Any user can create a community by sending a `CreateCommunityMsg` to the Fora module.

`CreateCommunityMsg(initialDIDDoc Object, initialMetaData Object) => (DID string, Error)`

The Fora module will call `Create` on the DID module. If successful, it will set all of the properties specified in `initialDIDDoc`. It will also attempt to set all the `initialMetaData` in its own store. Any error along the way will abort the transaction and return.

The initialDIDDoc MUST contain an initial `members` verification method, and it MUST contain an initial `authentication` relationship.

**Read:**

Any user should be able to resolve the DID to the DID Document using the DID method and DID module.

The user should also be able to easily retrieve all the metadata stored in Fora module regarding a particular community DID.

**Update:**

Users may try to trigger updates to the community DID document by sending the Fora module update requests. It is the responsibility of the Fora module to read the DID document and verify that the update has been authorized by the relevant entity using the verification methods, relationships, and services specified in the DID document.

The `authentication` relationship in the DID document can make arbitrary changes to the DID document. The Fora module will enact any changes that are authorized by the `authentication` relationship.

The other narrow permissions are specified in the `services` section that all Fora community DIDs must specify.

`admitMember`:

The `admitMember` service will allow the specified authority to admit a new member into the `members` verification method.

The authority submits an `AdmitMember` request to Fora module. The Fora module will look up the verification method of the authority using the DID doc and verify that the request is properly authorized. If it is, then Fora module will add the new member. It will error if the member struct is not properly constructed or if the request is not correctly authorized.

All other services are authorized and executed in similar ways.

Required Services:

`admitMember`, `banMember`, `updateMetadata`

Optional Services:

`revokeMember`, `defaultHosts`

NOTE: Future versions of the Fora protocol may require or enable new services.

**Delete**

The Fora module MAY or MAY NOT enable deletion of communities. If so, the Fora module MUST allow the `authentication` relationship to delete the community and MAY allow a `delete` service that allows some delegated authority to delete the community DID.



### Examples

DID: `did:module-dids:cosmos-hub:fora:communityA`
```
id: "did:module-dids:cosmos-hub:fora:communityA"

// The controller of the communityA DID is the Fora module
controller: "did:module-dids:cosmos-hub:fora"

verification-methods: {[
    {
        "id": "did:module-dids:cosmos-hub:fora:communityA#admittors",
        "controller": "did:module-dids:cosmos-hub:fora#members",
        "type": "forav1:KofNMultiSig"
        "total": 5,
        "threshold": 3,
        "admittors": {
            // NOTE: Admittor DIDs stay static and indirectly point to the DID of the current admittor
            {
                admittor: did:module-dids:cosmos-hub:fora:communityA:admittor1,
                power: 0.2,
            },
            {
                admittor: did:module-dids:cosmos-hub:fora:communityA:admittor2,
                power: 0.2,
            }
            ...
        }
    },
    {
        "id": "did:module-dids:cosmos-hub:fora:communityA#enforcers",
        "controller": "did:module-dids:cosmos-hub:fora#members",
        "type": "forav1:KofNMultiSig"
        "total": 5,
        "threshold": 1,
        "enforcers": {
            // NOTE: Enforcer DIDs stay static and indirectly point to the DID of the current enforcer
            did:module-dids:cosmos-hub:fora:communityA:enforcer1,
            did:module-dids:cosmos-hub:fora:communityA:enforcer2,
            ...
        }
    },
    {
        "id": "did:module-dids:cosmos-hub:fora:communityA#members",
        "controller": "did:module-dids:cosmos-hub:fora",
        "type": "forav1:WeightedForaVote"
        "delayPeriod": 4weeks,
        "participationThreshold": 0.8,
        "yesThreshold": 0.5,
        "members: {
            {
                member: "did:cosmos-hub:userdid321",
                power: 10,
                status: active,
                statusTimestamp: 0:00,Jan1,2021,
                authority: admittor1,
            },
            {
                member: "did:cosmos-hub:userdid123",
                power: 15,
                status: active,
                statusTimestamp: 0:00,Jan1,2019,
                authority: admittor4,
            },
            {
                member: "did:cosmos-hub:userdid987",
                power: 3,
                status: banned,
                statusTimestamp: 23:59,Dec32,2020,
                authority: enforcer3,
            },
            ...
        },
    },
    // All the indidual admittors and enforcers must point to their current authority.
    {
        id: "did:module-dids:cosmos-hub:fora:communityA#admittor1,
        "type": "UserDID"
        "controller": did:module-dids:cosmos-hub:fora:communityA#members,
        authority: did:example:cosmos-hub:user123,
    },
    ...
]}

services: {[
    {
        // If the admittors authenticate an `admitMember` action,
        // the Fora module will add the member to the member list as ACTIVE.
        id: "did:module-dids:cosmos-hub:fora:communityA#service1",
        type: ForaAdmitService,
        serviceEndpoint: "did:module-dids:cosmos-hub:fora:communityA/admitMember",
        authentication: did:module-dids: cosmos-hub:fora:communityA#admittors,
    },
    {
        // If the admittors authenticate an `updateMember` action,
        // the Fora module will update the member in the member list.
        id: "did:module-dids:cosmos-hub:fora:communityA#service1",
        type: ForaUpdateService,
        serviceEndpoint: "did:module-dids:cosmos-hub:fora:communityA/updateMember",
        authentication: did:module-dids: cosmos-hub:fora:communityA#admittors,
    },
    {
        // If the admittors authenticate a `revokeMember` action,
        // the Fora module will update a new member to REVOKED.
        id: "did:module-dids:cosmos-hub:fora:communityA#service1",
        type: ForaRevokeService,
        serviceEndpoint: "did:module-dids:cosmos-hub:fora:communityA/admitMember",
        authentication: did:module-dids: cosmos-hub:fora:communityA#admittors,

    },
    {
        // If the endforcers authenticate a `banMember` action,
        // the Fora module will update the member to BANNED.
        id: "did:module-dids:cosmos-hub:fora:communityA#service2",
        type: ForaBanService,
        serviceEndpoint: "did:module-dids:cosmos-hub:fora:communityA/banMember",
        authentication: did:module-dids: cosmos-hub:fora:communityA#enforcers,
    },
    {
        // This endpoint will return a list of default hosts.
        // If admittors authenticate, then the list of hosts will change.
        id: "did:module-dids:cosmos-hub:fora:communityA#service3",
        type: ForaHostDiscoveryService
        serviceEndpoint:  "did:module-dids:cosmos-hub:fora:communityA/defaultHosts",
        hosts: {hostURI}, ...
        // anyone may access this endpoint, but they must authenticate to change the hosts value
        authentication: did:module-dids: cosmos-hub:fora:communityA#admittors,
    },
    {
        // This endpoint will authorize metadata updates if the authority authenticates
        // The members must collectively authenticate via a governance vote to update community metadata
        id: "did:module-dids:cosmos-hub:fora:communityA#service4",
        type: ForaUpdateMetadataService,
        serviceEndpoint: "did:module-dids:cosmos-hub:fora:communityA/updateMetadata",
        authentication: did:module-dids: cosmos-hub:fora:communityA#members,
    }
]}
```