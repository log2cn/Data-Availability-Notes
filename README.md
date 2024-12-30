# Implementations and underlying libraries

ethereum/consensus-specs: [specs/fulu/polynomial-commitments-sampling.md](https://github.com/ethereum/consensus-specs/blob/dev/specs/fulu/polynomial-commitments-sampling.md)

[ethereum/c-kzg-4844](https://github.com/ethereum/c-kzg-4844): A minimal implementation of the Polynomial Commitments API for EIP-4844 and EIP-7594.

[Layr-Labs/eigenda](https://github.com/Layr-Labs/eigenda)

[protolambda/go-kzg](https://github.com/protolambda/go-kzg): FFT, data-recovery and KZG commitments, in Go.

[availproject/plonk](https://github.com/availproject/plonk/blob/v0.12.0-polygon-2/src/commitment_scheme/kzg10/key.rs#L297)

[crate-crypto](https://github.com/orgs/crate-crypto/repositories)

# Posts and discussions

[Data availability encoding](https://notes.ethereum.org/@dankrad/danksharding_encoding)

[A Universal Verification Equation for Data Availability Sampling](https://ethresear.ch/t/a-universal-verification-equation-for-data-availability-sampling/13240)

[ethereum/consensus-specs](https://github.com/ethereum/consensus-specs#specs)

[A Mathematical Theory of Danksharding](https://github.com/ingonyama-zk/papers/blob/main/danksharding_math.pdf)

[KZG polynomial commitments](https://dankradfeist.de/ethereum/2020/06/16/kate-polynomial-commitments.html)

[PCS multiproofs using random evaluation](https://dankradfeist.de/ethereum/2021/06/18/pcs-multiproofs.html)

[Data Availability Sampling Phase 1 Proposal](https://hackmd.io/@vbuterin/das)

EIP-7594: [From 4844 to Danksharding: a path to scaling Ethereum DA](https://ethresear.ch/t/from-4844-to-danksharding-a-path-to-scaling-ethereum-da/18046)

l2beat: [Data Availability Summary](https://l2beat.com/data-availability/summary)

[2D data availability with Kate commitments](https://ethresear.ch/t/2d-data-availability-with-kate-commitments/8081)

[Fast amortized Kate proofs](https://github.com/khovratovich/Kate/blob/master/Kate_amortized.pdf) ([iacr](https://eprint.iacr.org/2023/033))

[Multiplying a Toeplitz matrix by a vector](https://alinush.github.io/2020/03/19/multiplying-a-vector-by-a-toeplitz-matrix.html)

[A note on data availability and erasure coding](https://github.com/ethereum/research/wiki/A-note-on-data-availability-and-erasure-coding)

# Math
```
    Coeff = F^(-1) * P * Y // compute Coeff from Y
<=> F * Coeff = P * Y
<=> Y = P * F * Coeff // Y[i] = Coeff(w^rbo(i))

theorems:
    P^2 = I // bit reversal permutation matrix
    F = [w^(ij)] where i, j = 0, 1, ..., N-1
    F^(-1) = N^(-1) * [w^(-ij)] where i, j = 0, 1, ..., N-1
```
