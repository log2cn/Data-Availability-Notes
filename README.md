# Refs 

[Data availability encoding](https://notes.ethereum.org/@dankrad/danksharding_encoding)

[A Universal Verification Equation for Data Availability Sampling](https://ethresear.ch/t/a-universal-verification-equation-for-data-availability-sampling/13240)

[ethereum/consensus-specs/specs/fulu/polynomial-commitments-sampling.md](https://github.com/ethereum/consensus-specs/blob/dev/specs/fulu/polynomial-commitments-sampling.md)

[A Mathematical Theory of Danksharding](https://github.com/ingonyama-zk/papers/blob/main/danksharding_math.pdf)

[KZG polynomial commitments](https://dankradfeist.de/ethereum/2020/06/16/kate-polynomial-commitments.html)

[PCS multiproofs using random evaluation](https://dankradfeist.de/ethereum/2021/06/18/pcs-multiproofs.html)

[Data Availability Sampling Phase 1 Proposal](https://hackmd.io/@vbuterin/das)

[2D data availability with Kate commitments](https://ethresear.ch/t/2d-data-availability-with-kate-commitments/8081)

[ethereum/c-kzg-4844](https://github.com/ethereum/c-kzg-4844)

[Fast amortized Kate proofs](https://github.com/khovratovich/Kate/blob/master/Kate_amortized.pdf)

[Multiplying a Toeplitz matrix by a vector](https://alinush.github.io/2020/03/19/multiplying-a-vector-by-a-toeplitz-matrix.html)

[ingonyama-zk/icicle: A hardware acceleration library for compute intensive cryptography](https://github.com/ingonyama-zk/icicle)

[protolambda/go-kzg: KZG and FFT utils](https://github.com/protolambda/go-kzg)

[A note on data availability and erasure coding](https://github.com/ethereum/research/wiki/A-note-on-data-availability-and-erasure-coding)

[availproject/plonk](https://github.com/availproject/plonk/blob/v0.12.0-polygon-2/src/commitment_scheme/kzg10/key.rs#L297)

# Math (rm)
```
Encoder.Encode:
    input[n,l]
    coeff[n,l] // output
    coeff[i] = [w^(-ij)] @ (F_inv * (F * inputFr)[i*l:i*(l+1)]) // i in [0:n], j in [0:l]
    coeff = F * F_inv * (F * input)
          = (F * input)

KzgMultiProofGnarkBackend.ComputeMultiFrameProof:
    proof(f) = F * h(f)
             = F * Toeplitz(f) * s
             = F * (Cyc(f2) * s2)[0:n]
             = F * (F_inv * diag(F * f2) * (F * s2))[0:n]
             = F * (F_inv * (F * f2) * (F * s2))[0:n]
             = F * (F_inv * (coeffStore * FFTPointsT))[0:n]
    // The inner product of two vectors of length l
    (coeffStore * FFTPointsT)[i] = coeffStore[i] @ FFTPointsT[i] // i in [0:2n] 

KzgMultiProofGnarkBackend.computeCoeffStore:
    coeffStore = F * f[2n,l]
    coeffStore.col(j) = F * f2[..., m-j-2l, m-j-l]
SRSTable.Precompute:
    FFTPointsT = F * s[2n,l]
    FFTPointsT.col(j) = F * s2[..., m-j-2l, m-j-l]

variables:
    f: coefficients
constants:
    s: s^0, s^1, ..., s^nl
    F: Fourier Matrix
    l: chunklen
    n: numchunks
theorems:
    proof(f) = F * h(f)
    h(f) = Toeplitz(f) * s
    Cyc(f) = F_inv * diag(F * f) * F
definitions:
    proof(f) = (f(x) - f(s)) / (x - s), x = w, w^2, ..., w^n
```

# Data flow diagram

Recommended VS Code extention: [markdown-mermaid](https://marketplace.visualstudio.com/items?itemName=bierner.markdown-mermaid)

```mermaid
flowchart TB

disperser -- []byte --> blobstore

blobstore -- []byte --> encserver
blobstore --> relay.Server.GetBlob

encserver -- []byte --> prover
prover -- []fr.Element --> encoder
encoder -- []coeffs --> prover
prover -- []fr.Element --> prover1
prover1 -- []bn254.G1Affine --> prover
prover -- []encoding.Frame --> encserver

encserver --> encmgr

relay -- []encoding.Frame --> node

dispatcher -- []v2.BlobHeader --> node
node -- Signature --> dispatcher

encserver -- []encoding.Frame --> chunkstore
chunkstore -- []encoding.Frame --> relay

node -- []v2.BlobHeader --> v2.ValidateBatchHeader 
node -- []BlobCommitment.Commitment []encoding.Frame --> v2.ValidateBlobs --> Verifier.UniversalVerify
node -- []BlobHeader.BlobCommitment --> verify[VerifyBlobLength VerifyCommitEquivalenceBatch]
node --> batchstore --> ServerV2.GetChunks

encmgr -- []v2.BlobHeader --> headerstore
headerstore -- []v2.BlobHeader --> newbatch

newbatch -- []v2.BlobHeader --> dispatcher
dispatcher --> Dispatcher.HandleSignatures

newbatch[Dispatcher.NewBatch]
dispatcher[Dispatcher.HandleBatch]
prover[Prover.GetFrames]
prover1[ComputeMultiFrameProof]
encoder[Encoder.Encode]
encmgr[EncodingManager.HandleBatch]
encserver[EncoderServerV2.handleEncodingToChunkStore]

relay[relay.Server.GetChunks]
node[*node* ServerV2.StoreChunks]
batchstore[(node.StoreV2)]

blobstore[(blobstore)]
chunkstore[(chunkstore)]
headerstore[(BlobMetadataStore)]
```

# Data structures
```
encoding.Frame: 
    Proof:  bn254.G1Affine
    Coeffs: fr.Element

v2.Batch:
    ReferenceBlockNumber: uint64
    BlobCertificates:     []v2.BlobHeader

v2.BlobHeader:
    Commitment:       []byte
    LengthCommitment: []byte
    LengthProof:      []byte
    Length:           uint32
```

# Finite field elements
```
fr.Element: [4]uint64
fp.Element: [4]uint64
bn254.G1Affine: [2]fp.Element
```
