# The Consensus Layer

## Introduction

The Cardano consensus layer has two important responsibilities:

1. **Running the blockchain consensus protocol**: In the context of a blockchain, consensus - that is, "majority of opinion" - means everyone involved in running the blockchain *agrees on what the one true chain is*. This means that the consensus layer is in charge of adopting blocks, choosing between competing chains if there are any, and deciding when to produce blocks of its own.

2. **Maintaining state for decision-making**: The consensus layer is responsible for *maintaining all the state required to make these decisions*. To decide whether or not to adopt a block, the protocol needs to validate that block with respect to the state of the ledger. If it decides to switch to a different chain (a different tine of a fork in the chain), it must keep enough history to be able to reconstruct the ledger state on that chain. To be able to produce blocks, it must maintain a mempool of transactions to be inserted into those blocks.

The consensus layer mediates between the network layer below it, which deals with concerns such as communication protocols and peer selection, and the ledger layer above it, which specifies what the state of the ledger looks like and how it must be updated with each new block. The consensus layer does not need to know the exact nature of the ledger state, or indeed the contents of the blocks (apart from some header fields required to run the consensus protocol).

## Key Properties

The consensus layer exhibits several key properties that make it robust and flexible:

### Protocol Independence

The consensus protocol is designed to be independent from a concrete choice of block, as well as a concrete choice of ledger, so that a single protocol can be run with different kinds of blocks and/or ledgers. Each of the three main responsibilities (leader check, chain selection, and block verification) defines its own 'view' on the data it requires.

The design maintains clear boundaries between different layers:
- **Network Layer**: Handles communication and peer management.
- **Consensus Layer**: Manages consensus protocol execution and state.
- **Ledger Layer**: Defines ledger structure, state, and update rules.

This separation allows each layer to evolve independently while maintaining clean interfaces between them.

### Three Core Responsibilities
1. **Leader Check**: Runs at every slot and determines if the node should produce a block. This may require information extracted from the ledger state (e.g., stake information for Ouroboros Praos).

2. **Chain Selection**: Refers to the process of choosing between two competing chains. The main criterion is typically chain length, but protocols may have additional requirements (e.g., preferring newer hot keys when cold keys are the same).

3. **Block Verification**: While most validation is a ledger concern, the consensus layer is responsible for validating header fields specific to the consensus protocol (e.g., cryptographic proofs for entropy derivation in Praos).

### Combinator Pattern
The consensus layer leverages combinators to build complex functionality from simpler components. For example:

- **Hard Fork Combinator**: Manages transitions between different ledger types at specific points.
- **Leader Schedule**: Allows explicit control of block production schedules for testing.
- **Dual Ledger**: Enables comparison between implementation and specification by running both simultaneously.


## Advantages of using Haskell to implement the Consensus layer

- **Strong Correctness Guarantees**: Haskell's strong type system and purity help in writing code that is less prone to errors, which is crucial for a system where network downtime or blockchain corruption is unacceptable. 

- **Testability**: Haskell's features, such as pure functions and referential transparency, make it easier to write unit tests and simulate various scenarios, including IO failures or different OS scheduling algorithms. We have an **executable specification for block header processing** which builds on the ledger specification and can be used to **conformance test a critical part of the consensus code**. Executable Formal Specifications of Ouroboros protocols naturally lend themselves to trace checkers for non-deterministic situations. 

- **Abstraction and Composability**: The Consensus layer is designed to be **highly abstract and composable**, allowing it to work with **different consensus algorithms and ledgers**. Haskell's support for higher-order functions, type classes, and algebraic data types facilitate the creation of such abstractions and composable code. We leveraged this aspect of Haskell when developing the Hard-Fork Combinator, one of our "crown jewels". 

- **Maintainability and Adaptability**: The Cardano node has evolved significantly, with changes in consensus algorithms and ledgers. **Haskell's modularity and abstraction capabilities** contribute to writing code that is **easier to maintain and adapt to these changes**. 

- **Efficient Data Handling with Persistent Data Structures**: Haskell's support for **persistent data structures** is advantageous for managing the ledger state. These structures, by avoiding unnecessary data duplication, are crucial for **efficiently maintaining multiple copies of the ledger state**. This efficiency is particularly important for enabling **fast rollbacks in the presence of forks**, as the Consensus layer might need to switch between different chains. 

- **Sharing**: Haskell’s **built-in sharing makes block trees natural and efficient to handle** reducing errors and providing good performance. 
