# Math foundation
```
KzgMultiProofGnarkBackend.ComputeMultiFrameProof:
    proof(f) = F * h(f)
             = F * Toeplitz(f) * s
             = F * (Cyc(f2) * s2)[0:l]
             = F * (F2_inv * diag(F2 * f2) * (F2 * s2))[0:l]

SRSTable.PrecomputeSubTable:
    s2[0:l]  = SRSTable.s1[start:start+l]
    s2[l:2l] = 0
    return = F2 * s2

variables:
    f: coefficients
constants:
    s: s^0, s^1, ...
    x: w, w^2, ..., w^l
    F: Fourier Matrix
    l: chunklen
theorems:
    proof(f) = F * h(f)
    h(f) = Toeplitz(f) * s
    Cyc(f) = F_inv * diag(F * f) * F
definitions:
    proof(f)[k] = (f(x) - f(s)) / (x - s), x = w^k
    h(f)[k] = f[k:m] . s[0:m-k]
```

refs:

[Multiplying a Toeplitz matrix by a vector](https://alinush.github.io/2020/03/19/multiplying-a-vector-by-a-toeplitz-matrix.html)

[PCS multiproofs using random evaluation](https://dankradfeist.de/ethereum/2021/06/18/pcs-multiproofs.html)

# Data flow diagram

Recommended VS Code extention: [markdown-mermaid](https://marketplace.visualstudio.com/items?itemName=bierner.markdown-mermaid).

```mermaid
flowchart TB

disperser -- []byte --> blobstore

blobstore -- []byte --> encserver
blobstore --> relay.Server.GetBlob

encserver -- []byte --> prover
prover --> Encoder.Encode --> GetInterpolationPolyCoeff
%% coeffs(k)[i] = fft(chunk)[i] * w^(-ik)
prover -- []fr.Element --> prover1
prover1 -- []bn254.G1Affine --> prover
prover -- []encoding.Frame --> encserver

encserver -- v2.FragmentInfo --> encmgr

relay -- []encoding.Frame --> node

dispatcher -- []v2.BlobHeader --> node
node -- Signature --> dispatcher

encserver -- []encoding.Frame --> chunkstore
chunkstore -- v2.FragmentInfo --> encserver
chunkstore -- []encoding.Frame --> relay

node -- []v2.BlobHeader --> v2.ValidateBatchHeader 
node -- []BlobCommitment.Commitment []encoding.Frame --> v2.ValidateBlobs --> Verifier.UniversalVerify
node -- []BlobHeader.BlobCommitment --> verify[VerifyBlobLength VerifyCommitEquivalenceBatch]
node --> batchstore --> ServerV2.GetChunks

encmgr -- []v2.BlobHeader --> certstore
certstore -- []v2.BlobHeader --> newbatch

newbatch -- []v2.BlobHeader --> dispatcher
dispatcher --> Dispatcher.HandleSignatures

newbatch -- []merkletree.Proof --> verifstore 

newbatch[Dispatcher.NewBatch]
dispatcher[Dispatcher.HandleBatch]
prover[Prover.GetFrames]
prover1[ComputeMultiFrameProof]
encmgr[EncodingManager.HandleBatch]
encserver[EncoderServerV2.handleEncodingToChunkStore]

relay[relay.Server.GetChunks]
node[*node* ServerV2.StoreChunks]
batchstore[(node.StoreV2)]

blobstore[(blobstore)]
chunkstore[(chunkstore)]
verifstore[(BlobMetadataStore)]
certstore[(BlobMetadataStore)]
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
