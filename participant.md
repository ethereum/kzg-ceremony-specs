# Participant

A participatooor downloads the transcript from the coordinator, mixes their local randomness into the SRS and returns the transcript to the coordinator who then verifies that the contributor did not contribute in a malicious* manner.

## Generating randomness

The randomness that each participant contributes to a set of Powers of Tau is in the form of an integer in the range `(1, bls.r)`.

The participant MUST generate 4 different secrets from a _cryptographically-secure pseudo-random number generator_ (CSPRNG) of their choosing.

Each secret MUST meet the following requirements:
- Sourced from a CSPRNG
- Contain at least 255 bits of entropy
- Be different from the other 3 secrets
- Be different from all the other secrets in the transcript
- Be cleared from memory after the contribution is complete

Furthermore, each secret SHOULD meet the following requirements:
- Be uniformly distributed across $\mathbb{F}_r$. (i.e. avoid modulo bias with respect to `bls.r`).

### `KeyGen`
A good method for meeting the above requirements would be to make use of the `KeyGen` function offered by all compliant BLS libraries. This function takes in a seed of at least 32 bytes and returns a uniformly sampled integer of $\mathbb{F}_r$.


## Contributing to the ceremony

### Reserving a slot

### Downloading the transcript

### Verifying the transcript

In order to ensure that the coordinator is not tricking a participant into leaking some of the entropy they are contributing, the participant SHOULD perform the following checks:


#### Transcript structure:

- __Schema Check__ - Verify that the received `transcript.json` matches the `transcriptSchema.json` schema.
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

#### Point Checks

- Subgroup checks
    - __G1 Powers Subgroup check__ - For each of the $\mathbb{G}_1$ Powers of Tau (`g1_powers`), verify that they are actually elements of the prime-ordered subgroup.
    - __G2 Powers Subgroup check__ - For each of the $\mathbb{G}_2$ Powers of Tau (`g2_powers`), verify that they are actually elements of the prime-ordered subgroup.
    - __Running Product Subgroup check__ - Check that the last running product (the one the participant will interact with) is an element of the prime-ordered subgroup.

```python
def subgroup_checks(transcript: Transcript) -> bool:
    for sub_ceremony in transcript.sub_ceremonies:
        if not all(bls.G1.is_in_prime_subgroup(P) for P in sub_ceremony.powers_of_tau.g1_powers):
            return False
        if not all(bls.G2.is_in_prime_subgroup(P) for P in sub_ceremony.powers_of_tau.g2_powers):
            return False
        if not bls.G1.is_in_prime_subgroup(sub_ceremony.witness.running_products[:-1]):
            return False
    return True
```

### Updating the transcript

Once the participant has fetched the `transcript.json` file, for each of the `SubCeremonies`s within they MUST perform the following actions:

- Generate the secrets:
    - Sample a secret `x` from their CSPRNG as per Generating randomness above
- Update Powers of Tau
    - Multiply each of the `powers_of_tau.g1_powers` by incremental powers of `x` and overwrite the `powers_of_tau.g1_powers` in the transcript
    - Multiply each of the `powers_of_tau.g2_powers` by incremental powers of `x` and overwrite the `powers_of_tau.g2_powers` in the transcript
```python
def update_powers_of_tau(sub_ceremony: SubCeremony, x: int) -> SubCeremony:
    '''
    Updates the Powers of Tau within a transcript by multiplying each with a successive power of the secret x.
    '''
    x_i = 1
    for i in range(sub_ceremony.num_g1_powers):
        # Update G1 Powers
        sub_ceremony.powers_of_tau.g1_powers[i] = bls.G1.mul(x_1, sub_ceremony.powers_of_tau.g1_powers[i])
        # Update G2 Powers
        if i < sub_ceremony.num_g2_powers:
            sub_ceremony.powers_of_tau.g2_powers[i] = bls.G2.mul(x_1, sub_ceremony.powers_of_tau.g2_powers[i])
        x_i = (x_i * x) % bls.r
    return sub_ceremony
```
- Update Witness
    - Multiply `witness.running_products[-1]` by `x` and append it to the `witness.running_products` list.
    - Append `bls.G2.mul(x, bls.G2.g2)` to `witness.pot_pubkeys`
```python
def update_witness(sub_ceremony: SubCeremony, x: int) -> SubCeremony:
    new_product = bls.G1.mul(x, sub_ceremony.witness.running_products[-1])
    sub_ceremony.witness.running_products.append(new_product)
    new_pk = bls.G2.mul(x, bls.G2.g2)
    sub_ceremony.witness.pot_pubkeys.append(new_pk)
    return sub_ceremony
```


```python
def update_transcript(transcript: Transcript) -> Transcript:
    for sub_ceremony in transcript.sub_ceremonies:
        x = randbelow(bls.r)
        sub_ceremony = update_powers_of_tau(sub_ceremony, x)
        sub_ceremony = update_witness(sub_ceremony, x)
        del x
    return transcript
```

### Clearing the memory

### Uploading the transcript
