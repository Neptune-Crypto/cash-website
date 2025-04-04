+++
title = "Minimizing Storage Using Merkle Mountain Ranges"
date = "2022-04-11"
weight = 0

[extra]
author = "Thorkil Værge"
+++

In the Neptune protocol Merkle Mountain Ranges are used to commit to the set of transaction outputs.

The [Neptune White Paper](/whitepaper) states that Merkle Mountain Ranges (MMRs) have two properties
that are useful in the context of a blockchain, namely:

1. With only the commitment to the MMR, a new MMR commitment can be calculated when a new leaf is appended.
2. With only the commitment to the MMR, a new MMR commitment can be calculated when an existing leaf is mutated, provided that you also have the authentication path for the updated leaf.

<!-- more -->

This article shows how this is done and dives into the runtime and storage costs of these operations.

## Introduction

The Merkle Mountain Range (MMR) data structure was invented by Peter Todd in 2012 in an attempt to reduce the hardware costs of
both running a Bitcoin full node and of running a competitive mining operation[^1]. The suggestion has so far been rejected since the Bitcoin development, for better or worse, resists change. It's a data structure that builds upon
the concept of a Merkle tree, which was invented by 1979 by Ralph Merkle.

Some of the features in the MMR structure are rather intricate,
but they build upon each other, such that each algorithm flows naturally from the previous one.

It all culminates in one beautiful function that combines
the entire arsenal we build up throughout this post. You'll find
much of the meat of the explanation in graphs and in the picture text to the graphs, so don't miss those.

An MMR is a list of Merkle trees. When we store only the total
leaf count in the MMR along with the list of peaks, the Merkle roots, we have a Merkle
Mountain Range Accumulator, an MMRA. You can think of MMRA as a commitment to all the leaves in the trees.

> When we store only the total
leaf count in the MMR along with the list of peaks, the Merkle roots, we have a Merkle
Mountain Range Accumulator, an MMRA.

In Rust, the definition of the MMRA struct could look like this

```rust
pub struct MmrAccumulator {
    leaf_count: u128,
    peaks: Vec<HashDigest>,
}
```

> In the Neptune protocol, the inputs to new transactions can be validated with just the MMRA. This
contributes massively to scalability, since storage cost is reduced to a few kilobytes. In contrast,
validating the inputs of new transactions in Bitcoin requires the user to download the entire Bitcoin history,
which is hundreds of gigabytes, something that cannot be achieved on a modern smartphone.

### Merkle Trees

A Merkle tree is a binary tree whose root is a commitment to a list of values. Membership
in this set can be proven by revealing a list of hashes and a value that put together
in the right order hash to the root. A major benefit of Merkle trees is that membership
can be proven with authentication paths whose size grows with $log(N)$ where $N$ is the
number of leaves in the tree. So we don't have to reveal the entire list of leaves to prove membership of an element in this set.

The hashes are calculated from the values that the tree commits to and then from
the bottom up by concatenating hashes of the two children of each node.

![**Merkle tree** The root of the Merkle tree is calculated from the bottom up. The "`|`" symbol indicates concatenation.](/learn/mmr/merkle-tree-simple.svg)
***Figure 1** The root of the Merkle tree is calculated from the bottom up. The "`|`" symbol indicates concatenation.*

![**Merkle tree** An authentication path consists of commitments to disjoint sets of increasingly bigger size. An authentication path can be validated if the Merkle root is known. To prove that the digest at position 11 has the stated value, the prover sends the hash preimage of 11 and the hash digests of the node indices 10, 4, and 3. Where 10 commits to 10's preimage; 4 commits to 8 and 9; and 3 commits to 12, 13, 14, and 15. This way, the authentication path is a commitment to all leaves in the Merkle tree.](/learn/mmr/merkle-tree-with-authentication-path.svg)
***Figure 2** An authentication path consists of commitments to disjoint sets of increasingly bigger size. An authentication path can be validated if the Merkle root is known. To prove that the digest at position 11 has the stated value, the prover sends the hash preimage of 11 and the hash digests of the node indices 10, 4, and 3. Where 10 commits to 10's preimage; 4 commits to 8 and 9; and 3 commits to 12, 13, 14, and 15. This way, the authentication path is a commitment to all leaves in the Merkle tree.*

Notice that each element in the authentication path is a commitment to a list of leaves, the
leaves in the subtree of the authentication path element. The union of these disjoint sets of leaves is the
entire list of leaves.

### Mutating Merkle Mountain Ranges
A Merkle Mountain Range can be viewed as a list of Merkle trees where the individual Merkle trees are combined when
two trees reach the same size. The individual Merkle trees in the MMR are combined by adding parent nodes to the trees' previous roots.

Figure 3 and Figure 4 illustrate how, after each node is inserted, the algorithm must
check if two trees can be merged. If they can, then a parent node is added to merge two Merkle tres. In MMR, the nodes are indexed by the
order in which they are added to the structure.

![**MMR** Adding node number 19 does not merge any existing Merkle trees but adds a peak (19) to the list of peaks. The grey circles are the old peaks, the green circle is the newly added peak. The newly added peak digest is simply the hash digest that was inserted with the append.](/learn/mmr/mmr-append-no-join.svg)
***Figure 3** Adding node number 19 does not merge any existing Merkle trees but adds a peak (19) to the list of peaks. The grey circles are the old peaks, the green circle is the newly added peak. The newly added peak digest is simply the hash digest that was inserted with the append.*

![**MMR** Adding node number 20 merges the tree with root in 19 and 20 to a tree with root in 21. Adding node 21 merges the trees with root in 21 and 18 to form the tree with root in 22. The grey circles are the old peaks, the green circle is the newly added peak.](/learn/mmr/mmr-append-with-join.svg)
***Figure 4** Adding node number 20 merges the tree with root in 19 and 20 to a tree with root in 21. Adding node 21 merges the trees with root in 21 and 18 to form the tree with root in 22. The grey circles are the old peaks, the green circle is the newly added peak. The hash digest for node 21 is calculated by concatenating and hashing node 19 and 20. The hash digest of node 22 (the new peak) is calculated by concatenating and hashing the digests of node 18 and 21.*

From both Figure 3 and Figure 4, we can see that the new list of peaks can be calculated from the old list
of peaks along with the newly inserted leaf[^2]. In other words: We do not need the full set of hash digests
in the tree to calculate the new peaks. The old peaks function as authentication paths (in reverse order)
for any new peaks that needs to be calculated. This illustrates claim 1 above: **A new MMRA formed from an append
operation can be calculated when knowing the newly inserted leaf and the previous MMRA.**

Claim number 2 states: **We can calculate a new MMRA that results from the mutation of a leaf knowing only: the previous MMRA, the new leaf value, its index, and its authentication path**. Using the authentication path for a leaf, we have a commitment
to all values but the leaf in question. When a leaf is mutated, we must traverse a path from the mutated leaf
to its peak to get the new hash digest of its peak. This is exactly what the authentication path allows us to do. The authentication path
always contains the sibling node of the node for which we are able to calculate a hash. Exactly one of the peaks
in the MMR will be mutated by this operation (assuming the new leaf is different from the old leaf).

This operation of finding the new MMR peak from a leaf mutation works for both Merkle Mountain Ranges and for
Merkle trees. Figure 5 shows this process for an MMRA, it shows how a new peak can be calculated from an
authentication path, a node index, and a new leaf value. Calculating a new peak after a leaf mutation is computationally equivalent
to verifying an authentication path since the hash digests are calculated in the path from the mutated leaf to the peak.

![Updating the leaf at position 8. The new peak (15) can be calculated from the value of the new leaf, its index, and its authentication path. The way to calculate this, is to first hash together the concatenation of the new leaf digest at index 8 with the first element of the authentication path at index 9, this gives us the new digest value for index 10. We continue this way, until we reach the peak to find the new value for this element in the list of peaks. The red numbers indicate which numbers are updated. The red circles indicate the elements of the authentication path.](/learn/mmr/mmr-leaf-mutation.svg)
**Figure 5** *Updating the leaf at position 8. The new peak (15) can be calculated from the value of the new leaf, its index, and its authentication path. The way to calculate this, is to first hash together the concatenation of the new leaf digest at index 8 with the first element of the authentication path at index 9, this gives us the new digest value for index 10. We continue this way, until we reach the peak to find the new value for this element in the peaks list. The red numbers indicate which numbers are updated. The red circles indicate the elements of the authentication path.*

## Keeping Authentication Paths Authentic
But hold on! If only the peaks and the leaf count are stored, and you just know authentication paths for a small subset of the leaves, won't these authentication paths become invalid as the tree is mutated
through either an append or a leaf mutation? Yes they will! So the authentication paths have to be
updated each time the MMR is updated this way. Therefore we add two functions that operate on
authentication paths, these are `update_from_append` and `update_from_leaf_update`.

### Leaf Mutation

`update_from_leaf_update` takes an authentication path `my_authentication_path` you want to update, the new leaf digest `new_leaf` after the mutation, its index, and *its*
authentication path `leaf_mutation_authentication_path`. So we have two authentication paths: `my_authentication_path` that we want to update and
`leaf_mutation_authentication_path` that was used in the process of updating an existing leaf in the MMRA. Calculating the hashes from the bottom up from the mutated leaf,
we can find the elements[^3] of `my_authentication_path` that need to be mutated. Figure 6 illustrates this process.

![Leaf 8 is mutated. We have an authentication path for leaf 5 that we want to remain valid after the mutation of 8. The authentication path for leaf 8 is marked by red circles, and our authentication path is marked with blue. When 8 is mutated, all the red numbers, (8, 10, 14, 15) are mutated. Since 14 is in our authentication path, this element in our authentication path must be mutated. ](/learn/mmr/mmr-leaf-mutation-ap-update.svg)
***Figure 6** Leaf 8 is mutated. We have an authentication path for leaf 5 that we want to remain valid after the mutation of 8. The authentication path for leaf 8 is marked by red circles, and our authentication path is marked with blue. When 8 is mutated, all the red numbers, (8, 10, 14, 15) are mutated. Since 14 is in our authentication path, this element in our authentication path must be mutated.*

### Appending
`update_from_append` takes the authentication path we want to update, an MMRA (peaks and leaf count prior to the append), and the digest
of the leaf that is appended. From the new leaf digest and the old peaks, a new peak can be calculated by traversing from the new leaf
to the new peak where the old peaks work as authentication paths to calculate hashes toward the new peak. Along this path are the hashes
that may need to be appended to the authentication path we are updating. Figure 7 illustrates this process.

![Leaf 20 is inserted. We want to update an authentication path for leaf 17 so it remains valid after this append is done. That is achieved by finding the indices of the nodes that form the new authentication path for 17 and appending those to its existing authentication path. The peaks prior to the append are marked with grey circles, the updated authentication path is marked with blue circles, and the node digests calculated from the append operation are colored green. Updating the authentication path with the `append` function adds the digest at 21 to the authentication path. The digest at 21 can be calculated from the append operation.](/learn/mmr/mmr-append-with-join-ap-update.svg)
***Figure 7** Leaf 20 is inserted. We want to update an authentication path for leaf 17 so it remains valid after this append is done. That is achieved by finding the indices of the nodes that form the new authentication path for 17 and appending those to its existing authentication path. The peaks prior to the append are marked with grey circles, the updated authentication path is marked with blue circles, and the node digests calculated from the append operation are colored green. Updating the authentication path with the `append` function adds the digest at 21 to the authentication path. The digest at 21 can be calculated from the append operation.*

### Optimizing Through Batching
As mentioned in the beginning, the Neptune Protocol represents the entire UTXO set in an MMR. Appends
and leaf mutations happen to the UTXO set whenever a new block is mined. So if a block contains 1000
transaction inputs and 1000 transaction outputs, and a client tracks the authentication paths for 1000
leaves, and the entire MMR contains 1bn leaves, the workload from running `update_from_leaf_update` and
`update_from_append` becomes significant.

Let $N$ be the leaf count in the MMR, $M$ the number of UTXOs whose authentication path is managed,
and $P$ be the number of inputs plus outputs in a block, then the number of digests that we need to
calculate whenever a block is mined is $\log(N) \cdot M \cdot P$. With the above values this is 60 million hashes.
Luckily this number can be reduced with several orders of magnitude.

Whenever a leaf is mutated or appended it is the **same** path through the MMR that gets new digest
values; the same set of hash digests throughout the tree that are updated. So we can first calculate
all the hashes that are derivable from an operation and then
update all the authentication paths with these calculated hash digest. This way the number of digest
that we need to calculate is only $\log(N) \cdot P$, or thirty thousand with the above parameters. The hash digests that
are derivable from an append are all the new peaks and the new leaf's path to its peak. The hash digests that are
derivable from a leaf mutation all lie on the path from the mutated leaf to its peak. The intersection of the
derivable digests and the authentication paths are the nodes whose digests are updated by a mutation of the MMR.

## Stringing it all Together
When a new block is found, the old peaks, the transaction inputs (represented as leaf mutations),
the transaction outputs (represented as appends) and the new peaks have to be validated to ensure
a valid state update[^4]. Notice that we don't have the peaks in between applying these updates,
so we have to start from the old peaks, apply **all** appends and leaf mutations and then validate
that we end up with the stated new peaks. It's easy enough to append leaves, as we just need to know
what the new leaf digests are, but the leaf mutations (transaction inputs) present a problem:
Each leaf mutation comes with an authentication path but this authentication path is provided by the client
spending the funds, and they cannot know about the other transactions in the block. So the spenders
can only provide the authentication path prior to this block being mined, and as we have seen above,
authentication paths have to be updated after **each** leaf mutation. Wouldn't this suggest that a block
can only contain one input, as any other leaf update could not be provided with a valid authentication
path?

Luckily we have a fix for that. The solution is to run batch-update on all remaining authentication
paths after each leaf mutation that is calculated in this batch validation function.

In high-level pseudo code this function can be written as follows:
```python
def verify_batch_update( old_peaks, old_leaf_count, new_peaks, appended_leaves,
      mutated_leaves, mutated_leaf_indices, mutated_leaf_authentication_paths ):

    if !mutated_leaf_indices.has_unique_elements():
        return False

    accumulated_peaks_value = old_peaks
    accumulated_peak_count = old_leaf_count

    # apply leaf mutations (tx inputs)
    while !mutated_leaf_indices.is_empty():
        new_leaf_digest = mutated_leaves.pop()
        authentication_path = mutated_leaf_authentication_paths.pop()
        accumulated_peaks_value = calculate_new_peaks_from_leaf_mutation(
            accumulated_peaks_value,
            new_leaf_digest,
            authentication_path,
            accumulated_peak_count,
        )

        # update the remaining authentication paths in
        # `mutated_leaf_authentication_paths`
        batch_update_from_leaf_mutation(
            mutated_leaf_authentication_paths,
            authentication_path,
            new_leaf_digest,
        )

    # apply appends (tx outputs)
    for appended_leaf in appended_leaves:
        accumulated_peaks_value = calculate_new_peaks_from_append(
            accumulated_peak_count,
            accumulated_peaks_value,
            appended_leaf,
        )

        accumulated_peak_count += 1

    return accumulated_peaks_value == new_peaks
```

This verifier calculates $inputCount \cdot 2 \cdot \log(N) + outputCount \cdot \log(N)$ digests.

Before verifying the cryptographic payload in a new transaction, the verifier *first* needs to establish that the coins that are being spent actually exist. Specifically, he needs to verify that the input UTXOs exist in the UTXO set, which can easily contain hundreds of billions of records. In the Neptune protocol, the inputs to new transactions can be validated with just the MMRA. This
contributes massively to scalability, since storage cost is reduced to a few kilobytes. In contrast,
validating the inputs of new transactions in Bitcoin requires the user to download the entire Bitcoin history,
which is hundreds of gigabytes, something that cannot be achieved on a modern smartphone.


[^1]: See [https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2016-May/012715.html](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2016-May/012715.html) for arguments
about why to add MMR to Bitcoin's consensus mechanism.

[^2]: We also need the leaf count to calculate the new peaks, since this is needed to know
when trees can be merged. 

[^3]: At most **one** element in the list of hash digests that constitute `my_authentication_path`
needs to be updated. This is because the elements of an authentication path are commitments to
disjoint sets of leaves. When we traverse from the leaf `new_leaf` to the root, we traverse through
nodes whose digests are commitments to sets of leaves that all include `new_leaf`. Since `my_authentication_path`,
like any authentication path, is a list of commitments to disjoint sets, only **one** element from
the digests we can derive from `leaf_mutation_authentication_path` and `new_leaf` can be part of the updated
`my_authentication_path`. If multiple elements from the list of digests calculated from
`leaf_mutation_authentication_path` and `new_leaf` were part of the updated `my_authentication_path`, then
the updated `my_authentication_path` would not consist of commitments to disjoint sets. This observation
means that we can save some hash calculations in `update_from_leaf_update` and break out of a loop early.
However, as we shall see in the section "Optimizing Through Batching", this optimization is less relevant
than it seems.

[^4]: This validation has to be run **somewhere** but it actually doesn't have to be run by the peers. Instead,
the miner can run this validation in a STARK engine and provide a proof of a correct update along with a proof
of the verification of the previous block. This way, a newly participating peer only has to verify the STARK
proof in the latest block to verify the entire history of the blockchain.
