
The on-chain entity used to describe Fora entities will be as follows:

```
unique DID => Fora object
```

The Fora object is simply a document with a map from arbitrary string to a DID along with the list of DIDs that can update the mapping and their associated authentication mechanisms

```
fora:metadata => DID (memberDID: majority)
fora:admittor:1/3totalvotingpower => DID (admittorDID, memberDID: majority)
...
```

Fora admittor
```
members:400voteseach => [MemberDID, MemberDID, MemberDID...]
members:200voteseach => [MemberDID, MemberDID, MemberDID...]
...
```


