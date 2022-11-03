# Participant

A participant downloads the contribution file from the sequencer, mixes their local randomness into the SRS
and returns the contribution to the sequencer who then verifies that the contributor did not contribute
in a malicious* manner.

## Generating randomness

The randomness that each participant contributes to a set of Powers of Tau is in the form of an integer in the range `(1, bls.r)`.

The participant MUST generate 4 different secrets from a _cryptographically-secure pseudo-random number generator_ (CSPRNG) of their choosing.

Each secret MUST meet the following requirements:
- Sourced from a CSPRNG
- Contain at least 255 bits of entropy
- Be different from the other 3 secrets used in the other sub-ceremonies
- Be cleared from memory after the contribution is complete

Furthermore, each secret SHOULD meet the following requirements:
- Be uniformly distributed across $\mathbb{F}_r$. (i.e. avoid modulo bias with respect to `bls.r`).

### `KeyGen`
A good method for meeting the above requirements would be to make use of the `KeyGen` function offered by all compliant BLS libraries. This function takes in a seed of at least 32 bytes and returns a uniformly sampled integer of $\mathbb{F}_r$.


## Contributing to the ceremony

### Reserving a slot

### Downloading the contribution file

### Verifying the contribution file

In order to ensure that the sequencer is not tricking a participant into leaking some of the entropy they are contributing, the participant SHOULD perform the following checks:


#### Contribution file structure:

- __Schema Check__ - Verify that the received `contribution.json` matches the `contributionSchema.json` schema.
```python
def schema_check(contribution_json: str, schema_path: str) -> bool:
    with open(schema_path) as schema_file:
        schema = json.load(schema_file)
        try:
            jsonschema.validate(contribution_json, schema)
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

def subgroup_checks(batch_contribution: BatchContribution) -> bool:
    for contribution in batch_contribution.contributions:
        if not all(bls.G1.is_in_prime_subgroup(P) for P in contribution.powers_of_tau.g1_powers):
            return False
        if not all(bls.G2.is_in_prime_subgroup(P) for P in contribution.powers_of_tau.g2_powers):
            return False
    return True
```

### Updating the contribution file

Once the participant has fetched the `contribution.json` file, for each of the `Contributions`s within they MUST perform the following actions:

- Generate the secrets:
    - Sample a secret `x` from their CSPRNG as per Generating randomness above
- Update Powers of Tau
    - Multiply each of the `powers_of_tau.g1_powers` by incremental powers of `x` and overwrite the `powers_of_tau.g1_powers` in the contribution
    - Multiply each of the `powers_of_tau.g2_powers` by incremental powers of `x` and overwrite the `powers_of_tau.g2_powers` in the contribution

```python
def update_powers_of_tau(contribution: Contribution, x: int) -> Contribution:
    '''
    Updates the Powers of Tau within a sub-ceremony by multiplying each with a successive power of the secret x.
    '''
    x_i = 1
    for i in range(contribution.num_g1_powers):
        # Update G1 Powers
        contribution.powers_of_tau.g1_powers[i] = bls.G1.mul(x_1, contribution.powers_of_tau.g1_powers[i])
        # Update G2 Powers
        if i < contribution.num_g2_powers:
            contribution.powers_of_tau.g2_powers[i] = bls.G2.mul(x_1, contribution.powers_of_tau.g2_powers[i])
        x_i = (x_i * x) % bls.r
    return contribution
```

- Update Witness
    - Set `contribution.pot_pubkey` to `bls.G2.mul(x, bls.G2.g2)`

```python
def update_witness(contribution: Contribution, x: int) -> Contribution:
    contribution.pot_pubkey = bls.G2.mul(x, bls.G2.g2)
    return contribution
```

### Signing the contributions

The signing of contributions with Ethereum ECDSA keys and BLS signing the user's identity is RECOMMENDED.

- Getting the encoded identity:
    - Depending on whether the user logged in with an Ethereum Address or a GitHub handle, they will encode their identity into bytes via either 
- BLS-Sign Identity (RECOMMENDED):
    - Use the secret `x` as a private key to sign the user's encoded identity.
- ECDSA Sign the user's PoT Pubkeys (RECOMMENDED)
    - Only perform these steps if using an Ethereum Address to login
    - Build the EIP712 `TypedData` JSON object as per [contributionSigning.md](../cryptography/contributionSigning.md)
    - Use `web3.eth.signTypedData()` to sign the message


```python
def sign_identity(contribution: Contribution, x: int, identity: str) -> Contribution:
    encoded_identity = b''
    if identity[:2] == '0x':
        # Identity is a Ethereum address
        encoded_identity = eth_address_to_identity(identity).encode()
    else:
        # Identity is an GitHub ID
        encoded_identity = github_handle_to_identity(identity).encode()
    signature = bls.Sign(x, encoded_identity)
    contribution.bls_signature = signature
```

```python
def sign_contribution(batch_contribution: BatchContribution, ethereum_address: Optional[str]) -> BatchContribution:
    typed_data = contribution_to_typed_data_str(contribution)  # function defined in contributionSigning.md
    contribution.ecdsa_signature = web3.eth.sign_typed_data(ethereum_address, json.loads(typed_data))
    return batch_contribution
```

### Tying it all together

_Note:_ contribution signing can be done before the participant ever receives the contribution from the sequencer by deciding on the secret ahead of time. Doing so is the RECOMMENDED order of operations so as not to slow down the queue while waiting for the (user's) ECDSA signature.

```python
def contribute(batch_contribution: BatchContribution,
               identity: Optional[str] = None,
               ethereum_address: Optional[str] = None) -> BatchContribution:
    for contribution in batch_contribution.contributions:
        x = randbelow(bls.r)
        contribution = update_powers_of_tau(contribution, x)
        contribution = update_witness(contribution, x)
        if identity is not None:
            contribution = sign_identity(contribution, x, identity)
        del x
    if ethereum_address is not None:
        batch_contribution = sign_contribution(batch_contribution, ethereum_address)
    return batch_contribution
```

### Clearing the memory

### Uploading the contribution file
