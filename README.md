# Powers of Tau Specification
These documents comprise a specification for Ethereum's Powers of Tau (PoT) setup for use in [KZG commitments](https://dankradfeist.de/ethereum/2020/06/16/kate-polynomial-commitments.html) in [EIP-4844 (Proto-Danksharding)](https://eips.ethereum.org/EIPS/eip-4844) and ultimately [Full Danksharding](https://notes.ethereum.org/@dankrad/new_sharding).

The ceremony takes place between _participants_ and the _sequencer_.  _Participants_ are the entities that contribute their secret randomness to the final output $\tau$ s. The role of the _sequencer_ is to act as the common point of interaction for all participants as well as verifying participants' contribution as the ceremony progresses.

## Table of contents
* [10,000 ft Overview](#10000-ft-overview)
* Cryptography
    * [BLS](./docs/cryptography/BLS.md)
    * [SDK](./docs/cryptography/sdk.md)
* Contributions
    * [Object Definitions](./docs/contribution/contribution.md)
* Sequencer
    * [Requirements](./docs/sequencer/sequencer.md)
    * [Contributor Qualification](./docs/sequencer/contributorQualification.md)
    * [Contribution Queuing](./docs/sequencer/queueStrategy.md)
    * [API Overview](./docs/sequencer/sequencerApi.md)
    * [API Definition (Open API + JSON Schema)](./apiSpec)
* Participant/Client
    * [Requirements](./docs/participant/participant.md)
* [Initialization](#initialization)

## 10,000 ft Overview
We are performing a ceremony to generate Powers of Tau for KZG proofs on Ethereum. The output of the ceremony consists of 4 (distinct) sets of Powers of Tau each with a different maximum power:
1. $([\tau_1^0]_1, [\tau_1^1]_1, \dots, [\tau_1^{2^{12}-1}]_1])$, $([\tau_1^0]_2, [\tau_1^1]_2, \dots, [\tau_1^{64}]_2])$
2. $([\tau_2^0]_1, [\tau_2^1]_1, \dots, [\tau_2^{2^{13}-1}]_1])$, $([\tau_2^0]_2, [\tau_2^1]_2, \dots, [\tau_2^{64}]_2])$
3. $([\tau_3^0]_1, [\tau_3^1]_1, \dots, [\tau_3^{2^{14}-1}]_1])$, $([\tau_3^0]_2, [\tau_3^1]_2, \dots, [\tau_3^{64}]_2])$
4. $([\tau_4^0]_1, [\tau_4^1]_1, \dots, [\tau_4^{2^{15}-1}]_1])$, $([\tau_4^0]_2, [\tau_4^1]_2, \dots, [\tau_4^{64}]_2])$

### BLS Cryptography

Due to a lack of standardisation for a complete API for BLS curves, we define our own in [BLS.md](./docs/cryptography/BLS.md).

### Contributions

Due to the simplicity of this PoT setup, the full contribution is sent as a json file between the sequencer & participants. This allows the use of a RESTful API between the two. See [contribution documentation](./docs/contribution/contribution.md) for
the contribution format requirements.

## Initialization

The ceremony is initialized to [initialContribution.json](./initialContribution.json).

