---
title: "Distributed Hash Table (DHT)"
menu:
    guides:
        parent: concepts
---

## DHT

Distributed Hash Tables (DHT) are a distributed key-value store where key are cryptographic hashes. 

DHTs are usually too big to be stored entirely by each peer, so they are distributed: each "peer" (or "node") is responsible for a fraction of the DHT (redundantly). 
These subsets are called 'buckets', and map to some prefix of the hash. For exemple, a peer can be responsible for lookups for hash beginning by Qme8T33g...
More precisely, a peer maintains the DHT for hashes which share their prefix with its own PeerID (which is also a hash).

For example, the peer with PeerID "ABCDEF12345" can be responsible to maintain mapping for all hashes starting with the prefix "ABCD". 
Some hashes that would fall into this bucket would be ABCD38E56, ABCD09CBA or ABCD17ABB. 

The size of the buckets are directly related to the size of the prefix: the longer the prefix, the less hashes each nodes has to manage, the more nodes is needed.
Several nodes can be in charge of the same bucket if they have the same prefix.

In most DHTs' implementations, including Kademlia which is used by IPFS, the size of the buckets, or the size of the prefix, is dynamic. 
Buckets size growth and prefix size shortens when many nodes leaves the DHT, and vice versa. (or does it depends of the number of records and not the number of nodes? Is it at a bucket level or DHT level?)

Peers also keep connection to other peers so that they can forward requests if the requested hash is not in their own bucket.
If hashes are of lentgh n, they will keep n lists of peers: 
- the first list contains peers which ID have a different 1st bit.
- the second list contains peer which have their first bits indentical to its own, but a different second bit
- ...
- the m-th list contains peer which have their first m-1 bits identical, but a differnt m-th bit
- ...

The higher m, the harder it is to find peers which have the same ID up to m bits. The lists of "closest" peers typically remains empty.
"Close" here is defined as the XOR distance, so the longer the prefix they share, the closer they are.
List also have a maximum of entries k, otherwise the first lists would contain half the network, then a fourth, etc.

When a peer receives a lookup request, it will either answer with a value if it falls into its own bucket, or answer with the address of peer ID the closest to the requested hash.
If peer lists are well-filled, requests are forward to peers closer and closer to the requested hash, until a peer is finally able to answer it.
A request on a hash of length n will take log2(n) steps as a maximum. 

DHT's decentralisation provides additionnal advantages compared to a classic Key-Value store:
- *scalability* as each node only needs to maintain mapping for a fraction of the Key-Value pairs.
- *fault tolerance* via redunduncy, so that lookup are possible even if peers unexpectedly leave or join the DHT.
- *load balancing* as requests are made to different nodes and no unique peers process all the requests. Additionaly, any request can be addressed to any peer.

## The DHT of IPFS

In IPFS Kademlia's DHT, keys are not hashes but multihashes: a generalisation of the cryptographic hashes containing also information about which hashing function was used, and the length of the hash.

We use a DHT to lookup two types of objects (both represented by a multihash):
- Content IDs of the data added to IPFS. A lookup of this value will give the peerIDs of the peers having this content.
- PeerIDs of IPFS (libp2p?) nodes. A lookup will give all the multiaddresses to reach the peer(s) actually having the content.
Consequently, IPFS's DHT is use for content routing (1st lookup) and for peer routing (2nd lookup). 