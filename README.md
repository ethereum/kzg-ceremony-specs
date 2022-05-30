# Powers of Tau Specification


## 10'000 ft Overview
We are performing a ceremony to generate Powers of Tau for KZG proofs on Ethereum. The output of the ceremony consists of 4 (distinct) sets of Powers of Tau each with a different maximum power:
1. $([\tau^0]_1, [\tau^1]_1, \dots, [\tau^{2^{12}-1}]_1])$, $([\tau^0]_2, [\tau^1]_2, \dots, [\tau^{63}]_2])$
2. $([\tau^0]_1, [\tau^1]_1, \dots, [\tau^{2^{13}-1}]_1])$, $([\tau^0]_2, [\tau^1]_2, \dots, [\tau^{63}]_2])$
3. $([\tau^0]_1, [\tau^1]_1, \dots, [\tau^{2^{14}-1}]_1])$, $([\tau^0]_2, [\tau^1]_2, \dots, [\tau^{63}]_2])$
4. $([\tau^0]_1, [\tau^1]_1, \dots, [\tau^{2^{15}-1}]_1])$, $([\tau^0]_2, [\tau^1]_2, \dots, [\tau^{63}]_2])$

## Definitions

The output SRS consists of points on BLS12-381, the same curve used for BLS signatures in Ethereum PoS. The curve parameters can be found in the [IRTF Pairing Friendly Curves draft](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-pairing-friendly-curves-10#section-4.2.1).

### Notation

We make use of the following notation throughout this specification:
- $p$ - the field modulus
- $\mathbb{G}_1 \subset E_1(\mathbb{F}_p)$ - the additive subgroup of the curve defined over $\mathbb{F}_p$
- $\mathbb{G}_1 \subset E_2(\mathbb{F}_{p^2})$ - the additive subgroup of the curve defined over $\mathbb{F}_{p^2}$
- $g_1 \in \mathbb{G}_1$, $g_2 \in \mathbb{G}_2$  - the generators of $\mathbb{G}_1$ and $\mathbb{G}_2$ respectively
- $\mathcal{O}_1 \in \mathbb{G}_1$, $\mathcal{O}_2 \in \mathbb{G}_2$  - the points at infinity for $\mathbb{G}_1$ and $\mathbb{G}_2$ respectively
- $e: \mathbb{G}_1 \times \mathbb{G}_2 \to \mathbb{G}_T$ - the bilinear pairing function 
- $r$ - the order of the subgroups $\mathbb{G}_1$, $\mathbb{G}_2$, $\mathbb{G}_T$
- $[a]_1 = a.g_1$ - scalar multiplication (with generator) in $\mathbb{G}_1$
- $[a]_2 = a.g_2$ - scalar multiplication (with generator) in $\mathbb{G}_2$



## Transcript

Due to the simplicity of this PoT setup, we only send over the entire transcript to participants and they MAY choose whether to verify the every contribution before participating themselves.

```python
@dataclass
class PowersOfTau:
    g1_powers: List[G1Point]
    g2_powers: List[G2Point]
```

```python
@dataclass
class Witness:
    running_products: List[G1Point]
    pot_pubkeys: List[G2Point]
```

```python
@dataclass
class Transcript:
    num_g1_powers: int
    num_g2_powers: int
    pot: PowersOfTau
    witness: Witness
```

```python
@dataclass
class Ceremony
    transcripts: List[Transcript]
```

## Intialization

We initialize this trusted setup as follows:

```python
transcript = Transcript(
    num_g1_powers = n
    num_g2_powers = m
    pot = PowersOfTau(
        g1_powers= [bls.g1] * n
        g2_powers= [bls.g2] * m
    )
    witness = Witness(
        running_products = [bls.g1]
        pot_pubkeys = [bls.g2]
    )    
)
```

## Verification

We define the following checks:

### Transcript structure:

- Transcript matches schema
- G1 powers length check
- G2 Powers length check
- Witness length check

### Point Checks

- G1 Subgroup checks
- G2 Subgroup checks
- Witness Subgroup checks
- Non-zero check
- Pubkey uniqueness

#### Using `KeyValidate`

Implementations of the [IRTF BLS draft specification](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-bls-signature-04#section-2.5) MAY expose a `KeyValidate` method. This method performs the non-zero check along with a subgroup check and, as such, MAY be used instead of performing the subgroup and non-zero checks above. Note however, `KeyValidate` is only defined over the group which is used for pubkeys and as such `KeyValidate` methods from "cyphersuites" for both "minimal-pubkey-size" and "minimal-signature-size" are needed. 

### Pairing Checks

- Running Product construction
- Correct construction of G1 Powers
- Correct construction of G2 Powers


### Random Linear combination

Due to the high computational cost of performing pairings, implementors SHOULD make use of random linear combinations to reduce the verification time. The idea is that by pairings can be combined into a single pairing by having the verifier sample a random $r_i$ for each of the pairing checks, combining the points via a linear combination, and only performing one pairing check over the summed points.

```python
@dataclass
class ParingAccumulator:
    g1_l: List[G1Point]
    g2_l: List[G2Point]
    g1_r: List[G1Point]
    g2_r: List[G2Point]

    def append(self, g1_l: List[G1Point], g2_l: List[G2Point], g1_r: List[G1Point], g2_r: List[G2Point]) -> None:
        self.g1_l.append(g1_l)
        self.g2_l.append(g2_l)
        self.g1_r.append(g1_r)
        self.g2_r.append(g2_r)
    
    def random_linear_combination(self) -> Tuple[G1Point, G2Point, G1Point, G2Point]:
        l = len(self.g1_l)
        rand_nums = [randbelow(bls.r) for _ in range(l)]
        g1_ls = [bls.G1.mul(r, P) for r, P in zip(rand_nums, self.g1_l)]
        g2_ls = [bls.G2.mul(r, P) for r, P in zip(rand_nums, self.g2_l)]
        g1_rs = [bls.G1.mul(r, P) for r, P in zip(rand_nums, self.g1_r)]
        g2_rs = [bls.G2.mul(r, P) for r, P in zip(rand_nums, self.g2_r)]

        g1_l = reduce[g1_ls, bls.G1.add]
        g2_l = reduce[g2_ls, bls.G2.add]
        g1_r = reduce[g1_rs, bls.G1.add]
        g2_r = reduce[g2_rs, bls.G2.add]

        return g1_l, g2_l, g1_r, g2_r

def add_running_product_check(ceremony: Ceremony, accumulator: ParingAccumulator) -> PairingAccumulator:
    for transcript in ceremony.transcripts:
        products = transcript.witness.running_products
        pks = transcript.witness.pot_pubkeys
        l = len(witness.running_products)
        accumulator.append(products[:- 1], pks[1:], products[1:], [bls.g1] * l)
    return accumulator

def add_g1_powers_construction_check(ceremony: Ceremony, accumulator: ParingAccumulator) -> PairingAccumulator:
    for transcript in ceremony.transcript:\
        num_powers = transcript.num_g1_powers
        powers = transcript.pot.g1_powers
        pi =  transcript.witness.running_products[-1]
        accumulator.append(powers[:-1], pi, powers[1:], [bls.g1] * num_powers)
    return accumulator

def add_g2_powers_construction_check(ceremony: Ceremony, accumulator: ParingAccumulator) -> PairingAccumulator:
    for transcript in ceremony.transcript:
        num_powers = transcript.num_g2_powers
        g1_powers = transcript.pot.g1_powers
        g2_powers = transcript.pot.g2_powers
        accumulator.append([bls.g1] * num_powers, g2_powers, g1_powers, [bls.g2] * num_powers)
    return accumulator

def verify_ceremony_parings(ceremony: Ceremony) -> bool:
    accumulator = PairingAccumulator()
    accumulator = add_running_product_check(ceremony, accumulator)
    accumulator = add_g1_powers_construction_check(ceremony, accumulator)
    accumulator = add_g2_powers_construction_check(ceremony, accumulator)
    g1_l, g2_l, g1_r, g2_r = accumulator.random_linear_combination()
    return bls.paring(g1_l, g2_l) == bls.paring(g1_r, g2_r)
```

## Contribution

Upon receiving the Transcript from the Coordinator, a Participant
