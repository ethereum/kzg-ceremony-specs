# Coordinator

The primary job of the coordinator is to sequence participants in the ceremony by acting as the point of contact for participants. The coordinator is required to have a DoS-resilient internet connection and sufficiently powerful hardware to verify the transcript rapidly so that they are able to send the transcript to the next participant as soon as possible. Furthermore, the coordinator should be a semi-trusted entity as they have the power to censor participants, although they cannot affect the safety of the setup.

## Verification

Upon receiving the transcript back from a participant, the Coordinator MUST perform the following checks.

### Transcript structure:

- __Schema Check__ - Verify that the received `transcript.json` matches the `transcriptSchema.json` schema
```python
def schema_check(transcript_json: str, schema_path: str) -> bool:
    with open(schema_path) as schema_file:
        schema = json.load(schema_file)
        try:
            jsonschema.validate(transcript_json, schema)
            return True
        except Exception:
            pass
    return False
```

### Point Checks

- Subgroup checks
    - __G1 Powers Subgroup check__ - For each of the $\mathbb{G}_1$ Powers of Tau (`g1_powers`), verify that they are actually elements of the subgroup.
    - __G2 Powers Subgroup check__ - For each of the $\mathbb{G}_2$ Powers of Tau (`g2_powers`), verify that they are actually elements of the subgroup.
    - __Witness Subgroup checks__ - For each of the points in `witness`, check that they are actually elements of their respective subgroups.

```python
def subgroup_checks(transcript: Transcript) -> bool:
    for sub_ceremony in transcript.sub_ceremonies:
        if not all(bls.G1.is_in_G1(P) for P in sub_ceremony.powers_of_tau.g1_powers):
            return False
        if not all(bls.G2.is_in_G2(P) for P in sub_ceremony.powers_of_tau.g2_powers):
            return False
        if not all(bls.G1.is_in_G1(P) for P in sub_ceremony.witness.running_products):
            return False
        if not all(bls.G2.is_in_G2(P) for P in sub_ceremony.witness.pot_pubkeys):
            return False
    return True
```

- __Non-zero check__ - Check that none of the `running_products`s are equal to the point at infinity. Note we need only to check the `running_products` as we have later check the correct multiplication of `pot_pubkeys` it is faster and sufficient to check none of the secrets are 0.
```python
def non_zero_check(transcript: Transcript) -> bool:
    for sub_ceremony in transcript.sub_ceremonies:
        if not all(bls.G1.is_inf(P) for P in sub_ceremony.witness.running_products):
            return False
    return True
```

- __Witness continuity check__ - Check that the witness has not been tampered with by the coordinator. This is done by checking that the newly received transcript's witnesses overlap with the previous transcript's witnesses in all but the last location where an honest participant would have added their contribution.

```python
def witness_continuity_check(previous_transcript: Transcript, new_transcript: Transcript) -> bool:
    for previous_sub_ceremony, new_sub_ceremony in zip(previous_transcript.sub_ceremonies, new_transcript.sub_ceremonies):
        if previous_sub_ceremony.witness.running_products != new_sub_ceremony.witness.running_products[:-1]:
            return False
        if previous_sub_ceremony.witness.pot_pubkeys != new_sub_ceremony.witness.pot_pubkeys[:-1]:
            return False
    return True
```

#### Using `KeyValidate`

Implementations of the [IRTF BLS draft specification](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-bls-signature-04#section-2.5) MAY expose a `KeyValidate` method. This method performs the non-zero check along with a subgroup check and, as such, MAY be used instead of performing the subgroup and non-zero checks above. Note however, `KeyValidate` is only defined over the group which is used for pubkeys and as such `KeyValidate` methods from "cyphersuites" for both "minimal-pubkey-size" and "minimal-signature-size" are needed. 

### Pairing Checks

_Note:_ The following pairing checks SHOULD be batched via a random linear combination which would reduce the number of final exponentiations to 2 and decrease the number of Miller loops needed.

- __Running Product construction__ - Verify that the secret $\tau$ is correctly composed of sub-shares contributed by the earlier participants. Let $P_i$ be the $i$th running product: $P_i = [\prod_{j=1}^ix_i]_1$. Correct construction of the running products can then checked by$: $e(P_{i+1}, g_2) \stackrel{?}{=}e(P_i, [x_{i+1}]_2)$

```python
def running_product_construction_check(transcript: Transcript) -> bool:
    for sub_ceremony in transcript.sub_ceremonies:
       products = sub_ceremony.witness.running_products
       pks = sub_ceremony.witness.pot_pubkeys
       for pk, product, next_product in zip(pks[1:], products[:-1], products[1:]):
           if bls.pairing(product, pk) != bls.pairing(next_product, bls.G2.g2):
               return False
    return True
```

- __Correct construction of G1 Powers__ - Verify that the $\mathbb{G}_1$ points provided are indeed incremental powers of (the same) $\tau$ and that $[\tau]_1$ is the output after that latest contribution. This check is done by asserting that the next $\mathbb{G}_1$ point is the result of multiplying the previous one with $\tau$: $e([\tau^{i+1}]_1, g_2) \stackrel{?}{=}e([\tau^i]_1, [\tau]_2)$

```python
def g1_powers_check(transcript: Transcript) -> bool:
    for sub_ceremony in transcript.sub_ceremonies:
       powers = sub_ceremony.powers_of_tau.g1_powers
       pi = sub_ceremony.witness.running_products[-1]
       for power, next_power in zip(powers[:-1], powers[1:]):
           if bls.pairing(pi, power) != bls.pairing(bls.G1.g1, next_power):
               return False
    return True
```

- __Correct construction of G2 Powers__ - Verify that the $\mathbb{G}_2$ points provided are indeed incremental powers of $\tau$ and that $\tau$ is the same across $\mathbb{G}_1$ and $\mathbb{G}_2$. This check is done by verifying that $\tau^i$ is the same across $[\tau^i]_1$ and $[\tau^i]_2$. $e([\tau^i]_1, g_2) \stackrel{?}{=}e(g_2, [\tau^i]_2)$

```python
def g2_powers_check(transcript: Transcript) -> bool:
    for sub_ceremony in transcript.sub_ceremonies:
       g1_powers = sub_ceremony.powers_of_tau.g1_powers
       g2_powers = sub_ceremony.powers_of_tau.g2_powers
       for g1_power, g2_power in zip(g1_powers, g2_powers):
           if bls.pairing(bls.G1.g1, g1_power) != bls.pairing(g2_power, bls.G2.g2):
               return False
    return True
```

