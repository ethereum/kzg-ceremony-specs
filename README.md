# Powers of Tau Specification
These documents comprise a specification for Ethereum's Powers of Tau (PoT) setup for use in [KZG commitments](https://dankradfeist.de/ethereum/2020/06/16/kate-polynomial-commitments.html) in [EIP-4844 (Proto-Danksharding)](https://eips.ethereum.org/EIPS/eip-4844) and ultimately [Full Danksharding](https://notes.ethereum.org/@dankrad/new_sharding).

The ceremony takes place between _participants_ and the _coordinator_.  _Participants_ are the entities that contribute their secret randomness to the final output $\tau$ s. The roll of the _coordinator_ it to act as the common point of interaction for all participants as well as verifying participants' contribution as the ceremony progresses.

## 10'000 ft Overview
We are performing a ceremony to generate Powers of Tau for KZG proofs on Ethereum. The output of the ceremony consists of 4 (distinct) sets of Powers of Tau each with a different maximum power:
1. $([\tau_1^0]_1, [\tau_1^1]_1, \dots, [\tau_1^{2^{12}-1}]_1])$, $([\tau_1^0]_2, [\tau_1^1]_2, \dots, [\tau_1^{64}]_2])$
2. $([\tau_2^0]_1, [\tau_2^1]_1, \dots, [\tau_2^{2^{13}-1}]_1])$, $([\tau_2^0]_2, [\tau_2^1]_2, \dots, [\tau_2^{64}]_2])$
3. $([\tau_3^0]_1, [\tau_3^1]_1, \dots, [\tau_3^{2^{14}-1}]_1])$, $([\tau_3^0]_2, [\tau_3^1]_2, \dots, [\tau_3^{64}]_2])$
4. $([\tau_4^0]_1, [\tau_4^1]_1, \dots, [\tau_4^{2^{15}-1}]_1])$, $([\tau_4^0]_2, [\tau_4^1]_2, \dots, [\tau_4^{64}]_2])$

## BLS Cryptography

Due to a lack of standardisation for a complete API for BLS curves, we define our own in [BLS.md](/BLS.md).

## `ceremony.json`

Due to the simplicity of this PoT setup, the full transcript is sent as a json file between the coordinator & participants. This allows the use of a RESTful API between the two and verification of certain aspects of the `ceremony.json` file via the [ceremonySchema.json](./ceremonySchema.json) [JSON schema](https://json-schema.org/).


## `Ceremony` object

A `Ceremony` object it defined for ease of specification the structure of which is identical to that of the `ceremony.json`.

### `PowersOfTau`

```python
@dataclass
class PowersOfTau:
    g1_powers: List[bls.G1Point]
    g2_powers: List[bls.G2Point]
```
### `Witness`

```python
@dataclass
class Witness:
    running_products: List[bls.G1Point]
    pot_pubkeys: List[bls.G2Point]
```

### `Transcript`

```python
@dataclass
class Transcript:
    num_g1_powers: int
    num_g2_powers: int
    powers_of_tau: PowersOfTau
    witness: Witness
```

### `Ceremony`

```python
@dataclass
class Ceremony
    transcript: List[Transcript]
```

## Initialization

The ceremony is initialized to [initialCeremony.json](./initialCeremony.json).

