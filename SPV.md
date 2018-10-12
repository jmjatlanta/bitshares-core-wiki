My thoughts on SPV (Simple Payment Verification) which basically provides a way to validate a transaction is part of a black provided only a subset of blocks without knowledge of the entire blockchain.

## Proof of work

Provided that Proof of work does not come with finality (but rather probabilistic finality - e.g. 99.99% after 6 blocks), SPV on POW also is probabilistic. This makes it particular easy because all we need to do is validate that a particular transaction is part of a block (through merkle root proofs) and collect 6 subsequent blocks. From those, the required *work* can be identified from the *difficulty* miners had to resolve to find those blocks. Hence, we basically end up with an equation that connections electrical expense with probability of a transaction being part of a block. Parameters allow to fine-tune for required security.

## Delegated Proof-of-Stake

The difficulty here lies in the fact that no connection between electrical expense and validity of a block can be made. Instead, some (trusted) block producer signs a block with a secret private key. This brings us to two problems:

* How do I find out that a block producers is trusted?
* How do I ensure that the block producer has a certain key?

However, we do have an advantage in DPOS and that is **finality** that comes as early as 2/3 of the block producers individually approved a block (e.g. have built a block ontop of a chain containing that block).

### Rounds and Maintenance Intervals

It is important to note that block producers are a fixed set of entities that have been granted permission to produce blocks in a a-priori known order. In each **round** a block producer may only produce at most one block. Hence, if a block is supposed to be generated every 3 seconds, and a round has 20 witnesses, the round ends after exactly 60 seconds. Afterwards, a new round is shuffled from this set of block producers.

Every maintenance interval, the votes for block producers are tallies which might cause block producers to change. Additional producers may become active, active ones may become inactive, or active ones may be replaced.

## Problem Statement

All this is dealt with by DPOS internally, as part of the consensus protocol. If we were to verify the validity of a particular block, the only way to do so would be to go through every single block since genesis as we cannot otherwise ensure that the block producers have actually been voted the way it is claimed.

At the current state, a single block simply does not contain sufficient *information* to ensure provability of that block being part of the actual (BitShares) blockchain. The only solution to that, obviously, requires to provide **additional** information together with the block/transaction in question. Obviously, we are looking for a way that requires less than the entire blockchain to be provided to prove a single transaction/block.

## Proposal

### Step 1 - Announcing block producer

Provided that we need to be able to identify the block producers and their signing keys, we propose to:

* announce changes of block producers and signing keys in each first-block-in-round after the **maintenance interval*

This allows for keeping track of changes to block producers by providing only one block of information every 24hrs. It thus becomes trivial to keep track of block producers since genesis (as of writing, only `1095` have happened so far).

### Step 2 - BFT commitments of block producers per round

Provided that the order of block producers changes every round, we cannot verify that a block producer may have gone rough and provided a proof of a spoof block. For this reason, we introduce:

* a BFT-style operation that has to be broadcast by the block producer and can only be included in the **first-block-in-round** as a commitment of the producer to the first-block-in-round in the **previous round**.

This way, we can *count* how many block producers agree on the outcome of the **last** round. If this number is higher than 2/3 of the available block producers (which is a number we know because of Step 1), we can assume that the **previous round** has also reached irreversibility.

### Required data for a proof

Hence, for a proof that a transaction `tx` is part of a block `B(tx)`, we need the following information:

* the block header of `B(tx)` that contains the merkle root for proofing `tx in B(tx)`
* the previous `x` block headers up to the **first-block-in-round* of the round containing `B(tx)`
* the previous `y` **first-block-in-round* headers up to genesis





# Open Questions

* How to allow a block producer to change signing key?
* Can we make irreversible explicit if the witnesses commit to the first-block-in-round of the **previous round**?