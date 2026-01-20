# Chapter 17: The Zero-Knowledge Property

What does it mean to prove something without revealing how you proved it?

The question sounds paradoxical. A proof, by its nature, is a demonstration: an argument that convinces by showing. The verifier sees the proof and becomes convinced. How can seeing suffice for conviction while simultaneously revealing nothing?

The answer lies in distinguishing *what* the verifier learns from *what* the verifier sees. The verifier sees a transcript: a sequence of messages exchanged with the prover. The verifier learns (ideally) one bit: the statement is true. The zero-knowledge property formalizes the claim that nothing more than this single bit leaks.

Consider the stakes. You hold a private key that controls valuable assets. You want to prove you possess this key (to authenticate to a service, to sign a transaction, to unlock a system) without exposing the key itself. A naive proof ("here is the key") achieves authentication but destroys privacy. A zero-knowledge proof achieves both: the verifier becomes certain you know the key, yet learns nothing that would help them compute it.

This chapter develops the formal definition of zero-knowledge and explores its subtleties. The definition turns on a beautiful thought experiment: what if the verifier could have generated the entire proof transcript by themselves, without any prover present? If so, the transcript cannot have leaked anything; there's nothing in it the verifier couldn't have produced alone.

## The Simulation Argument

Chapter 16 introduced the Schnorr protocol simulator. The idea was almost too simple: to produce a valid transcript $(a, e, z)$ without knowing the witness $w$, pick $e$ and $z$ first (both uniformly random), then compute $a = g^z h^{-e}$. The transcript satisfies the verification equation by construction, and its distribution matches a real transcript exactly.

This is the **simulation paradigm**: a proof system is zero-knowledge if a simulator (an efficient algorithm with no access to the witness) can produce transcripts indistinguishable from real protocol executions.

Why does simulation imply privacy? Suppose the verifier could extract some information $I$ about the witness from a real transcript. Consider the simulator's output. The simulator doesn't know the witness, so its transcript cannot possibly encode $I$. But the simulator's transcript is indistinguishable from the real one. If the verifier can extract $I$ from real transcripts, they should also extract $I$ from simulated ones; yet simulated transcripts don't contain $I$. Contradiction. Therefore, real transcripts don't leak $I$ either.

The logic is subtle: we prove that real transcripts leak nothing by showing they're indistinguishable from transcripts that *obviously* leak nothing (because the simulator never had the secret).

There is something strange here. The proof is convincing precisely because it could have been fabricated. The simulator (who knows nothing) produces output identical to the prover who knows everything. This indistinguishability is not a flaw to be patched; it is the definition of success. The guarantee of privacy *is* the guarantee that a fake would be undetectable.


## The Graph Non-Isomorphism Protocol

The Schnorr protocol is too simple to fully illustrate simulation; the simulator's trick (compute $a$ from $e, z$) might seem like algebraic coincidence. Let's examine a more intuitive example: the Graph Non-Isomorphism protocol from Chapter 1.

**The setting.** Two graphs $G_0$ and $G_1$ are claimed to be non-isomorphic: no relabeling of vertices makes them identical. There's no obvious short certificate for this claim. The negative claim seems to require checking all $n!$ possible relabelings.

**The protocol.** The verifier picks a secret bit $b \in \{0, 1\}$, applies a random permutation $\pi$ to $G_b$, and sends $H = \pi(G_b)$ to the prover. The prover's task: identify which graph $H$ came from. If the graphs are truly non-isomorphic, they have different structural invariants (triangle counts, eigenvalue spectra, degree distributions). An unbounded prover computes these invariants and determines $b$ with certainty. The prover sends back $b' = b$.

**What does the verifier see?** After a successful execution:

- The challenge $H$ that she generated herself
- The bit $b'$ that matches her secret $b$

But wait: $b$ was her own random choice. $H$ was her own computation. The prover's response $b' = b$ just echoes her own randomness back. The transcript $(H, b')$ contains nothing the verifier didn't already know.

**The simulator.** Given only the graphs $G_0, G_1$ (not the prover's ability to distinguish them):

1. Pick $b \leftarrow \{0, 1\}$ uniformly at random
2. Pick $\pi$ uniformly from permutations of the vertex set
3. Compute $H = \pi(G_b)$
4. Output the transcript $(H, b)$

This is exactly what an honest verifier would see in a real execution. The simulator "plays both roles," generating both the verifier's message and the prover's response, and produces an identical distribution.

**Perfect zero-knowledge.** The simulated and real distributions are not merely close; they're identical. This is **perfect zero-knowledge**: the statistical distance between real and simulated transcripts is exactly zero.



## Formal Definition

Let $(\mathcal{P}, \mathcal{V})$ be an interactive proof system for a language $\mathcal{L}$. On input $x \in \mathcal{L}$, the prover $\mathcal{P}$ holds a witness $w$; the verifier $\mathcal{V}$ sees only $x$.

**The verifier's view** consists of:

1. The statement $x$
2. The verifier's random coins $r$
3. All messages received from the prover

We write $\text{View}_{\mathcal{V}}(\mathcal{P}(w) \leftrightarrow \mathcal{V})(x)$ for this random variable.

**Definition (Zero-Knowledge).** The proof system is **zero-knowledge** if there exists a probabilistic polynomial-time algorithm $\mathcal{S}$ (the simulator) such that for all $x \in \mathcal{L}$:

$$\text{View}_{\mathcal{V}}(\mathcal{P}(w) \leftrightarrow \mathcal{V})(x) \approx \mathcal{S}(x)$$

The symbol $\approx$ denotes indistinguishability; its precise meaning yields three flavors.



## Three Flavors of Zero-Knowledge

**Perfect zero-knowledge (PZK).** The distributions are identical:
$$\text{View}_{\mathcal{V}} \equiv \mathcal{S}(x)$$

No adversary, even with unlimited computational power, can distinguish real from simulated transcripts. The two distributions have zero statistical distance.

This is the strongest notion. The Schnorr protocol (Chapter 16) achieves PZK against honest verifiers: the simulator's output $(a, e, z)$ has exactly the same distribution as a real transcript.

**Statistical zero-knowledge (SZK).** The distributions are statistically close:
$$\Delta(\text{View}_{\mathcal{V}}, \mathcal{S}(x)) \leq \text{negl}(\lambda)$$

The statistical distance is negligible in the security parameter $\lambda$. An unbounded adversary might distinguish the distributions, but only with probability $2^{-\Omega(\lambda)}$ (effectively never).

SZK allows for protocols where perfect simulation is impossible but the gap is cryptographically small. Many commitment-based protocols achieve SZK.

**Computational zero-knowledge (CZK).** No efficient algorithm can distinguish the distributions:
$$\text{View}_{\mathcal{V}} \stackrel{c}{\approx} \mathcal{S}(x)$$

The distributions might be statistically far apart, but every polynomial-time distinguisher's advantage is negligible. Security relies on computational hardness; an unbounded adversary could distinguish.

CZK is the weakest but most practical notion. Modern SNARKs typically achieve CZK. The simulator might use pseudorandom values where the real protocol uses true randomness; distinguishing requires breaking the underlying assumption.


## Honest Verifiers and Malicious Verifiers

The definition above assumes the verifier follows the protocol honestly. What if she doesn't?

**Honest-verifier zero-knowledge (HVZK).** The simulator produces indistinguishable output when the verifier $\mathcal{V}$ follows the protocol specification exactly; in particular, it samples challenges uniformly at random.

This is what Schnorr's protocol achieves. The simulator works because it knows the honest verifier will choose $e$ uniformly. If the verifier could choose $e$ adversarially, based on the prover's commitment $a$, the simulator's technique breaks.

**Malicious-verifier zero-knowledge.** The simulator must produce indistinguishable output against *any* efficient verifier strategy $\mathcal{V}^*$, including:

- Adversarial challenge selection
- Auxiliary information from other sources
- Arbitrary protocol deviations

Consider the Graph Non-Isomorphism protocol again. An honest verifier sends $H = \pi(G_b)$ for her secret $b$. But a malicious verifier could send some other graph $H'$ (perhaps one she suspects is isomorphic to $G_0$ but isn't sure). The all-powerful prover will correctly identify whether $H'$ matches $G_0$, $G_1$, or neither. The verifier learns something she couldn't efficiently compute herself!

The protocol is HVZK but not malicious-verifier ZK. The prover, dutifully answering whatever question is posed, inadvertently becomes an oracle for graph isomorphism.

**Closing the gap.** Transforming HVZK protocols into malicious-verifier ZK requires additional machinery:

- *Coin-flipping protocols* force the verifier to commit to her randomness before seeing the prover's messages. The verifier's challenges become unpredictable even to her.
- *Trapdoor commitments* let the simulator "equivocate": commit to one value, then open to another after seeing the verifier's behavior.
- *The Fiat-Shamir transform* eliminates interaction entirely. With no verifier messages, there's no room for malicious behavior. The simulator controls the random oracle and programs it as needed.

Non-interactive proofs (after Fiat-Shamir) largely dissolve the HVZK/malicious distinction. The "verifier" merely checks a static proof string.


## The Simulator's Superpower: Rewinding

How can a simulator, without the witness, produce valid-looking transcripts? The answer involves a capability the real prover lacks: **rewinding**.

In a real protocol execution, time moves forward. The prover commits to $a$, then receives challenge $e$, then computes response $z$. The commitment precedes the challenge. The prover cannot see $e$ before sending $a$.

The simulator isn't bound by temporal order. It produces a transcript (a static object) not a live interaction. It can:

1. Choose $e$ first (pretending to know the future)
2. Compute $z$ however convenient
3. Work backward to find $a$ consistent with both

This is precisely what the Schnorr simulator does: pick $e, z$ first, compute $a = g^z h^{-e}$. The transcript $(a, e, z)$ looks like a real interaction, but it was computed backward.

For more complex protocols, the simulator may need to "run" the verifier multiple times, recording responses to different challenges and picking the right one. This is **rewinding**: the simulator rewinds the verifier to an earlier state and tries again with different randomness.

Rewinding is a proof technique, not a real capability. It demonstrates that the transcript could have been generated without the witness. Real provers cannot rewind real verifiers; they face a single, forward-moving timeline. But the simulator's ability to rewind shows that the information content of the transcript is not tied to the witness.

## The Central Confusion

Students encountering zero-knowledge often stumble on this point: *if the simulator can produce valid transcripts without the witness, what stops a cheater from doing the same?*

The answer is subtle but crucial.

A cheating prover and a simulator operate under different rules:

| Cheating Prover | Simulator |
|----------------|-----------|
| Interacts in real time | Produces a transcript offline |
| Commits before seeing challenge | Can choose challenge first |
| Cannot rewind the verifier | Can run the verifier many times |
| Must work for false statements | Only needs to work for true statements |

The cheating prover faces a live verifier who sends unpredictable challenges. The prover commits to $a$, then receives $e$, then must produce $z$. Without the witness, the prover cannot know in advance which $e$ will come. They must commit to a strategy that works for *all* (or most) possible challenges; by soundness, this is impossible for false statements.

The simulator faces a different task: produce a single transcript that *looks* like a real interaction. It can pick the challenge first, then reverse-engineer the commitment. This works precisely because the simulator knows the statement is true (simulation is only required for $x \in \mathcal{L}$).

**Soundness is about real interaction.** Zero-knowledge is about information content. The simulator's success shows the transcript contains no extractable information about the witness. It doesn't help a cheating prover because the cheater faces a different game: one where they can't rewind, can't choose challenges, and must work on false statements.


## The Limits of Zero-Knowledge

Perfect and statistical zero-knowledge seem strictly stronger than computational. Are they always preferable?

No. There are fundamental limits.

**Theorem (Fortnow, Aiello-Håstad).** Any language with a statistical zero-knowledge proof lies in $\text{AM} \cap \text{coAM}$.

The class $\text{AM}$ (Arthur-Merlin) is roughly IP with public coins. The class $\text{AM} \cap \text{coAM}$ is believed to be much smaller than NP. In particular, it likely contains no NP-complete problems.

**Implication.** If you want statistical zero-knowledge proofs for NP-complete problems, you're out of luck (assuming standard complexity-theoretic conjectures).

The way forward is to relax both soundness and zero-knowledge:

- **Computational soundness (arguments):** Security against cheating provers who are computationally bounded.
- **Computational zero-knowledge:** Security against distinguishers who are computationally bounded.

Modern SNARKs take both paths. They are *arguments* (computationally sound) with *computational zero-knowledge*. This combination enables practical ZK proofs for arbitrary computations, including NP-complete problems and beyond.

## Zero-Knowledge in the Wild: Sum-Check

Let's ground this in the core protocol of the book. The sum-check protocol proves:

$$H = \sum_{b \in \{0,1\}^n} g(b)$$

In each round, the prover sends a univariate polynomial $g_i(X_i)$, the restriction of $g$ to a partial evaluation. The verifier checks degree bounds and eventually evaluates $g$ at a random point.

**Is sum-check zero-knowledge?** Not inherently. The univariate polynomials $g_i$ reveal partial information about $g$. If $g$ encodes secret witness data, this information leaks.

For applications where $g$ is derived from public inputs (verifiable computation on public data), this leakage is harmless. For private-witness applications, we need modifications.

**Masking techniques** (Chapter 18) add zero-knowledge to sum-check:

- Add random low-degree polynomials that cancel in the sum
- Commit to intermediate values instead of revealing them
- Use randomization to hide the structure of $g$

The key insight: zero-knowledge is a *system-level* property, not a per-protocol property. We can compose non-ZK building blocks (sum-check, FRI, polynomial commitments) into ZK systems by carefully controlling what the verifier sees.


## Proofs of Knowledge

Zero-knowledge concerns what the verifier learns. A related but distinct property concerns what the prover demonstrates.

**Proof of existence:** "There exists $w$ such that $R(x, w) = 1$."

The prover demonstrates the statement is true (a witness exists) without necessarily revealing or even knowing the witness.

**Proof of knowledge:** "I know $w$ such that $R(x, w) = 1$."

The prover demonstrates not just existence but *possession*. This requires an additional property: **knowledge extraction**.

**Definition.** A proof system has knowledge extraction if there exists an efficient extractor $\mathcal{E}$ such that: whenever a (possibly cheating) prover $\mathcal{P}^*$ convinces the verifier, $\mathcal{E}^{\mathcal{P}^*}$ (with oracle access to $\mathcal{P}^*$) extracts a valid witness $w$.

The extractor typically works by rewinding. It runs the prover once, records the response to challenge $e_1$, rewinds, gives challenge $e_2$, and extracts the witness from the two transcripts. This is exactly what special soundness (Chapter 16) provides for $\Sigma$-protocols.

**Zero-knowledge proofs of knowledge** combine both properties. The prover demonstrates possession of a secret without revealing it. This is the foundation of digital signatures (prove you know the signing key), anonymous credentials (prove you possess a valid credential), and confidential transactions (prove you know the secret amounts that balance).



## Auxiliary Input

Composition complicates things. When a ZK proof is used as a subroutine in a larger protocol, the "verifier" in the subroutine may have learned information from earlier stages.

**Definition (Auxiliary-Input ZK).** A protocol is auxiliary-input zero-knowledge if for every efficient verifier $\mathcal{V}^*$ with auxiliary input $z$:

$$\text{View}_{\mathcal{V}^*(z)}(\mathcal{P}(w) \leftrightarrow \mathcal{V}^*(z))(x) \approx \mathcal{S}(x, z)$$

The simulator receives the same auxiliary input $z$ as the verifier.

This definition handles composed protocols. Even if the verifier has side information about the statement or witness, the proof reveals nothing new. The simulator, given the same side information, produces indistinguishable transcripts.

Auxiliary-input ZK is essential for security in complex systems where many proofs interleave.


## Key Takeaways

1. **Zero-knowledge** means existence of a simulator: an efficient algorithm that produces transcripts indistinguishable from real executions, without access to the witness.

2. **The simulation argument** shows that if real and simulated transcripts are indistinguishable, real transcripts leak nothing; they contain no information the verifier couldn't generate alone.

3. **Three flavors**: Perfect (identical distributions), Statistical (negligible statistical distance), Computational (no efficient distinguisher).

4. **HVZK vs. malicious-verifier ZK**: HVZK only protects against honest verifiers; malicious-verifier ZK protects against adversarial verifier strategies. Non-interactive proofs largely collapse this distinction.

5. **The simulator's superpower is rewinding**: choosing challenges before commitments, trying multiple paths. Real provers cannot rewind; this is why simulation doesn't break soundness.

6. **The central confusion resolved**: Simulators and cheating provers play different games. The simulator works offline on true statements; the cheating prover faces live interaction on false statements.

7. **Limits of SZK**: Statistical zero-knowledge proofs exist only for languages in AM ∩ coAM, likely not NP-complete problems. Computational ZK sidesteps this barrier.

8. **Proofs of knowledge** add extraction: the prover demonstrates possession, not just existence. Zero-knowledge proofs of knowledge enable proving you know a secret without revealing it.

9. **Sum-check isn't inherently ZK**: The intermediate polynomials leak information. Masking techniques (Chapter 18) restore privacy.

10. **Auxiliary-input ZK** handles composed protocols where the verifier has side information. The simulator receives the same auxiliary input and still produces indistinguishable transcripts.
