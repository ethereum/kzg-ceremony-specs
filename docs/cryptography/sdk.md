# SDK

We define an SDK as an OPTIONAL interface between the cryptography defined in this repo and the rest of the functionality required to implement the participant software. The goal is to simplify the job of an implementor by giving them a standardised interface to handle the cryptographic aspects of this ceremony.

## Interface

### 

# Crypto SDK

Implementing & using this SDK is **OPTIONAL**.

The purpose of this SDK is to standardise the interface between the cryptography aspect of this participant software and the rest. By isolating the cryptography into a component that implementors just interact with via an SDK, it is easier for implementors to swap out the cryptography implementation, & makes testing easier.




### Interface Types

```typescript
interface PowersOfTau{
    g1_powers: string[]
    g2_powers: string[]
}
```

```typescript
interface Contribution{
    num_g1_powers: number
    num_g2_powers: number
    powers_of_tau: PowersOfTau
    pot_pubkey?: string
}
```

```typescript
interface BatchContribution {
    contributions: Contribution[]
}
```

### `contribute`

This function takes in a `BatchContribution` object and a list of hex-encoded `secrets[]` and returns a new `BatchTranscript` according to the method specified in [`participant.md`](../participant/participant.md). Note that this function __does not perform the recommended subgroup checks__, instead it is expected that this will be done separately via the `checkContribution` function. `contribute()` will raise an `Error` should it fail to deserialise and of the strings or otherwise not be able to proceed.

```typescript
function contribute(contribution: BatchContribution, secrets: string[]): BatchContribution {
}
```

### `checkContribution`

This function performs the prime-ordered subgroup checks as described in [`participant.md`](../participant/participant.md). Should all the subgroup checks in the contribution pass, this function returns `true`.

```typescript
function checkContribution(contribution: BatchContribution): boolean {
}
```
