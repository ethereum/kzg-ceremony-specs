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

## Verification

We define the following checks:

### Transcript structure:

- Transcript matches schema
- G1 powers length check
- G2 Powers length check
- Witness length check

### Point Checks

- __G1 Powers Subgroup check__ - For each of the $\mathbb{G}_1$ Powers of Tau (`g1_powers`), verify that they are actually elements of the subgroup.
- __G2 Powers Subgroup check__ - For each of the $\mathbb{G}_2$ Powers of Tau (`g2_powers`), verify that they are actually elements of the subgroup.
- __Witness Subgroup checks__ - For each of the points in `witness`, check that they are actually elements of their respective subgroups.
- __Non-zero check__ - Check that none of the `running_products`s are equal to the point at infinity. Note we need only the `running_products` as it is faster and sufficient to check none of the secrets are 0.
- __Pubkey uniqueness__ - Check that there are no duplicate `pot_pubkey`s across all of the `Transcript`s.

```python
def g1_subgroup_check(P: G1Point) -> bool:
    # If your BLS library API offers a G1 subgroup check, you SHOULD use that as it is likely much faster
    return bls.G1.is_inf(bls.G1.mul(bls.r, P))


def g2_subgroup_check(P: G2Point) -> bool:
    # If your BLS library API offers a G2 subgroup check, you SHOULD use that as it is likely much faster
    return bls.G2.is_inf(bls.G2.mul(bls.r, P))


def subgroup_checks(ceremony: Ceremony) -> bool:
    for transcript in ceremony.transcripts:
        if not all(g1_subgroup_check(P) for P in transcript.powers_of_tau.g1_powers):
            return False
        if not all(g2_subgroup_check(P) for P in transcript.powers_of_tau.g2_powers):
            return False
        if not all(g1_subgroup_check(P) for P in transcript.witness.running_products):
            return False
        if not all(g2_subgroup_check(P) for P in transcript.witness.pot_pubkeys):
            return False
    return True


def non_zero_check(ceremony: Ceremony) -> bool:
    for transcript in transcripts:
        if not all(bls.G1.is_inf(P) for P in transcript.witness.running_products):
            return False
    return True


def pubkey_uniqueness_check(ceremony: Ceremony) -> bool:
    '''
    Note: pubkeys MUST be unique across all transcripts
    Note: This algorithm (comparing bit-representations) suffices iff the pubkeys are stored in affine co-ordinates.
          If projective coordinates are used, then pubkeys must be compared using bls.G2.is_equal()
    '''
    pubkeys = []
    for transcript in ceremony:
        pubkeys += transcript.witness.pot_pubkeys
    return len(pubkeys) == len(list(set(pubkeys)))


def point_checks(ceremony: Ceremony) -> bool:
    if not subgroup_checks(ceremony):
        return False
    if not non_zero_check(ceremony):
        return False
    return pubkey_uniqueness_check(ceremony)
```

#### Using `KeyValidate`

Implementations of the [IRTF BLS draft specification](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-bls-signature-04#section-2.5) MAY expose a `KeyValidate` method. This method performs the non-zero check along with a subgroup check and, as such, MAY be used instead of performing the subgroup and non-zero checks above. Note however, `KeyValidate` is only defined over the group which is used for pubkeys and as such `KeyValidate` methods from "cyphersuites" for both "minimal-pubkey-size" and "minimal-signature-size" are needed. 

### Pairing Checks

- __Running Product construction__ - Verify that the secret $\tau$ is correctly composed of sub-shares contributed by the earlier participants. This is done by checking that $\tau_{i+1} = \tau_i * x_{i+1}$ for each of the previous contributions. This can be performed by performing the multiplication in the exponent of $\mathbb{G}_T$: $e([\tau_{i+1}]_1, g_2) \stackrel{?}{=}e([\tau_i]_1, [x_{i+1}]_2)$
- __Correct construction of G1 Powers__ - Verify that the $\mathbb{G}_1$ points provided are indeed incremental powers of (the same) $\tau$ and that $[\tau]_1$ is the output after that latest contribution. This check is done by asserting that the next $\mathbb{G}_1$ point is the result of multiplying the previous one with $\tau$: $e([\tau^{i+1}]_1, g_2) \stackrel{?}{=}e([\tau^i]_1, [\tau]_2)$
- __Correct construction of G2 Powers__ - Verify that the $\mathbb{G}_2$ points provided are indeed incremental powers of $\tau$ and that $\tau$ is the same across $\mathbb{G}_1$ and $\mathbb{G}_2$. This check is done by verifying that $\tau^i$ is the same across $[\tau^i]_1$ and $[\tau^i]_2$. $e([\tau^i]_1, g_2) \stackrel{?}{=}e(g_2, [\tau^i]_2)$


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
        powers = transcript.powers_of_tau.g1_powers
        pi =  transcript.witness.running_products[-1]
        accumulator.append(powers[:-1], pi, powers[1:], [bls.g1] * num_powers)
    return accumulator

def add_g2_powers_construction_check(ceremony: Ceremony, accumulator: ParingAccumulator) -> PairingAccumulator:
    for transcript in ceremony.transcript:
        num_powers = transcript.num_g2_powers
        g1_powers = transcript.powers_of_tau.g1_powers
        g2_powers = transcript.powers_of_tau.g2_powers
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
