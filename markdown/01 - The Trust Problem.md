# Chapter 1: The Trust Problem

> *"A proof that requires trust in the prover is not yet a proof, it is a promise."*

You send your computation to the cloud. The cloud sends back an answer.

Why should you believe it?

This question sounds almost paranoid until you think carefully about what's at stake. The server might be compromised. The operator might be malicious. The hardware might be faulty. The software might contain bugs. Even if everything works correctly, how would you *know*? The only evidence you have is the answer itself, and the answer, by itself, proves nothing.

Here's the fundamental asymmetry: executing a computation takes resources (time, memory, energy). But *checking* whether the computation was done correctly... also takes resources. In many cases, the same resources. If you could check the answer cheaply, you wouldn't have outsourced the computation in the first place.

This is the **trust problem in computation**: how do you verify without redoing all the work?

### Truth Without a Judge

For millennia, knowledge has traveled through testimony. One person tells another: "I computed this result." The listener judges whether to believe. This judgment rests on reputation, authority, past behavior. All the machinery of social trust.

What if claims could carry their own evidence? Not testimony backed by reputation. Not certificates issued by authorities. Something stranger: an object that proves itself. If the claim is false, the object cannot exist. If the object exists and passes inspection, the claim must be true. No judge required.

This is not metaphor. The technology exists. A computational claim can be accompanied by a mathematical object, a *proof*, that anyone can verify in milliseconds. The proof works not because you trust the prover, but because mathematics makes cheating impossible. Two machines that have never communicated can verify the same proof and reach the same conclusion, not because they negotiated, but because the structure of mathematics forces the same answer from any system capable of arithmetic.

We will call this *arithmetic consensus*: agreement enforced by structure rather than achieved by persuasion. Chapter 2 develops the mechanism (the Schwartz-Zippel lemma) and explores why this represents a genuinely new foundation for intersubjective truth. For now, hold this question: what becomes possible when "I trust you" can be replaced with "I verified the math"?

This book teaches you how to build such proofs.



## When Verification Is Easy

Before confronting the hard case, consider situations where verification *is* easy, given the right certificate.

**Factorization**: Given $n$ and a claim that $p, q$ are its prime factors. Verification: multiply $p \times q$, check it equals $n$, verify $p$ and $q$ are prime. Finding those factors is believed to require exponential time; checking them takes polynomial time.

**Graph coloring**: Given a graph and a claimed 3-coloring. Verification: for each edge, check that its endpoints have different colors. Finding such a coloring is NP-hard; verifying one is linear in the number of edges.

**Satisfying assignments**: Given a Boolean formula and a claimed satisfying assignment. Verification: substitute the values and evaluate each clause. Finding such an assignment is NP-complete; checking one is polynomial.

These are problems in **NP**: the class of problems where, if someone hands you a proposed solution, you can *check* whether it's correct in reasonable time (polynomial in the input size). NP doesn't say anything about how hard it is to *find* a solution, only how hard it is to *verify* one. The proposed solution serves as a *witness* or *certificate* of correctness.

Note the asymmetry: NP captures "easy to verify," not "hard to find." Some NP problems are easy to solve (every problem in P is also in NP). The interesting cases are those where finding appears hard but verifying is easy. This gap is what proof systems exploit.


## When Verification Seems As Hard As Computation

But many problems don't have short certificates.

The obvious verification strategy is to recompute: run the same algorithm on the same inputs and compare results. This works, but it defeats the purpose. You outsourced because you couldn't (or didn't want to) pay the computational cost. Verification that costs as much as the original computation is no verification at all.

For a moment, consider what "cheap verification" would even mean. The computation processes some input of size $n$, takes $T$ steps, and produces an output. Cheap verification would mean checking correctness in time $o(T)$: strictly less than the original computation. Ideally, much less. Ideally, polylogarithmic in $T$, or even constant.

But this seems impossible. How can you verify a computation without understanding what it computed? How can you understand what it computed without retracing its steps? The answer is computed from the input through a long chain of operations; surely checking requires following that chain?

The remarkable discovery of the 1980s and 1990s was that this intuition is wrong.



## Interactive Proofs: The Breakthrough

The key insight came from complexity theory, and it involved a conceptual leap: *interaction* and *randomness* together can create verification power that neither possesses alone.

Consider this scenario. A computationally unbounded prover claims to have solved a problem. A polynomially bounded verifier wants to check this claim. The verifier cannot solve the problem themselves (that's the whole point), but they can engage in a conversation with the prover.

In an **interactive proof**, the verifier sends random challenges, the prover responds, and after some number of rounds, the verifier decides whether to accept or reject the claim.

The magic is in two properties:

**Completeness**: If the claim is true, an honest prover can always convince the verifier to accept.

**Soundness**: If the claim is false, no prover, no matter how clever or powerful, can convince the verifier to accept, except with negligible probability.

The probability in soundness comes from the verifier's randomness. The prover doesn't know in advance what challenges the verifier will send. A cheating prover must prepare for all possible challenges, and this is where they fail. The space of possible challenges is exponentially large; the prover cannot succeed at all of them if the claim is false.



## Randomness Creates Asymmetry

Here's a simple example that illustrates the power of randomness in verification.

Suppose I claim two polynomials $p(x)$ and $q(x)$ are identical. Both polynomials have degree at most $d$, and their coefficients are elements of a large finite field $\mathbb{F}$ of size $|\mathbb{F}| = 2^{256}$.

**Without randomness**, verifying this claim requires comparing all $d+1$ coefficients. If $d$ is large, this is expensive.

**With randomness**, verification becomes trivial:

1. Pick a random $r \in \mathbb{F}$

2. Evaluate $p(r)$ and $q(r)$

3. Accept if they're equal, reject otherwise

If $p = q$, then $p(r) = q(r)$ for all $r$. Verification always succeeds.

If $p \neq q$, then $p - q$ is a nonzero polynomial of degree at most $d$. Such a polynomial has at most $d$ roots. The probability that our random $r$ hits a root is at most $d / |\mathbb{F}|$.

With $d = 10^6$ and $|\mathbb{F}| = 2^{256}$:

$$\Pr[\text{cheating succeeds}] \leq \frac{10^6}{2^{256}} \approx 2^{-236}$$

This is so small it's effectively zero. One random evaluation suffices.

This is the **Schwartz-Zippel lemma** in action. We'll see it again and again throughout this book. It's perhaps the most important tool in interactive proofs: random evaluation catches disagreement between polynomials with overwhelming probability.



## From IP to Succinctness

The theoretical study of interactive proofs established profound results:

**IP = PSPACE**: Interactive proofs with polynomial-time verifiers can verify exactly the class **PSPACE**. What is PSPACE? It's the class of problems solvable using a reasonable amount of memory (polynomial in the input size), but with no limit on time. A PSPACE algorithm might run for centuries, but it can only use a bounded scratch pad. This includes problems like determining the winner in generalized chess (with an $n \times n$ board) or evaluating quantified Boolean formulas ("for all $x$, there exists $y$, such that..."). These problems are believed to be far harder than NP. The verifier's randomness and the prover's computational power combine to verify claims that seem uncheckable.

But these theoretical protocols had a problem: they weren't *succinct*. The total communication (the number of bits exchanged between prover and verifier) could be polynomial in the computation size. Better than redoing the computation, but not by much.

The goal of **succinct arguments** is more ambitious: proofs that are *polylogarithmic* in the computation size, or even constant. A computation taking billions of steps should yield a proof of hundreds or thousands of bits, not billions.

Achieving this goal required new proof models: extensions and variants of interactive proofs that enabled different trade-offs between interaction, query access, and succinctness.



## The Proof System Zoo

The path from interactive proofs to modern SNARKs runs through several distinct proof models. Understanding this taxonomy clarifies where different techniques come from and why modern systems take the forms they do.

### Interactive Proofs (IP)

The starting point. A prover and verifier exchange messages. The verifier uses randomness to catch cheating. Security is *information-theoretic*: even an all-powerful prover cannot convince the verifier of a false statement (except with negligible probability).

Think of it as courtroom cross-examination. The prover (witness) wants to convince the verifier (judge) of some claim. The judge cannot independently verify the facts; they weren't there, they don't have the evidence. But through clever questioning, the judge can probe for inconsistencies. An honest witness has nothing to hide; their answers will be consistent. A lying witness must maintain a web of fabrications, and random probing questions will eventually find a thread that unravels it.

The class **IP** contains all languages with such protocols where the verifier runs in polynomial time. The theorem **IP = PSPACE** (Shamir, 1990) shows this class is remarkably large, far larger than NP. The verifier's random questions, combined with the prover's unbounded computational power, can verify claims that no static certificate could capture.

### Multi-Prover Interactive Proofs (MIP)

What if the verifier could interrogate *multiple* provers who cannot communicate with each other?

Imagine two suspects in separate rooms: the classic police interrogation. The detective asks each suspect questions, comparing answers for consistency. If the suspects are telling the truth, their stories align effortlessly. If they're lying, they can't coordinate their lies without communicating, and they can't communicate. The detective doesn't need to know the truth themselves; they only need to catch inconsistencies between the two stories.

In a **Multi-Prover Interactive Proof**, two or more provers share the witness but cannot exchange messages during the protocol. The verifier sends different challenges to each prover and cross-checks their responses.

The deep insight here is **non-adaptivity**. In a single-prover IP, the prover sees the verifier's first challenge before answering, then sees the second challenge before answering again. The prover *adapts* to each challenge in sequence. With two non-communicating provers, the verifier can send different questions simultaneously; neither prover knows what the other was asked. This forces both provers to commit to a consistent story *before* seeing the cross-examination.

This apparently simple change unleashes enormous verification power:

**MIP = NEXP** (Babai, Fortnow, Lund, 1991): Multi-prover proofs can verify problems in **NEXP**, which stands for nondeterministic exponential time. What does this mean? Recall that NP is the class where solutions can be verified quickly. NEXP is the exponentially larger cousin: problems where the solution itself might be exponentially large (so even *writing it down* takes exponential time), but once written, it can be checked in exponential time. These are vastly harder problems than NP or even PSPACE.

The gap from PSPACE to NEXP is vast. The non-communication constraint is what makes it possible: the verifier can probe two points of a story simultaneously, catching inconsistencies that a single adaptive prover could finesse.

This idea of forcing commitment before challenge reappears throughout SNARK design. When we study polynomial commitment schemes, we'll see the same principle: the prover commits to a polynomial, then the verifier challenges. The commitment plays the role of the second prover: it locks in answers before the questions are known.

### Probabilistically Checkable Proofs (PCP)

Here the model shifts from interaction to *query access*. The prover writes down a static proof string $\pi$ (potentially very long). The verifier doesn't read the whole string. Instead, they query a few random positions and decide based on what they see.

A **PCP** is characterized by two parameters, both functions of the input size $n$:

- How many random bits the verifier uses

- How many positions in the proof string the verifier queries

The verifier's decision depends only on the input, their random coin flips, and the few bits they read from the proof.

The **PCP Theorem** (Arora, Safra; Arora, Lund, Motwani, Sudan, Szegedy; 1992) is one of the landmark results of complexity theory:

$$\textbf{NP} = \textbf{PCP}[O(\log n), O(1)]$$

What does "every NP problem has a PCP" mean? Recall that an NP problem is one where solutions can be verified quickly given a witness (like checking that a proposed graph coloring is valid). The PCP theorem says something stronger: for any such problem, there exists a way to encode the witness into a longer proof string such that the verifier uses only $O(\log n)$ random bits and queries only a *constant* number of proof positions. The proof might be polynomial-size, but verification reads only $O(1)$ bits.

How can this possibly work? The key is **structured redundancy**.

Think of a completed Sudoku puzzle. The puzzle has internal constraints: each row, column, and 3×3 box must contain the digits 1-9 exactly once. Now imagine "corrupting" one cell by changing a 7 to a 3. This single error violates the constraint for its row, its column, and its box. One mistake creates evidence in multiple places. A random spot-check has a decent chance of catching it.

PCPs work the same way, but with vastly more redundancy. The proof is not the raw witness; it's an *encoded* version where local constraints interlock globally. The encoding transforms the witness into a form where any error, any deviation from a valid proof, creates detectable inconsistencies across many positions.

The technology: low-degree polynomial encoding. The witness is interpreted as evaluations of a polynomial, then extended to many more points. Polynomial structure ensures that errors propagate: a polynomial that's wrong at even one point must disagree with the correct polynomial almost everywhere (Schwartz-Zippel, again). Random queries catch these disagreements with high probability.

This is remarkable. A satisfying assignment to a million-variable formula might require a million bits to write down. But there exists an encoding, a PCP, where checking validity requires reading only, say, 3 bits. The encoding has redundancy; errors anywhere propagate everywhere, detectable by sparse sampling.

### The MIP-PCP Connection

There's a deep connection between multi-prover proofs and PCPs. The two non-communicating provers in an MIP can be simulated by a single long proof string: each possible pair of questions to the two provers corresponds to a position in the string, with the answer pair as the value at that position.

The non-communication constraint in MIP becomes a consistency requirement in PCP: the answers at different positions must be consistent with some underlying witness. The verifier's power to cross-check provers becomes the power to query random positions and check consistency.

This connection was key to proving MIP = NEXP and to subsequent PCP constructions.

### Interactive Oracle Proofs (IOP)

The models above involve a choice: interaction (IP) or query access (PCP). **Interactive Oracle Proofs** combine both.

In an IOP, the protocol has multiple rounds. In each round, the prover sends a *proof string* (or, more abstractly, an *oracle*). The verifier can query this oracle at chosen positions, then sends a challenge. The prover responds with another oracle, and so on.

Why combine interaction and oracles? Each compensates for the other's weakness. Pure PCPs require enormous proof strings to achieve low soundness error; the proof must anticipate all possible verifier randomness. Pure IPs require many rounds of back-and-forth; the verifier probes incrementally, each round narrowing the space of consistent lies. IOPs get the best of both: the prover commits to an oracle (like a PCP), then the verifier challenges (like an IP), then another oracle, another challenge. Each oracle only needs to handle the challenges that could follow *given previous commitments*.

This hybrid captures what modern SNARK constructions actually do:

1. Prover commits to a polynomial (an oracle that the verifier can query for evaluations)

2. Verifier sends a random challenge

3. Prover commits to another polynomial

4. Repeat

5. Verifier makes a few queries and decides

The IOP abstraction separates the *protocol logic* from the *implementation of oracles*. The oracle is abstract; the verifier magically gets evaluations at chosen points. Chapter 11 shows how polynomial commitment schemes instantiate these oracles cryptographically.

### Linear PCPs

A further refinement, important for understanding Groth16 and pairing-based SNARKs.

In a standard PCP, the proof is a string $\pi \in \{0,1\}^m$ and the verifier queries specific bit positions. In a **Linear PCP**, the proof is a *linear function* $\pi: \mathbb{F}^k \to \mathbb{F}$, and the verifier queries the function at chosen input vectors.

That is, instead of asking "what is $\pi[i]$?" the verifier asks "what is $\pi(q)$ for query vector $q$?"

Think of it as a library where you can't check out books, but you can ask the librarian to compute weighted sums of page counts. "How many total pages in books 1, 3, and 7?" The librarian answers. You ask several such questions. From the answers (linear combinations of the underlying data), you try to verify some property of the books themselves.

The linearity constraint is extraordinarily powerful. If the proof is a linear function, then the verifier's queries are also linear operations. This enables a beautiful trick: **encode the proof in a group where only linear operations are possible**.

Elliptic curve groups support addition but not multiplication. If the proof vector $\vec{\pi}$ is encoded as group elements $[{\pi_i}] = \pi_i \cdot G$, the verifier can compute $[\pi(q)] = \sum_i q_i [\pi_i]$ (a linear combination) but cannot compute arbitrary functions of the proof. The prover, who knows the discrete logs, can create the encoding. The verifier, who doesn't, is restricted to linear queries. The group itself enforces the "linear PCP" constraint.

Groth16 (Chapter 12) is built on linear PCPs. The prover's messages are linear combinations of structured reference string elements, which are themselves encodings of powers of a secret. The verifier checks linear relationships via pairings: bilinear maps that allow *one* multiplication in the exponent, just enough to check quadratic constraints.

### From Proof Models to SNARKs

All modern SNARKs arise from one of these proof models combined with cryptographic compilation:

| Proof Model | + Cryptography | = SNARK Family |
|-------------|----------------|----------------|
| IP / Sum-check | + Polynomial Commitments | Spartan, HyperPlonk |
| IOP (polynomial) | + KZG / FRI | PLONK, Marlin, STARKs |
| Linear PCP | + Pairings | Groth16, BCTV14 |
| PCP | + Merkle trees | Kilian-style arguments |

The pattern: start with an information-theoretically secure protocol, then use cryptography to make the prover's messages short and binding.

**Polynomial Commitment Schemes** (Chapter 9-10) instantiate IOP oracles: the prover commits to a polynomial, the verifier queries evaluations, and a short proof demonstrates correctness of each evaluation.

**Fiat-Shamir** (Chapter 11) eliminates interaction: derive the verifier's challenges from hashes of the transcript. The prover computes the entire interaction locally and outputs a static proof.

The combination yields **SNARKs**: Succinct Non-interactive Arguments of Knowledge. A SNARK for a computation of size $n$ has:

- Proof size: $O(\log n)$ or even $O(1)$

- Verification time: $O(\log n)$ or $O(1)$

- Prover time: $O(n \log n)$ or similar quasi-linear

The asymmetry is achieved. Verification is exponentially cheaper than computation.



## Zero-Knowledge: Proving Without Revealing

There's another dimension to this story. So far, we've focused on *soundness*: preventing false claims from being verified. But what about *privacy*?

Suppose you want to prove you know a password without revealing the password itself. Or that you have sufficient funds for a transaction without revealing your balance. Or that you satisfy some credential requirement without exposing your identity.

**Zero-knowledge proofs** achieve exactly this. The proof convinces the verifier that the statement is true, but reveals *nothing* beyond this single bit of information. The verifier learns "yes, this is true" and nothing else.

The formal definition involves a *simulator*: an algorithm that produces transcripts indistinguishable from real proof transcripts, without access to the secret witness. If such a simulator exists, the proof is zero-knowledge; the transcript could have been generated by someone who didn't know the secret, so the transcript cannot leak the secret.

Zero-knowledge adds a layer of privacy to succinct verification. Together, they form **zkSNARKs**: Zero-Knowledge Succinct Non-interactive Arguments of Knowledge.



## The Architecture of Modern Proofs

This book develops the theory and practice of zkSNARKs. The architecture has emerged from decades of research, but it follows a consistent pattern:

**1. Arithmetization** (Chapters 4-8): Convert the computational claim into algebraic form. A program becomes a circuit. A circuit becomes a system of polynomial equations. The claim "I computed correctly" becomes "these polynomials satisfy this identity."

**2. Information-Theoretic Protocol** (Chapters 3, 7): Design an interactive protocol where the prover sends polynomials (or claims about polynomials) and the verifier checks them via random evaluations. This protocol is sound against unbounded provers; no cryptographic assumptions yet.

**3. Cryptographic Compilation** (Chapters 6, 9-10): Replace the abstract polynomials with cryptographic commitments. The prover commits to polynomials before seeing challenges. Polynomial commitment schemes (KZG, FRI, IPA) provide this binding.

**4. Fiat-Shamir Transform** (Chapter 11): Eliminate interaction. The verifier's random challenges are derived from a hash of the transcript. The prover computes the entire interaction locally and outputs a static proof.

The result: a proof that anyone can verify, that reveals nothing about the witness, and that is exponentially smaller than the computation it attests to.



## Why This Matters

Each application is a trust assumption eliminated.

**Verifiable computation** removes trust in the cloud. You outsource to untrusted servers, receive a proof with the result, and verify cheaply. The server's incentives, security practices, and internal controls become irrelevant. You don't trust the server; you verify the proof.

**Blockchain scalability** removes trust in centralized sequencers. Layer 2 solutions process thousands of transactions off-chain, producing a single proof that the main chain verifies. The sequencer cannot lie about execution. Transaction throughput increases by orders of magnitude without introducing new trust assumptions.

**Privacy-preserving credentials** remove trust in identity intermediaries. Prove you're over 21 without revealing your birthdate. Prove you passed a background check without revealing what was checked. The verifier learns exactly one bit: valid or not. No data broker, no identity provider, no linkable trail.

**Computational integrity** removes trust in institutions. Scientific simulations, machine learning inference, financial calculations—any computation can be accompanied by a proof of correctness. The question changes from "do I trust this organization?" to "does this proof verify?"

The pattern is consistent: find a trust assumption, replace it with mathematics.



## The Road Ahead

The chapters that follow develop this technology piece by piece.

We begin with polynomials (Chapter 2), the universal language of algebraic proof systems. The sum-check protocol (Chapter 3) shows how to verify exponential sums in polynomial time, the foundational technique underlying almost everything that follows.

Multilinear extensions (Chapter 4) and univariate polynomials (Chapter 5) provide two complementary encoding schemes for computational data. Commitment schemes (Chapter 6) bind provers to their claims.

The GKR protocol (Chapter 7) verifies arbitrary circuits using sum-check. Arithmetization (Chapter 8) shows how real computations become circuits.

Polynomial commitment schemes (Chapters 9-10) provide the cryptographic foundation: KZG, IPA, and FRI, each with different trade-offs between proof size, verification time, and trust assumptions.

The SNARK recipe (Chapter 11) explains how these pieces assemble. Groth16 (Chapter 12), PLONK (Chapter 13), lookup arguments (Chapter 14), and STARKs (Chapter 15) are complete systems, each optimizing different aspects.

$\Sigma$-protocols (Chapter 16) and zero-knowledge (Chapters 17-18) add privacy. The sum-check renaissance (Chapters 19-21) develops the latest techniques for fast proving.

Composition and recursion (Chapter 22) enable proofs about proofs: unlimited computation with constant verification. The book concludes with system selection guidance (Chapter 23), MPC's parallel path (Chapter 24), open frontiers (Chapter 25), and the broader cryptographic landscape (Chapter 26).

By the end, you'll understand not just what zkSNARKs do, but *how* they work: the mathematical structures that make the impossible possible.



## Key Takeaways

1. **The trust problem**: Verifying computation naively requires redoing the work. This is useless for outsourced computation.

2. **Interactive proofs unlock verification asymmetry**: Random challenges let verifiers check claims they couldn't compute themselves. IP = PSPACE shows the power of interaction.

3. **Schwartz-Zippel is the key tool**: Random polynomial evaluation catches cheating with overwhelming probability.

4. **The proof system zoo**: IP → MIP → PCP → IOP → Linear PCP. Each model trades off interaction, query access, and algebraic structure differently.

5. **The PCP theorem is foundational**: NP = PCP[O(log n), O(1)]. Any NP statement has a proof checkable by reading constantly many bits.

6. **IOPs combine interaction and oracles**: The prover sends oracles (polynomials), the verifier queries and challenges. This is what modern SNARKs actually do.

7. **Cryptographic compilation makes it practical**: Polynomial commitments instantiate oracles. Fiat-Shamir eliminates interaction. Proofs become short and non-interactive.

8. **Zero-knowledge adds privacy**: Prove without revealing. The transcript leaks nothing beyond validity.

9. **The architecture is modular**: Arithmetization → Information-theoretic protocol (IOP) → Cryptographic compilation (PCS) → Fiat-Shamir.

10. **Different proof models yield different SNARKs**: Linear PCPs → Groth16. Polynomial IOPs → PLONK, STARKs. Sum-check IPs → Spartan.
