# Powers of Tau Specification


## 10'000 ft Overview
We are performing a ceremony to generate Powers of Tau for KZG proofs on Ethereum. The output of the ceremony consists of 4 (distinct) sets of Powers of Tau each with a different maximum power:
1. $([\tau^0]_1, [\tau^1]_1, \dots, [\tau^{2^{12}-1}]_1])$, $([\tau^0]_2, [\tau^1]_2, \dots, [\tau^{64}]_2])$
2. $([\tau^0]_1, [\tau^1]_1, \dots, [\tau^{2^{13}-1}]_1])$, $([\tau^0]_2, [\tau^1]_2, \dots, [\tau^{64}]_2])$
3. $([\tau^0]_1, [\tau^1]_1, \dots, [\tau^{2^{14}-1}]_1])$, $([\tau^0]_2, [\tau^1]_2, \dots, [\tau^{64}]_2])$
4. $([\tau^0]_1, [\tau^1]_1, \dots, [\tau^{2^{15}-1}]_1])$, $([\tau^0]_2, [\tau^1]_2, \dots, [\tau^{64}]_2])$

## BLS Cryptography

Due to a lack of standardisation for a complete ALI for BLS curves, we define our own in [BLS.md](/BLS.md).

## Transcript

Due to the simplicity of this PoT setup, we only send over the entire transcript to participants and they MAY choose whether to verify the every contribution before participating themselves.

```python
@dataclass
class PowersOfTau:
    g1_powers: List[bls.G1Point]
    g2_powers: List[bls.G2Point]
```

```python
@dataclass
class Witness:
    running_products: List[bls.G1Point]
    pot_pubkeys: List[bls.G2Point]
```

```python
@dataclass
class Transcript:
    num_g1_powers: int
    num_g2_powers: int
    powers_of_tau: PowersOfTau
    witness: Witness
```

```python
@dataclass
class Ceremony
    transcript: List[Transcript]
```

## Initialization

The ceremony is initialized to [initial_ceremony.json](./initial_ceremony.json).

