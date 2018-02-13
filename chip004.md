# Revised Block Headers

## Abstract
Blocks are required to commit to a tree of state changes. This replaces Bitcoin's transaction root in the block header. An additional blockheader field is created. The new field commits to all information validating those state changes. Block height must also be committed to in the header.

## Motivation

Adding additional information to the block header simplifies verification, and allows new SPV applications as well as future TXO-set optimizations. When re-examining these, we chose to logically separate them into state changes, and state change validations. Validation information can be safely discarded by most nodes after a period of time.

## Specification

Each block header MUST include two 32 byte fields: `hashStateChanges` and `hashValidations`. These fields represent the merkle root of trees committing to TXOs created and consumed, and the witnesses and ancillary information. The block header is constructed as follows:

```
[version][hashPrevBlock][hashStateChanges][hashValidations][time][blockHeight][difficultyBits][POS+POET]
```

The `hashStateChanges` tree is constructed by creating a Patricia Trie from the list of all inputs consumed by the block, and the list of transactions in the block. Each node includes additional information describing how many TXOs are created in transactions summarized by that node. As such, a proof in this tree can establish the index in the block of any TXO. Transactions are sorted lexically. Inputs are sorted to match transactions. As such there is a canonical structure for the tree.

The `hashValidations` tree is constructed by creating tuples of `(witnessInfo, witness)` for each input, in the same order they appear in the `hashStateChanges` tree. After the tuples corresponding to the inputs of a single transaction the aggregate signature for that transaction is included. This gives the tree a canonical order that intuitively matches the `hashStateChanges` tree.