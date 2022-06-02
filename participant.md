# Participant

A participatooor downloads the ceremony from the coordinator, mixes their local randomness into the SRS and returns the transcript to the coordinator who then verifies that the contributor did not contribute in a malicious* manner.

## Generating randomness

The randomness that each participant contributes to a set of Powers of Tau is in the form of an integer in the range `(1, bls.curve_order)`.

The participant MUST generate 4 different secrets from a _cryptographically-secure pseudo-random number generator_ (CSPRNG) of their choosing.

Henceforth, the secrets will be identified as: `(x_0, x_1, x_2, x_3)`

Each secret MUST meet the following requirements:
- Sourced from a CSPRNG
- Contain at least 255 bits of entropy
- Be different from the other 3 secrets
- Be different from all the other secrets in the transcript
- Be cleared from memory after the contribution is complete

Furthermore, each secret SHOULD meet the following requirements:
- Be uniformly distributed across $\mathbb{F}_r$. (i.e. avoid modulo bias with respect to `bls.curve_order`).

### `KeyGen`
A good method for meeting the above requirements would be to make use of the `KeyGen` function offered by all compliant BLS libraries. This function takes in a seed of at least 32 bytes and returns a uniformly sampled integer of $\mathbb{F}_r$.


