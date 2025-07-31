# The Cardano Settlement (Ledger) layer

The Cardano Ledger layer is the component of the Cardano blockchain responsible for defining, maintaining, and updating the state of the distributed ledger. It acts as the authoritative source of truth for all transactions, balances, and state transitions on the network.

## Introduction

This layer specifies:
- **What the state of the ledger looks like**: A structure that contains information about **how funds in the system are distributed accross accounts**. This includes: account balances, how such balances should be adjusted when transactions and proposals are processed, the ADA currently held in the treasury reserve, list of stake pools operating the network, and so on.
- **How the ledger must be updated**: The ledger can be updated in response to certain events, such as: receiving a new transaction, crossing an epoch boundary, enacting a governance proposal, and so on. These updates must follow a set of **“transition rules”** (or just “rules”). For example, the *UTXOW transition rule* checks that, among other things, a given transaction is signed by the appropriate parties. These rules describe the different behaviors that determine how the whole system evolves and, taken together, they comprise **a full description of the ledger protocol**. 

Each transition rule consists of the following components:
• An **environment** consisting of data (read from the ledger state or the outside world) which should be considered constant for the purposes of the rule.
• An **initial state**, consisting of the subset of the full ledger state that is relevant to the rule and which the rule can update.
• A **signal or event**, with associated data, that the rule can receive or observe.
• A set of **preconditions** that must be met in order for the transition to be valid.
• A **new state** that results from the transition rule.

The transition rules can be **composed** in the sense that they may require other transition rules to hold as part of their preconditions. For example, the UTXOW rule mentioned above requires the *UTXO rule*, which checks that the inputs to the transaction exist, that the transaction is balanced, and several other conditions.

### Key Properties

- **Determinism:** The ledger is implemented as a set of pure functions, ensuring that the same inputs always produce the same outputs, independent of external factors.
- **Formal Specification:** The Cardano Ledger is formally specified in mathematical language (Agda), and the Haskell implementation is tested for conformance, providing high assurance of correctness.
- **Extensibility:** The ledger is designed to evolve across eras, supporting new features and protocol upgrades without compromising security or requiring disruptive hard forks.
- **Security:** It enforces cryptographic guarantees for transaction validity, asset ownership, and protocol integrity.
- **Separation of Concerns:** The ledger layer focuses solely on the rules for state transitions and value accounting.

## Advantages of using Haskell to implement the Ledger layer

- **The Ledger is, at its core, a pure function**: it validates and applies inputs (transactions or blocks) to a current state, producing a new state or rejecting the input with an error. For the protocol to function correctly, this process must be deterministic and reproducible - the same inputs must always yield the same outputs, without depending on external factors. Haskell supports this goal natively: **functions are pure by default**, and side effects must be handled explicitly via e.g. monads. **This makes Haskell a natural fit for implementing a pure and deterministic ledger**.

- **Illegal states unrepresentable**: Another advantage is that the Ledger is implemented as a **state machine** - with the ledger state evolving through validation rules that act as transitions. While state machines are universal computation models, Haskell offers unusually powerful tools for implementing them safely: **type-indexed state machines where illegal states and transitions are unrepresentable by construction** - providing a model that is safe and easy to reason about. 

- **Mathematical computations**: Also, substantial part of the Ledger involves mathematical computations, such as **value conservation and fee calculation**. We ensure their accuracy and precision - which are essential to the protocol - by leveraging Haskell's rich type system: **the operands are carefully represented with types that reflect their meaning and constraints**, so nonsensical operations are ruled out at compile time. Edge cases like integer overflow - a common source of problems - are explicitly modeled and rigorously handled. Moreover, by using Haskell's mathematical abstractions, like Monoid and Group, we can mirror algebraic reasoning in code - so we can write **generic, composable and correct implementations, grounded in well-understood mathematical laws** that are natural properties to test. 

- **Integrated formal specifications**: The code also mirrors the formal specifications for the Ledger. Not only does Haskell code map well to mathematical specifications, but we have **integrated the code with the Conway formal specification** written in Agda. This allows us to run conformance tests that verify that the validation rules in the Haskell implementation are applied **exactly as described in the formal specification**. This process has helped us find and fix a few issues and discrepancies. In the future it would be natural to take this to the next step and include formally verified code in production systems. 

- **DSLs**: Haskell is also famously useful for defining **domain-specific languages (DSLs)**, and we make great use of this. For example, we defined `ImpSpec` and `constrained-generators`, which allow us to easily write expressive, targeted tests that often double as executable documentation for various scenarios. 

- **Improving performance with evaluation strategies**: Another aspect of Haskell that this Ledger implementation relies on is its **fine-grained control over evaluation strategy**. This has a positive impact on **performance**, because it allows us to both avoid performing unnecessary computations and to enforce strictness where needed to preserve memory safety. 

- **Era abstraction**: Evolving the codebase across eras is a challenge: each era introduces new features, but much of the core logic stays the same or only slightly changes. We don't want to copy-paste the whole ledger for each era - that would be brittle, error-prone and quickly become unwieldy. Instead, **we use a combination of Haskell techniques to abstract over eras - such as: type classes, type families, composable rules, pattern synonyms**. These allow reuse of logic, while still enabling customization. This approach doesn’t just reduce duplication - it adds safety : **transactions from different eras have distinct type-level representation, making it impossible to mistakenly reference attributes and features that don't exist in a given era**.

- **Refactor with confidence**: Finally, Haskell gives us the invaluable ability to **refactor with confidence**. As visible in the Ledger’s Git history, the codebase has evolved significantly over time. With the benefit of hindsight, we are continually improving the code for clarity, safety, and performance. This is essential, because it allows us to build new features on clean and solid foundations, for the long term. **Without Haskell’s strong typing, many of these improvements would have been too risky to attempt, and we’d be stuck carrying technical debt indefinitely**. 

## References

- [Cardano Ledger Documentation](https://cardano-ledger.readthedocs.io/en/latest/)
- [Technology – Essential Cardano](https://www.essentialcardano.io/article/technology)
- [Designing in Layers – Cardano Settlement Layer](https://why.cardano.org/en/introduction/designing-in-layers/)
- [Formal Ledger Specifications (GitHub)](https://github.com/input-output-hk/formal-ledger-specifications) 
