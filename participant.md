# Participant

A participatooor downloads the ceremony from the coordinator, mixes their local randomness into the SRS and returns the transcript to the coordinator who then verifies that the contributor did not contribute in a malicious* manner.

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

### Updating the transcript

Once the participant has fetched the ceremony file, for each of the `Transcript`s within they MUST perform the following actions:

- Generate the secrets:
    - Sample a secret `x` from their CSPRNG as per Generating randomness above
- Update Powers of Tau
    - Multiply each of the `powers_of_tau.g1_powers` by incremental powers of `x` and overwrite the `powers_of_tau.g1_powers` in the transcript
    - Multiply each of the `powers_of_tau.g2_powers` by incremental powers of `x` and overwrite the `powers_of_tau.g2_powers` in the transcript
```python
def update_powers_of_tau(transcript: Transcript, x: int) -> Transcript:
    '''
    Updates the Powers of Tau within a transcript by multiplying each with a successive power of the secret x.
    '''
    x_i = 1
    for i in range(transcript.num_g1_powers):
        # Update G1 Powers
        transcript.powers_of_tau.g1_powers[i] = bls.G1.mul(x_1, transcript.powers_of_tau.g1_powers[i])
        # Update G2 Powers
        if i < transcript.num_g2_powers:
            transcript.powers_of_tau.g2_powers[i] = bls.G2.mul(x_1, transcript.powers_of_tau.g2_powers[i])
        x_i = (x_i * x) % bls.r
    return transcript
```
- Update Witness
    - Multiply `witness.running_products[-1]` by `x` and append it to the `witness.running_products` list.
    - Append `bls.G2.mul(x, bls.G2.g2)` to `witness.pot_pubkeys`
```python
def update_witness(transcript: Transcript, x: int) -> Transcript:
    new_product = bls.G1.mul(x, transcript.witness.running_products[-1])
    transcript.witness.running_products.append(new_product)
    new_pk = bls.G2.mul(x, bls.G2.g2)
    transcript.witness.pot_pubkeys.append(new_pk)
    return transcript
```


```python
def update_transcript(ceremony: Ceremony) -> Ceremony:
    for transcript in ceremony.transcripts:
        x = randbelow(bls.r)
        transcript = update_powers_of_tau(transcript, x)
        transcript = update_witness(transcript, x)
        del x

       
        
```

### Clearing the memory

### Uploading the transcript
