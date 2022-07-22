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
interface Witness{
    running_products: string[]
    pot_pubkeys: string[]
}
```

```typescript
interface Transcript{
    num_g1_powers: number
    num_g2_powers: number
    powers_of_tau: PowersOfTau
    witness: Witness
}
```

```typescript
interface Ceremony {
    transcript: Transcript[]
}
```

### `updateCeremony`

This function takes in a `Ceremony` object and a list of hex-encoded `secrets[]` and returns a new `Ceremony` according to the method specified in [`participant.md`](./participant.md). Note that this function does not perform the required subgroup checks, instead it is expected that this will be done separately via the `ceremonySubgroupCheck` function.

```typescript
function updateCeremony(oldCeremony: Ceremony, secrets: string[]): Ceremony {
}
```

### `ceremonySubgroupCheck`

This function performs the subgroup checks as described in [`participant.md`](./participant.md). Should all the subgroup checks in every transcript pass, this function returns `true`.

```typescript
function ceremonySubgroupCheck(transcript: Transcript): boolean {
}
```
