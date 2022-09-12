# `contribution.json`

Due to the simplicity of this PoT setup, the full contribution is sent as a json file between the sequencer & participants. This allows the use of a RESTful API between the two and verification of certain aspects of the `contribution.json` file via the [contributionSchema.json](../apiSpec/contributionSchema.json) (see: [JSON schema](https://json-schema.org/)).


## `Contribution` object

We define a `Contribution` object for ease of specification the structure of which is identical to that of the `initialContribution.json`.

### `PowersOfTau`

```python
@dataclass
class PowersOfTau:
    g1_powers: List[bls.G1Point]
    g2_powers: List[bls.G2Point]
```

### `SubContribution`

```python
@dataclass
class SubContribution:
    num_g1_powers: int
    num_g2_powers: int
    powers_of_tau: PowersOfTau
    pot_pubkey: bls.G2Point
```

### `Contribution`

```python
@dataclass
class Contribution
    sub_contribution: List[SubContribution]
```