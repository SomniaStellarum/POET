# Core Poet Base Protocol Design Notes

## Overview
We follow the definitions and the reference source code of simple proofs of sequential work. Please read the paper and analyze the python source code. The base protocol implements the construction defined in the paper.

## Constants
- t:int = 150. A statistical security parameter. (Note: is 150 needed for Fiat-Shamir or 21 suffices?)
- w:int = 256. A statistical security parameter.

## Input
- n:int - time parameter
- x: {0,1}^w = rndBelow(2^w - 1) - verifier provided input statement

## Definitions

- N : The time parameter which we assume is of the form 2^n-1 for an integer n

- m:int 0 <= m <= n. Defines how much data to store

- M : Memory available to the prover, of the form (t + n*t + 1 + 2^{m+1})*w, 0 <= m <= n . For example, with w=256, n=40, t=150 and m=20 prover should use around 70MB of memory and make N/1000 queries for openH.

- H : (0, 1)^{<= w(n+1)} -> (0, 1)^w as a random oracle defined. We use sha256 and the commitment x to construct H in the following way: H(i) = sha256( sha256(x) || i).

- φ : (phi) value of Root label l_epsilon computed by PoSW^Hx(N)

- φP : (phi_P) Result of PoSW^Hx(N) stored by prover. List of labels in the m highest layers of the DAG.

- γ : (gamma) (0,1}^{t*w}. A random challenge sampled by verifier and sent to prover for interactive proofs. Created by concatenation of {gamma_1, ..., gamma_t} where gamma_i = rnd_in_range of (0,1)^w

- τ := openH(x,N,φP,γ) proof computed by prover based on verifier provided challenge γ. A list of t tuples where each tuple is defined as: (l_{gamma_i}, dict{alternate_siblings: l_{the alternate siblings}) for 1 <= i <= t (the security param). So, for each i, the tuple contains the label of the node at index gamma_i, as well as the labels of all siblings of the nodes on the path from the node gamma_i to the root.

- NIP (Non-interactive proof): a proof τ created by computing openH for the challenge γ := (H(φ,1),...H(φ,t)). e.g. without receiving γ from the verifier. Verifier asks for the NIP and verifies it like any other openH using verifyH.

- verifyH(x,N,φ,γ,τ) ∈ {accept, reject} - function computed by verifier based on prover provided proof τ.

### Actors (should be implemented as types)
- Prover: The service providing proofs for verifiers
- Verifier: A client using the prover by providing the input statement x, and verifying the prover provided proofs (by issuing random challenges or by verifying a non-interactive verifier provided proof for {PoSW^Hx(N), x}

## Base Protocol Test Use Cases

### User case 1 - Basic random challenges verification test
1. Verifier generates random commitment x and sends it to the prover
2. Prover computes PoSWH(x,N) by making N sequential queries to H and stores φP
3. Prover sends φ to verifier
4. Verifier creates a random challenge γ and sends it to the prover
5. Prover returns proof τ to the verifier
6. Verifier computes verifyH() for τ and outputs accept

### Use case 2 - NIP Verification test
1. Verifier generates random commitment x and sends it to the prover
2. Prover computes PoSWH(x,N) by making N sequential queries to H and stores φP
3. Prover creates NIP and sends it to the verifier
4. Verifier computes verifyH() for NIP and outputs accept

### User case 3 - Memory requirements verification
- Use case 1 with a test that prover doesn't use more than M memory

### User case 4 - Bad proof detection
- Modify use case 1 where a random bit is changed in τ proof returned to the verifier by the prover
- Verify that verifyH() outputs reject for the verifier

### User case 5 - Bad proof detection
- Modify use case 2 where a random bit is changed in the NIP returned to the verifier by the prover
- Verify that verifyH() outputs reject for the verifier

# Base Protocol POC Implementation Guide
1. Implement the base protocol POC and its test use cases with go-lang
2. Implement a type for the verifier functions
3. Implement a type for the prover functions
4. Implement prover and verifier as distinct types that use the function types
5. Use [sha256-smd](https://github.com/minio/sha256-simd) library for sha256()
6. Implement the test use cases as standard go-lang tests

# First task for the open source community is to implement the POC as defined in this doc.