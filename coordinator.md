# Coordinator

The primary job of the coordinator is to sequence participants in the ceremony by acting as the point of contact for participants. The coordinator is required to have a DoS-resilient internet connection and sufficiently powerful hardware to verify the transcript rapidly so that they are able to send the transcript to the next participant as soon as possible. Furthermore, the coordinator should be a semi-trusted entity as they have the power to censor participants, although they cannot affect the safety of the setup.

## Verification

Upon receiving the transcript back from a participant, the Coordinator MUST perform the following checks.

### Transcript structure:

- __Schema Check__ - Verify that the received `ceremony.json` matches the `ceremonySchema.json` schema
```python
def schema_check(ceremony_json: str, schema_path: str) -> bool:
    with open(schema_path) as schema_file:
        schema = json.load(schema_file)
        try:
            jsonschema.validate(ceremony_json, schema)
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
def subgroup_checks(ceremony: Ceremony) -> bool:
    for transcript in ceremony.transcripts:
        if not all(bls.G1.is_on_curve(P) for P in transcript.powers_of_tau.g1_powers):
            return False
        if not all(bls.G2.is_on_curve(P) for P in transcript.powers_of_tau.g2_powers):
            return False
        if not all(bls.G1.is_on_curve(P) for P in transcript.witness.running_products):
            return False
        if not all(bls.G2.is_on_curve(P) for P in transcript.witness.pot_pubkeys):
            return False
    return True
```

- __Non-zero check__ - Check that none of the `running_products`s are equal to the point at infinity. Note we need only to check the `running_products` as we have later check the correct multiplication of `pot_pubkeys` it is faster and sufficient to check none of the secrets are 0.
```python
def non_zero_check(ceremony: Ceremony) -> bool:
    for transcript in ceremony.transcripts:
        if not all(bls.G1.is_inf(P) for P in transcript.witness.running_products):
            return False
    return True
```
- __Pubkey uniqueness__ - Check that there are no duplicate `pot_pubkey`s across all the `Transcript`s.

```python
def pubkey_uniqueness_check(ceremony: Ceremony) -> bool:
    '''
    Note: pubkeys MUST be unique across all transcripts
    Note: This algorithm (comparing bit-representations) suffices iff the pubkeys are stored in affine co-ordinates.
          If projective coordinates are used, then pubkeys must be compared using bls.G2.is_equal()
    '''
    pubkeys = []
    for transcript in ceremony.transcripts:
        pubkeys += transcript.witness.pot_pubkeys
    return len(pubkeys) == len(list(set(pubkeys)))
```

- __Witness continuity check__ - Check that the witness has not been tampered with by the coordinator. This is done by checking that the newly received ceremony's witnesses overlap with the previous ceremony's witnesses in all but the last location where an honest participant would have added their contribution.

```python
def witness_continuity_check(previous_ceremony: Ceremony, new_ceremony: Ceremony) -> bool:
    for previous_transcript, new_transcript in zip(previous_ceremony.transcripts, new_ceremony.transcripts):
        if previous_transcript.witness.running_products[:-1] != new_transcript.witness.running_products:
            return False
        if previous_transcript.witness.pot_pubkeys[:-1] != new_transcript.witness.pot_pubkeys:
            return False
    return True
```

#### Using `KeyValidate`

Implementations of the [IRTF BLS draft specification](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-bls-signature-04#section-2.5) MAY expose a `KeyValidate` method. This method performs the non-zero check along with a subgroup check and, as such, MAY be used instead of performing the subgroup and non-zero checks above. Note however, `KeyValidate` is only defined over the group which is used for pubkeys and as such `KeyValidate` methods from "cyphersuites" for both "minimal-pubkey-size" and "minimal-signature-size" are needed. 

### Pairing Checks

- __Running Product construction__ - Verify that the secret $\tau$ is correctly composed of sub-shares contributed by the earlier participants. This is done by checking that $\tau_{i+1} = \tau_i * x_{i+1}$ for each of the previous contributions. This can be performed by performing the multiplication in the exponent of $\mathbb{G}_T$: $e([\tau_{i+1}]_1, g_2) \stackrel{?}{=}e([\tau_i]_1, [x_{i+1}]_2)$
- __Correct construction of G1 Powers__ - Verify that the $\mathbb{G}_1$ points provided are indeed incremental powers of (the same) $\tau$ and that $[\tau]_1$ is the output after that latest contribution. This check is done by asserting that the next $\mathbb{G}_1$ point is the result of multiplying the previous one with $\tau$: $e([\tau^{i+1}]_1, g_2) \stackrel{?}{=}e([\tau^i]_1, [\tau]_2)$
- __Correct construction of G2 Powers__ - Verify that the $\mathbb{G}_2$ points provided are indeed incremental powers of $\tau$ and that $\tau$ is the same across $\mathbb{G}_1$ and $\mathbb{G}_2$. This check is done by verifying that $\tau^i$ is the same across $[\tau^i]_1$ and $[\tau^i]_2$. $e([\tau^i]_1, g_2) \stackrel{?}{=}e(g_2, [\tau^i]_2)$

