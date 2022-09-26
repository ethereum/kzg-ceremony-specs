# BLS

### Notation

We make use of the following notation throughout this specification:
- $p$ - the field modulus
- $E_1(\mathbb{F}_p)$ the curve defined over $\mathbb{F}_p$
- $E_2(\mathbb{F}_p^2)$ the curve defined over $\mathbb{F}_{p^2}$
- $\mathbb{G}_1 \subset E_1(\mathbb{F}_p)$ - the prime-ordered additive subgroup of $E_1(\mathbb{F}_p)$
- $\mathbb{G}_1 \subset E_2(\mathbb{F}_{p^2})$ - the prime-ordered additive subgroup of $\mathbb{F}_{p^2}$
- $g_1 \in \mathbb{G}_1$, $g_2 \in \mathbb{G}_2$  - the generators of $\mathbb{G}_1$ and $\mathbb{G}_2$ respectively
- $\mathcal{O}_1 \in \mathbb{G}_1$, $\mathcal{O}_2 \in \mathbb{G}_2$  - the points at infinity for $\mathbb{G}_1$ and $\mathbb{G}_2$ respectively
- $e: \mathbb{G}_1 \times \mathbb{G}_2 \to \mathbb{G}_T$ - the bilinear pairing function 
- $r$ - the order of the subgroups $\mathbb{G}_1$, $\mathbb{G}_2$, $\mathbb{G}_T$
- $[a]_1 = a.g_1$ - scalar multiplication (with generator) in $\mathbb{G}_1$
- $[a]_2 = a.g_2$ - scalar multiplication (with generator) in $\mathbb{G}_2$

## Standard

The output SRS consists of points on BLS12-381, the same curve used for BLS signatures in Ethereum PoS. The curve parameters can be found in the [IRTF Pairing Friendly Curves draft standard v10](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-pairing-friendly-curves-10#section-4.2.1).

## API

At present there isn't a wide-spread standard that defines an API with sufficient endpoints to meet the requirements of this ceremony. As such, in this section we define the functions and parameters that we assume an implementor will have access to from the source of BLS cryptography.

### IRTF BLS Standard

The [IRTF CFRG BLS Signature draft standard v04](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-bls-signature-04) makes use of all the API end-points needed to implement this ceremony, but does not REQUIRE implementations to expose them publicly.

Implementations making use of the IRTF BLS Standards SHOULD choose the `BLS_POP_BLS12381G2_XMD:SHA-256_SSWU_RO_POP_` cyphersuite.

### Encoding & types

- `bls.G1Point` - the type of a $\mathbb{G}_1$ point. A 48 byte object with encoding defined as per the z-cash specs.
- `bls.G2Point` - the type of a $\mathbb{G}_2$ point. A 48 byte object with encoding defined as per the z-cash specs.

### Parameters

- `bls.r` the curve order. `r = 52435875175126190479447740508185965837690552500527637822603658699938581184513`

### G1

- `bls.G1.g1` - generator of group $\mathbb{G}_1$
- `bls.G1.add(P, Q)` - EC group addition of points `P` and `Q` $\in \mathbb{G}_1$, returns a $\mathbb{G}_1$ point.
- `bls.G1.mul(x, P)` - Scalar multiplication of point `P` by `x` $\in \mathbb{F}_r$, returns a $\mathbb{G}_1$ point.
- `bls.G1.is_inf(P)` - Returns `True` if `P` $=\mathcal{O}_1$, `False` otherwise
- `bls.G1.is_in_prime_subgroup(P)` - $\mathbb{G}_1$ prime-ordered subgroup check. Returns `True` if `P` $\in\mathcal{G}_1$, `False` otherwise

### G2

- `bls.G2.g2` - generator of group $\mathbb{G}_2$
- `bls.G2.add(P, Q)` - EC group addition of points `P` and `Q` $\in \mathbb{G}_2$, returns a $\mathbb{G}_2$ point.
- `bls.G2.mul(x, P)` - Scalar multiplication of point `P` by `x` $\in \mathbb{F}_r$, returns a $\mathbb{G}_2$ point.
- `bls.G2.is_inf(P)` - Returns `True` if `P` $=\mathcal{O}_2$, `False` otherwise
- `bls.G2.is_in_prime_subgroup(P)` - $\mathbb{G}_2$ prime-ordered subgroup check. Returns `True` if `P` $\in\mathcal{G}_2$, `False` otherwise

### Pairing

- `bls.pairing(P, Q)` - The bilinear map from `P` $\in\mathbb{G}_1$ and `Q` $\in\mathbb{G}_2$ to $\mathbb{G}_T$

### `KeyGen`

- `KeyGen(IKM)` - Takes in `IKM` a byte-string of at least 32 bytes and returns a uniformly random integer `x`, `0 < x < r`. Defined in the IETF BLS Draft standards and REQUIRED to be provided by that API.
