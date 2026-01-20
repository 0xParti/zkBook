# Chapter 9: Polynomial Commitment Schemes: The Cryptographic Engine

In 2010, Aniket Kate, a graduate student at the University of Waterloo, was thinking about a gap between theory and practice.

Interactive proof systems had shown that polynomials were the key to verification: checking a polynomial at a random point could catch cheaters with overwhelming probability. But these protocols assumed the verifier could somehow "see" the polynomial. In practice, the polynomial was huge. Sending it would defeat the purpose of succinct proofs.

Kate, together with Gregory Zaverucha and Ian Goldberg, asked: what if we could *commit* to a polynomial without revealing it, then later *prove* individual evaluations without exposing anything else?

Their answer was KZG, a scheme where the commitment is a single group element (32 bytes), and each evaluation proof is also a single group element. No matter whether the polynomial has 10 coefficients or 10 million, the commitment stays the same size. This was the missing piece that made practical SNARKs possible.

---

Everything we've built (sum-check, GKR, arithmetization) reduces complex claims to polynomial identities. A prover claims that polynomial $p(X)$ has certain properties: it equals another polynomial, it vanishes on a domain, it evaluates to a specific value at a point.

But here's the catch: verifying these claims directly would require the verifier to see the entire polynomial. For a polynomial of degree $n$, that's $n+1$ coefficients, exactly as much data as the original computation. We've achieved nothing.

**Polynomial Commitment Schemes (PCS)** solve this problem. A PCS allows a prover to commit to a polynomial with a short commitment, then later prove claims about the polynomial (its evaluations at specific points) without revealing the polynomial itself. The commitment is binding (the prover can't change the polynomial), and the proofs are succinct (much smaller than the polynomial).

This is where abstract algebra meets cryptography. This chapter explores two fundamental approaches: **KZG** (using pairings) and **IPA/Bulletproofs** (using discrete logarithms).



## The PCS Abstraction

A polynomial commitment scheme consists of three algorithms:

**Commit $(f) \to C$**: Given a polynomial $f(X)$, produce a short commitment $C$.

**Open $(f, z) \to (v, \pi)$**: Given the polynomial $f$, a point $z$, compute the evaluation $v = f(z)$ and a proof $\pi$ that this evaluation is correct.

**Verify $(C, z, v, \pi) \to \{\text{accept}, \text{reject}\}$**: Given the commitment, point, claimed value, and proof, check correctness.

**Properties**:

- **Binding**: A commitment $C$ can only be opened to evaluations consistent with *one* polynomial (computationally)
- **Hiding** (optional): The commitment reveals nothing about the polynomial
- **Succinctness**: Commitments and proofs are much smaller than the polynomial

The key insight: if the prover is bound to a specific polynomial before seeing the verifier's challenge point, and the commitment is much smaller than the polynomial, then we can verify polynomial identities by checking at random points.



## KZG: Constant-Size Proofs from Pairings

The Kate-Zaverucha-Goldberg (KZG) scheme achieves the holy grail: constant-size commitments *and* constant-size evaluation proofs. No matter how large the polynomial, the proof is just one group element.

### The Magic Ingredient: Pairings

A **bilinear pairing** is a map $e: G_1 \times G_2 \to G_T$ between three groups with the property:

$$e(aP, bQ) = e(P, Q)^{ab}$$

for all scalars $a, b$ and group elements $P \in G_1$, $Q \in G_2$.

This seemingly simple equation has profound consequences. It allows us to check multiplicative relationships *in the exponent*. Given commitments $g^a$ and $g^b$, we cannot compute $g^{ab}$ (that would break CDH). But we can *verify* that $g^c = g^{ab}$ by checking:

$$e(g^a, g^b) = e(g^c, g)$$

One multiplication check "for free" in the hidden exponent world. This is exactly what polynomial evaluation needs.

### The Trusted Setup

KZG requires a **structured reference string (SRS)**: a set of public parameters computed from a secret:

1. Choose a random secret $\tau \in \mathbb{F}_p$ (the "toxic waste")
2. Compute the SRS: $(g, g^\tau, g^{\tau^2}, \ldots, g^{\tau^D})$
3. **Destroy** $\tau$

The SRS encodes powers of the secret $\tau$ "in the exponent." Anyone can use these elements without knowing $\tau$ itself.

**The critical security requirement**: If anyone learns $\tau$, they can forge proofs for false statements. The setup must ensure $\tau$ is never known to any party. In practice, this is done via multi-party computation ceremonies where many participants contribute randomness, and security holds as long as *any one* participant is honest.

### Commitment

To commit to a polynomial $f(X) = \sum_{i=0}^{d} c_i X^i$:

$$C = g^{f(\tau)} = g^{\sum c_i \tau^i} = \prod_{i=0}^{d} (g^{\tau^i})^{c_i}$$

The prover computes this using the SRS elements, never learning $\tau$. The result is a single group element: the polynomial "evaluated at the secret point $\tau$, hidden in the exponent."

### Evaluation Proof

To prove $f(z) = v$ for a public point $z$:

1. **The polynomial identity**: If $f(z) = v$, then $(X - z)$ divides $f(X) - v$. Define:
   $$w(X) = \frac{f(X) - v}{X - z}$$
   This quotient $w(X)$ is a valid polynomial of degree $d-1$.

2. **The proof**: Commit to the quotient:
   $$\pi = g^{w(\tau)}$$

3. **Verification**: The verifier checks:
   $$e(\pi, g^\tau \cdot g^{-z}) = e(C \cdot g^{-v}, g)$$

### Why Verification Works

The pairing check verifies that the polynomial identity holds at $\tau$:

$$e(g^{w(\tau)}, g^{\tau - z}) = e(g^{f(\tau) - v}, g)$$

By bilinearity:
$$e(g,g)^{w(\tau)(\tau - z)} = e(g,g)^{f(\tau) - v}$$

This holds iff $w(\tau)(\tau - z) = f(\tau) - v$, which is exactly the polynomial identity $f(X) - v = w(X)(X - z)$ evaluated at $\tau$.

**Why this implies soundness**: Suppose the prover lies; they claim $f(z) = v$ when actually $f(z) \neq v$. Then $f(X) - v$ is *not* divisible by $(X - z)$, so no polynomial $w(X)$ satisfies the identity $f(X) - v = w(X)(X - z)$. Without such a $w(X)$, the prover must instead find some $w'(X)$ where the identity fails as polynomials but happens to hold at $\tau$:

$$w'(\tau)(\tau - z) = f(\tau) - v$$

But the prover doesn't know $\tau$; it's hidden in the SRS. From their perspective, $\tau$ is a random field element. Two distinct degree-$d$ polynomials agree on at most $d$ points (Schwartz-Zippel), so the probability that a "wrong" $w'$ accidentally satisfies the check at the unknown $\tau$ is at most $d/|\mathbb{F}|$ (negligible for large fields).

The logic: if you can pass the check at a *random* point without knowing which point, you must have the correct polynomial identity.



## Worked Example: KZG in Action

Let's trace through a complete example.

**Setup**: Maximum degree $D = 2$, secret $\tau = 5$.

- SRS: $(g, g^5, g^{25})$

**Commit to $f(X) = X^2 + 2X + 3$**:
$$C = g^{f(5)} = g^{25 + 10 + 3} = g^{38}$$

**Prove $f(1) = 6$**:

- Check: $f(1) = 1 + 2 + 3 = 6$
- Quotient: $w(X) = \frac{f(X) - 6}{X - 1} = \frac{X^2 + 2X - 3}{X - 1}$

  Factor: $X^2 + 2X - 3 = (X + 3)(X - 1)$

  So $w(X) = X + 3$

- Proof: $\pi = g^{w(5)} = g^{5 + 3} = g^8$

**Verify**:

- LHS: $e(\pi, g^\tau \cdot g^{-z}) = e(g^8, g^5 \cdot g^{-1}) = e(g^8, g^4) = e(g,g)^{32}$
- RHS: $e(C \cdot g^{-v}, g) = e(g^{38} \cdot g^{-6}, g) = e(g^{32}, g) = e(g,g)^{32}$

Both sides equal. The verification passes.

### Managing Toxic Waste: Powers of Tau Ceremonies

The trusted setup creates a serious practical problem: someone must generate τ, compute the powers, and then *verifiably destroy* τ. How do you convince the world that the toxic waste is truly gone?

The solution is **multi-party computation (MPC) ceremonies**. Instead of trusting a single party, we chain together contributions from many independent participants:

1. **Participant 1** picks secret $\tau_1$, computes $[1]_1, [\tau_1]_1, [\tau_1^2]_1, \ldots$ and destroys $\tau_1$
2. **Participant 2** picks secret $\tau_2$, raises each element to $\tau_2$, getting $[1]_1, [\tau_1\tau_2]_1, [(\tau_1\tau_2)^2]_1, \ldots$ and destroys $\tau_2$
3. Continue for hundreds or thousands of participants...

The final structured reference string encodes powers of $\tau = \tau_1 \cdot \tau_2 \cdot \tau_3 \cdots \tau_n$. The crucial insight: **the setup is secure if *any single participant* destroyed their secret**. This is the "1-of-N" trust model; you only need to trust that *one* honest participant existed among potentially thousands.

**Notable ceremonies**:

- **Zcash Powers of Tau** (2017-2018): 87 participants contributed to a universal phase, followed by circuit-specific ceremonies for Sapling
- **Ethereum KZG Ceremony** (2023): Over 140,000 contributions for the EIP-4844 blob commitments, the largest ceremony ever conducted

**Universal vs. circuit-specific**: Some ceremonies produce parameters usable for any circuit up to a size bound (universal), while others are tailored to specific circuits. KZG setups are inherently universal; the same powers of tau work for any polynomial of degree at most $d$.

The scale of modern ceremonies makes collusion effectively impossible. When 140,000 independent participants contribute, the probability that *all* of them colluded or were compromised approaches zero.

## KZG: Properties and Trade-offs

**Advantages**:

- **Constant commitment size**: One group element, regardless of polynomial degree
- **Constant proof size**: One group element per evaluation
- **Constant verification time**: A few pairings and exponentiations
- **Batch verification**: Multiple evaluations can be verified together

**Disadvantages**:

- **Trusted setup**: The "toxic waste" must be destroyed. If compromised, soundness breaks.
- **Not post-quantum**: Pairing-based cryptography falls to quantum computers
- **Non-universal** (in basic form): The SRS size limits the maximum polynomial degree

### Batch Opening

KZG has a remarkable property: proving evaluations at multiple points is barely more expensive than proving one.

To prove $f(z_1) = v_1, \ldots, f(z_k) = v_k$:

1. Define the vanishing polynomial $Z(X) = \prod_i (X - z_i)$
2. Compute the interpolating polynomial $R(X)$ such that $R(z_i) = v_i$
3. The quotient $w(X) = \frac{f(X) - R(X)}{Z(X)}$ exists iff all evaluations are correct
4. The proof is just $g^{w(\tau)}$ (still one group element!)



## IPA/Bulletproofs: No Trusted Setup

**Historical context**: The Inner Product Argument emerged from a different lineage than KZG. Bootle et al. (2016) introduced the core folding technique for efficient inner product proofs. Bünz et al. (2017) refined this into **Bulletproofs**, originally designed for *range proofs*, proving that a committed value lies in a range $[0, 2^n)$ without revealing it. This was motivated by confidential transactions in cryptocurrencies: prove your balance is non-negative without revealing the amount.

The terminology can be confusing:

- **IPA** (Inner Product Argument) is the *technique*: the recursive folding protocol that proves $\langle \vec{a}, \vec{b} \rangle = c$
- **Bulletproofs** is the *system* that used IPA for range proofs and general arithmetic circuits
- **Halo** (2019, Bowe et al.) showed how to use IPA for polynomial commitments in recursive proof composition, avoiding trusted setup entirely

Today, "IPA" and "Bulletproofs" are often used interchangeably to describe the folding-based polynomial commitment scheme. The key innovation: achieving transparency (no toxic waste) at the cost of logarithmic proofs and linear verification.

### The Key Insight: Polynomial Evaluation as Inner Product

A polynomial evaluation is an inner product:

$$f(z) = \sum_{i=0}^{n-1} c_i z^i = \langle \vec{c}, \vec{z} \rangle$$

where $\vec{c} = (c_0, \ldots, c_{n-1})$ are coefficients and $\vec{z} = (1, z, z^2, \ldots, z^{n-1})$ is the evaluation vector.

If we can prove inner product claims efficiently, we can prove polynomial evaluations!

### Pedersen Vector Commitments

The basic Pedersen vector commitment uses generators $G_0, \ldots, G_{n-1}$ for coefficients and $H$ for blinding:

$$C = \sum_{i=0}^{n-1} c_i \cdot G_i + r \cdot H$$

For polynomial evaluation proofs, we need an additional generator $U$ to encode the claimed inner product value. The full commitment for proving $\langle \vec{c}, \vec{z} \rangle = v$ becomes:

$$P = \langle \vec{c}, \vec{G} \rangle + v \cdot U + r \cdot H$$

This structure is essential: the generator $U$ carries the inner product term through the folding process, allowing the cross-term commitments to properly encode both coefficient and inner product information.

### The Folding Trick

The brilliant idea of IPA is recursive "folding" that shrinks the problem by half each round.

**Setup**: Prover holds coefficient vector $\vec{c}$ of length $n$. They've committed to it as $P = \langle \vec{c}, \vec{G} \rangle + v \cdot U$ where $v = \langle \vec{c}, \vec{z} \rangle$ is the claimed evaluation. (We omit the blinding term $rH$ for clarity.)

**One round of folding**:

1. Split $\vec{c} = (\vec{c}_L, \vec{c}_R)$ into two halves
2. Split $\vec{z} = (\vec{z}_L, \vec{z}_R)$ and $\vec{G} = (\vec{G}_L, \vec{G}_R)$ similarly
3. Prover computes and sends cross-term *commitments*:
   $$L = \langle \vec{c}_L, \vec{G}_R \rangle + \langle \vec{c}_L, \vec{z}_R \rangle \cdot U$$
   $$R = \langle \vec{c}_R, \vec{G}_L \rangle + \langle \vec{c}_R, \vec{z}_L \rangle \cdot U$$

   Note: $L$ commits to the left coefficients using right generators, plus the cross inner product. Similarly for $R$.
4. Verifier sends random challenge $\alpha$
5. **Prover** computes the folded coefficient vector (secretly):
   $$\vec{c}' = \alpha \cdot \vec{c}_L + \alpha^{-1} \cdot \vec{c}_R$$
6. **Both parties** compute (using public information):
   - Folded evaluation vector: $\vec{z}' = \alpha^{-1} \cdot \vec{z}_L + \alpha \cdot \vec{z}_R$
   - Folded generators: $\vec{G}' = \alpha^{-1} \cdot \vec{G}_L + \alpha \cdot \vec{G}_R$
   - Updated commitment: $P' = L^{\alpha^2} \cdot P \cdot R^{\alpha^{-2}}$

**Why this works**: We need to show that $P'$ is a valid commitment to $(\vec{c}', v')$ under the folded generators $\vec{G}'$.

First, expand what $P'$ *should* be if the prover is honest:
$$P'_{\text{honest}} = \langle \vec{c}', \vec{G}' \rangle + v' \cdot U$$

where $v' = \langle \vec{c}', \vec{z}' \rangle$ is the new inner product claim.

Now expand $\langle \vec{c}', \vec{G}' \rangle$ using the folding formulas:
$$\langle \vec{c}', \vec{G}' \rangle = \langle \alpha \vec{c}_L + \alpha^{-1} \vec{c}_R, \, \alpha^{-1} \vec{G}_L + \alpha \vec{G}_R \rangle$$

Distributing the inner product (which is bilinear):
$$= \alpha \cdot \alpha^{-1} \langle \vec{c}_L, \vec{G}_L \rangle + \alpha \cdot \alpha \langle \vec{c}_L, \vec{G}_R \rangle + \alpha^{-1} \cdot \alpha^{-1} \langle \vec{c}_R, \vec{G}_L \rangle + \alpha^{-1} \cdot \alpha \langle \vec{c}_R, \vec{G}_R \rangle$$
$$= \langle \vec{c}_L, \vec{G}_L \rangle + \langle \vec{c}_R, \vec{G}_R \rangle + \alpha^2 \langle \vec{c}_L, \vec{G}_R \rangle + \alpha^{-2} \langle \vec{c}_R, \vec{G}_L \rangle$$

Similarly, expanding the new inner product $v' = \langle \vec{c}', \vec{z}' \rangle$:
$$v' = \langle \vec{c}_L, \vec{z}_L \rangle + \langle \vec{c}_R, \vec{z}_R \rangle + \alpha^2 \langle \vec{c}_L, \vec{z}_R \rangle + \alpha^{-2} \langle \vec{c}_R, \vec{z}_L \rangle = v + \alpha^2 L_{\text{ip}} + \alpha^{-2} R_{\text{ip}}$$

Now look at $P' = L^{\alpha^2} \cdot P \cdot R^{\alpha^{-2}}$ and expand each term:

- $P = \langle \vec{c}_L, \vec{G}_L \rangle + \langle \vec{c}_R, \vec{G}_R \rangle + v \cdot U$
- $L = \langle \vec{c}_L, \vec{G}_R \rangle + L_{\text{ip}} \cdot U$
- $R = \langle \vec{c}_R, \vec{G}_L \rangle + R_{\text{ip}} \cdot U$

So:
$$L^{\alpha^2} \cdot P \cdot R^{\alpha^{-2}} = \alpha^2 L + P + \alpha^{-2} R$$
$$= \alpha^2 (\langle \vec{c}_L, \vec{G}_R \rangle + L_{\text{ip}} \cdot U) + (\langle \vec{c}_L, \vec{G}_L \rangle + \langle \vec{c}_R, \vec{G}_R \rangle + v \cdot U) + \alpha^{-2}(\langle \vec{c}_R, \vec{G}_L \rangle + R_{\text{ip}} \cdot U)$$

Collecting terms:
$$= \underbrace{\langle \vec{c}_L, \vec{G}_L \rangle + \langle \vec{c}_R, \vec{G}_R \rangle + \alpha^2 \langle \vec{c}_L, \vec{G}_R \rangle + \alpha^{-2} \langle \vec{c}_R, \vec{G}_L \rangle}_{= \langle \vec{c}', \vec{G}' \rangle} + \underbrace{(v + \alpha^2 L_{\text{ip}} + \alpha^{-2} R_{\text{ip}})}_{= v'} \cdot U$$

This equals $\langle \vec{c}', \vec{G}' \rangle + v' \cdot U = P'_{\text{honest}}$. The update formula produces exactly the right commitment!

**The recursion**: After $\log_2 n$ rounds, the vectors have length 1. The prover reveals the final scalar, and the verifier checks directly.

### Final Verification: The Endgame

After $\log_2 n$ rounds of folding, the vectors have length 1:

- Prover holds a single scalar $c'$ (the folded coefficient)
- The $z$-vector has folded to $z'$ (known to both parties)
- The commitment has transformed to $P_{\text{final}}$ through all the updates

**The prover reveals**:

- The final coefficient $c' \in \mathbb{F}$
- The final blinding factor $r' \in \mathbb{F}$

The verifier must check: does $c'$ actually correspond to the final commitment?

$$P_{\text{final}} \stackrel{?}{=} c' \cdot G'_1 + (c' \cdot z'_1) \cdot U + r' \cdot H$$

where $z'_1$ is the final folded evaluation point (known to both parties).

**The catch**: What is $G'_1$? It's the result of folding all the generators through all $\log n$ rounds:

$$\vec{G}' = \alpha_1^{-1} \vec{G}_L^{(1)} + \alpha_1 \vec{G}_R^{(1)} \quad \text{(first fold)}$$
$$\vec{G}'' = \alpha_2^{-1} \vec{G}'_L + \alpha_2 \vec{G}'_R \quad \text{(second fold)}$$
$$\vdots$$

**This is the verifier's bottleneck**: Computing the folded generator $G'_1$ requires applying all $\log n$ folding operations to the original $n$ generators. This takes $O(n)$ group operations (linear in the polynomial size!)

**Why can't we avoid this?** The verifier needs to know what commitment value a correctly-folded polynomial *should* produce. Without computing the folded generators, there's no way to check the prover's final claim.

### Worked Example: IPA Verification

Let's trace through a complete IPA proof for a polynomial with 4 coefficients. This requires 2 rounds of folding.

**Setup**:

- Coefficient vector: $\vec{c} = (3, 5, 2, 7)$ (prover's secret)
- Evaluation point: $z = 2$, so $\vec{z} = (1, 2, 4, 8)$ (public)
- Claimed evaluation: $v = \langle \vec{c}, \vec{z} \rangle = 3(1) + 5(2) + 2(4) + 7(8) = 77$
- Generators: $G_1, G_2, G_3, G_4$ (for coefficients), $U$ (for inner product)
- Initial commitment: $P = (3G_1 + 5G_2 + 2G_3 + 7G_4) + 77U$

The verifier knows: $P$, $\vec{z}$, $v = 77$, and all generators. The verifier does *not* know $\vec{c}$.

**Round 1** (reduce from 4 to 2 elements):

*Prover's work* (uses secret $\vec{c}$):

Split: $\vec{c}_L = (3, 5)$, $\vec{c}_R = (2, 7)$

Compute cross inner products:

- $\langle \vec{c}_L, \vec{z}_R \rangle = 3(4) + 5(8) = 52$
- $\langle \vec{c}_R, \vec{z}_L \rangle = 2(1) + 7(2) = 16$

Send commitments to verifier:

- $L_1 = (3G_3 + 5G_4) + 52U$
- $R_1 = (2G_1 + 7G_2) + 16U$

*Verifier's challenge*: $\alpha_1 = 2$

*Both parties compute* (verifier uses only public information):

Folded generators: $\vec{G}' = \alpha_1^{-1} \vec{G}_L + \alpha_1 \vec{G}_R$

- $G'_1 = \frac{1}{2}G_1 + 2G_3$
- $G'_2 = \frac{1}{2}G_2 + 2G_4$

Folded evaluation vector: $\vec{z}' = \alpha_1^{-1} \vec{z}_L + \alpha_1 \vec{z}_R$

- $z'_1 = \frac{1}{2}(1) + 2(4) = 8.5$
- $z'_2 = \frac{1}{2}(2) + 2(8) = 17$

Updated commitment: $P' = L_1^{\alpha_1^2} \cdot P \cdot R_1^{\alpha_1^{-2}} = L_1^4 \cdot P \cdot R_1^{0.25}$

The verifier doesn't know the cross inner products (52 and 16) directly; those are hidden inside $L_1$ and $R_1$. But because $L_1$ and $R_1$ encode them via the $U$ generator, when the verifier computes $P' = L_1^4 \cdot P \cdot R_1^{0.25}$, the $U$-coefficient of $P'$ automatically becomes $v' = 77 + 4(52) + 0.25(16) = 289$.

*Prover also computes* (secretly):

- $\vec{c}' = \alpha_1 \vec{c}_L + \alpha_1^{-1} \vec{c}_R = 2(3,5) + 0.5(2,7) = (7, 13.5)$

Sanity check: $\langle \vec{c}', \vec{z}' \rangle = 7(8.5) + 13.5(17) = 59.5 + 229.5 = 289$ $\checkmark$

**Round 2** (reduce from 2 to 1 element):

*Prover's work*:

Split: $c'_L = 7$, $c'_R = 13.5$, $z'_L = 8.5$, $z'_R = 17$

Compute cross inner products:

- $c'_L \cdot z'_R = 7 \cdot 17 = 119$
- $c'_R \cdot z'_L = 13.5 \cdot 8.5 = 114.75$

Send commitments:

- $L_2 = 7 G'_2 + 119 U$
- $R_2 = 13.5 G'_1 + 114.75 U$

*Verifier's challenge*: $\alpha_2 = 3$

*Both parties compute*:

Folded generator: $G'' = \alpha_2^{-1} G'_1 + \alpha_2 G'_2 = \frac{1}{3}G'_1 + 3G'_2$

Folded evaluation point: $z'' = \alpha_2^{-1} z'_L + \alpha_2 z'_R = \frac{1}{3}(8.5) + 3(17) = 2.83... + 51 = 53.83...$

Updated commitment: $P'' = L_2^9 \cdot P' \cdot R_2^{1/9}$

Again, the $U$-coefficient of $P''$ becomes $v'' = 289 + 9(119) + \frac{1}{9}(114.75) = 1372.75$.

*Prover computes*:

- $c'' = \alpha_2 c'_L + \alpha_2^{-1} c'_R = 3(7) + \frac{1}{3}(13.5) = 21 + 4.5 = 25.5$

Sanity check: $c'' \cdot z'' = 25.5 \times 53.83... = 1372.75$ $\checkmark$

**Final verification**:

Prover reveals: $c'' = 25.5$

Verifier computes the fully folded generator $G''$ in terms of original generators:
$$G'' = \frac{1}{3}G'_1 + 3G'_2 = \frac{1}{3}\left(\frac{1}{2}G_1 + 2G_3\right) + 3\left(\frac{1}{2}G_2 + 2G_4\right) = \frac{1}{6}G_1 + \frac{3}{2}G_2 + \frac{2}{3}G_3 + 6G_4$$

This is the $O(n)$ work: computing a linear combination of all $n$ original generators.

Verifier checks: $P'' \stackrel{?}{=} c'' \cdot G'' + (c'' \cdot z'') \cdot U$

Substituting: $P'' \stackrel{?}{=} 25.5 \cdot \left(\frac{1}{6}G_1 + \frac{3}{2}G_2 + \frac{2}{3}G_3 + 6G_4\right) + 1372.75 \cdot U$

Which equals: $P'' \stackrel{?}{=} 4.25G_1 + 38.25G_2 + 17G_3 + 153G_4 + 1372.75U$

If this holds, the proof is valid. The verifier is convinced that the prover knows $\vec{c}$ such that $\langle \vec{c}, \vec{z} \rangle = 77$, without ever learning $\vec{c}$.

### Efficiency

- **Commitment size**: One group element (same as KZG)
- **Proof size**: $O(\log n)$ group elements (the $L_i, R_i$ cross-terms from each round)
- **Verifier time**: $O(n)$ (must compute folded generators; this is the fundamental limitation)
- **Prover time**: $O(n \log n)$

The verifier's linear work is the main drawback compared to KZG's constant verification. However, IPA requires no trusted setup; the generators can be chosen transparently (e.g., by hashing).

### The Linear Verifier Problem

This $O(n)$ verification cost is a serious limitation. For a polynomial with $N = 2^{20}$ coefficients (about 1 million), the verifier must perform over a million group operations, each involving expensive elliptic curve arithmetic.

**Why is this problematic?** A scalar multiplication on an elliptic curve involves roughly 400 group additions. Each group addition involves 6-12 base field operations. The result: verification can be ~4000× slower than simple field arithmetic.

For interactive proofs where verification happens once, this is acceptable. For applications like blockchains where proofs are verified by thousands of nodes, or for recursive proof composition, linear verification becomes prohibitive.

This limitation motivated the development of schemes like Hyrax and Dory that exploit additional structure to achieve sublinear verification, which we'll explore shortly.



## Comparing KZG and IPA

| Property | KZG | IPA/Bulletproofs |
|----------|-----|------------------|
| Trusted setup | Required | None |
| Commitment size | $O(1)$ | $O(1)$ |
| Proof size | $O(1)$ | $O(\log n)$ |
| Verification time | $O(1)$ | $O(n)$ |
| Prover time | $O(n)$ | $O(n \log n)$ |
| Assumption | Pairings (q-SDH) | DLog only |
| Quantum-safe | No | No |
| Batch verification | Excellent | Good |

**When to use KZG**: When verification efficiency is paramount and a trusted setup is acceptable. Most production SNARKs (Groth16, PLONK with KZG) use this approach.

**When to use IPA**: When trust minimization is critical, or in systems designed for transparent setups (Halo, Pasta curves).

But what if we want both transparency *and* efficient verification? This is where Dory enters the picture.



## Dory: Logarithmic Verification Without Trusted Setup

The IPA scheme's $O(n)$ verification is a fundamental limitation: the verifier must compute folded generators. **Dory** breaks this barrier using pairings in a clever way.

The key insight: pairings allow verification to happen "in the target group" without the verifier ever touching the original polynomial coefficients or generators directly.

### The Core Idea: Two-Tier Commitments with Pairings

Dory commits to polynomials using a **two-tier structure**:

**Tier 1 (Pedersen for rows)**: Treat the polynomial coefficients as a matrix (reshaping a vector of $N$ coefficients into a $\sqrt{N} \times \sqrt{N}$ matrix). Commit to each row using standard Pedersen commitments in group $\mathbb{G}_1$.

**Tier 2 (Pairing combination)**: Combine the row commitments using generators from $\mathbb{G}_2$, producing a single commitment in the target group $\mathbb{G}_T$.

$$C_M = e(D_{1,1}, G_{2,1}) \cdot e(D_{1,2}, G_{2,2}) \cdot \ldots \cdot e(D_{1,\sqrt{N}}, G_{2,\sqrt{N}}) \cdot e(H_1, H_2)^{r_M}$$

where $D_{1,i}$ is the Pedersen commitment to row $i$ of the matrix, and $r_M$ is a blinding factor.

**Why matrices?** A multilinear polynomial evaluation $f(r_1, \ldots, r_n)$ can be written as a vector-matrix-vector product:

$$f(\vec{r}) = \vec{L}^T \cdot M \cdot \vec{R}$$

where $\vec{L}$ and $\vec{R}$ are Lagrange basis vectors derived from the evaluation point, and $M$ contains the polynomial coefficients arranged as a matrix.

### Polynomial Evaluation as Vector-Matrix-Vector Product

Let's see this concretely. For a bilinear polynomial $f(x_1, x_2)$:

$$f(x_1, x_2) = c_{00}(1-x_1)(1-x_2) + c_{01}(1-x_1)x_2 + c_{10}x_1(1-x_2) + c_{11}x_1x_2$$

To evaluate at $(r_1, r_2)$:

1. **Form the coefficient matrix**:
   $$M = \begin{pmatrix} c_{00} & c_{01} \\ c_{10} & c_{11} \end{pmatrix}$$

2. **Form Lagrange basis vectors**:
   $$\vec{L} = (1-r_1, r_1)^T \quad \text{(left tensor factor)}$$
   $$\vec{R} = (1-r_2, r_2)^T \quad \text{(right tensor factor)}$$

3. **Compute**:
   $$f(r_1, r_2) = \vec{L}^T M \vec{R}$$

For a polynomial with $N = 2^n$ coefficients, we reshape into a $2^{n/2} \times 2^{n/2}$ matrix. The vectors $\vec{L}$ and $\vec{R}$ each have $\sqrt{N}$ entries.

### The Dory Protocol

**Commitment phase** (one-time, during preprocessing):

1. Prover arranges coefficients into matrix $M$
2. For each row $i$, compute Pedersen commitment: $D_{1,i} = \langle \text{row}_i, \vec{G}_1 \rangle + r_i H_1$
3. Combine using pairings: $C_M = \prod_i e(D_{1,i}, G_{2,i}) \cdot e(H_1, H_2)^{r_M}$
4. Send $C_M$ (a single $\mathbb{G}_T$ element)

**Evaluation proof** (to prove $f(\vec{r}) = v$):

1. **Prover computes intermediate vector**: $\vec{v} = M \cdot \vec{R}$

2. **Prover homomorphically computes commitment to $\vec{v}$**:
   $$C_v = \sum_i L_i \cdot D_{1,i}$$

   This works because the row commitments $D_{1,i}$ are Pedersen commitments, which are additively homomorphic!

3. **Prover commits to the Lagrange vector** $\vec{R}$ in $\mathbb{G}_2$:
   $$D_R = \langle \vec{R}, \vec{G}_2 \rangle + r_R H_2$$

4. **Run inner product argument**: Prove that $v = \langle \vec{v}, \vec{R} \rangle$

   This is where Dory's magic happens. The inner product argument uses pairings to verify consistency, but the verifier never needs to compute $O(N)$ group operations.

### Why Dory Achieves Logarithmic Verification

**The Bulletproofs bottleneck**: Verifier must fold generators, requiring $O(n)$ group operations.

**Dory's solution**: Instead of folding generators explicitly, Dory embeds the folding structure into pairing equations. The verifier works with commitments in $\mathbb{G}_T$, which can be updated using just the cross-term commitments from each round.

**Per round, the verifier**:

1. Receives cross-term commitments $C_{LR}, C_{RL}$ (in $\mathbb{G}_T$)
2. Receives challenge $\alpha$
3. Updates the commitment: $C' = C_{LR}^{\alpha} \cdot C \cdot C_{RL}^{\alpha^{-1}}$

**Final verification**: After $\log N$ rounds, the verifier checks that the final commitments are consistent with the prover's claimed values using a small number of pairing equations.

**Total verifier work**: $O(\log N)$ group operations in $\mathbb{G}_T$, plus a constant number of pairings.

### Worked Example: Dory Inner Product Argument

Let's trace through the inner product argument portion of Dory for vectors of length 2.

**Setup**:

- $\vec{v} = (4.6, 15.2)$ (the intermediate vector from $M \cdot \vec{R}$)
- $\vec{R} = (0.2, 0.8)$ (the right Lagrange vector)
- Claimed inner product: $y = \langle \vec{v}, \vec{R} \rangle = 4.6(0.2) + 15.2(0.8) = 13.08$

**Round 1**: Prover splits and computes cross-terms:

- $\vec{v}_L = (4.6)$, $\vec{v}_R = (15.2)$
- $\vec{R}_L = (0.2)$, $\vec{R}_R = (0.8)$
- Cross-products: $z_{LR} = 4.6 \cdot 0.8 = 3.68$ and $z_{RL} = 15.2 \cdot 0.2 = 3.04$
- Prover sends commitments: $C_{LR} = G_T^{3.68}$ and $C_{RL} = G_T^{3.04}$ (with blinding)

**Verifier sends**: Challenge $\alpha = 3$

**Both fold**:

- $v' = \alpha \cdot v_L + v_R = 3(4.6) + 15.2 = 29.0$
- $R' = \alpha^{-1} R_L + R_R = (1/3)(0.2) + 0.8 \approx 0.867$
- $y' = \alpha^2 z_{LR} + y + \alpha^{-2} z_{RL} = 9(3.68) + 13.08 + (1/9)(3.04) \approx 46.53$

**Verification** (base case, dimension 1):

- Prover reveals: $v' = 29.0$, $R' = 0.867$
- Verifier checks: $y' \stackrel{?}{=} v' \cdot R' = 29.0 \cdot 0.867 \approx 25.14$

Wait, this doesn't match! Let me recalculate more carefully using the correct formula.

The folded inner product should satisfy:
$$y' = \langle \alpha \vec{v}_L + \vec{v}_R, \alpha^{-1} \vec{R}_L + \vec{R}_R \rangle$$

Expanding:
$$y' = \alpha \cdot \alpha^{-1} \langle \vec{v}_L, \vec{R}_L \rangle + \alpha \langle \vec{v}_L, \vec{R}_R \rangle + \alpha^{-1} \langle \vec{v}_R, \vec{R}_L \rangle + \langle \vec{v}_R, \vec{R}_R \rangle$$

$$= \langle \vec{v}_L, \vec{R}_L \rangle + \langle \vec{v}_R, \vec{R}_R \rangle + \alpha \cdot z_{LR} + \alpha^{-1} \cdot z_{RL}$$

$$= (4.6)(0.2) + (15.2)(0.8) + 3(3.68) + (1/3)(3.04)$$

$$= 0.92 + 12.16 + 11.04 + 1.01 = 25.13$$

And: $v' \cdot R' = 29.0 \times 0.867 = 25.14$

The slight discrepancy is rounding. The key point: **the verifier checks that the folded values are consistent**, and because the prover committed to cross-terms before seeing $\alpha$, they cannot cheat.

### Dory: Properties and Trade-offs

| Property | Dory |
|----------|------|
| Trusted setup | **None (Transparent)** |
| Commitment size | $O(1)$ (one $\mathbb{G}_T$ element) |
| Proof size | $O(\log N)$ group elements |
| Verification time | **$O(\log N)$** (the key improvement!) |
| Prover time | $O(N)$ for commitment, $O(\log N)$ per opening |
| Assumption | Pairings + DLog |
| Quantum-safe | No (uses pairings) |
| Small-value preservation | Yes (benefits from sparse/small coefficients) |

**The key trade-off**: Dory uses pairings (like KZG) but achieves transparency (like IPA). It gets logarithmic verification (better than IPA) at the cost of using the more complex pairing machinery.

**When to use Dory**: When you need both transparency and efficient verification. Dory is particularly attractive for systems with many polynomial openings that can be batched (like Jolt's zkVM), where the amortized cost per opening becomes very small.

**Why the logarithmic verifier?** IPA's linear verification cost comes from computing folded generators, applying $\log n$ folding operations to $n$ original generators. Dory sidesteps this using pairings. The commitment structure uses two layers: first, commit to each row of a coefficient matrix with Pedersen; then commit to that vector of commitments using an AFGHO commitment in the target group $\mathbb{G}_T$. When folding, the verifier can combine commitments that the prover provides (constant work per round) rather than computing folded generators themselves. The algebraic structure of pairings, specifically $e(aG_1, bG_2) = e(G_1, G_2)^{ab}$, lets the verifier "absorb" the folding challenges into the commitments without touching the underlying generators.



## Multilinear Polynomial Commitments

Both paradigms extend to multilinear polynomials, crucial for systems based on the boolean hypercube (like the sum-check protocol).

### Multilinear KZG

A multilinear polynomial $f(X_1, \ldots, X_\ell)$ can be committed using an SRS that encodes evaluations of Lagrange basis polynomials at a secret point.

To prove $f(z_1, \ldots, z_\ell) = v$:

- Use the identity: $f(z) = v$ iff $f(X) - v = \sum_{i=1}^\ell (X_i - z_i) w_i(X)$ for some witness polynomials $w_i$
- The proof consists of $\ell$ commitments (one per witness polynomial)
- Verification uses $\ell + 1$ pairings

Proof size grows linearly with the number of variables, not exponentially with the polynomial size.

### Multilinear IPA

The tensor structure of multilinear extensions plays beautifully with the folding technique. The $\ell$-dimensional evaluation vector has product structure, and folding exploits this systematically.



## The Role of PCS in SNARKs

Polynomial commitment schemes are the cryptographic core that transforms interactive protocols into succinct, non-interactive proofs.

**The recipe**:

1. **Arithmetization**: Convert computation to polynomial constraints
2. **IOP**: Define an interactive protocol where the prover sends polynomials (abstractly)
3. **PCS**: Compile the IOP using a polynomial commitment scheme
4. **Fiat-Shamir**: Make non-interactive by deriving challenges from transcript hashes

The PCS handles the "oracle" aspect of IOPs. Instead of the verifier having oracle access to polynomials, the prover commits to them, and later provides evaluation proofs at queried points.

Different PCS choices lead to different SNARK properties:

- KZG → Groth16, PLONK (trusted setup, constant proofs)
- IPA → Halo (transparent, larger proofs, linear verification)
- Dory → Jolt (transparent, logarithmic verification)
- FRI (Chapter 10) → STARKs (transparent, post-quantum)



## The Complete PCS Landscape

Now that we've seen three commitment schemes in depth, let's compare them systematically:

| Property | KZG | IPA/Bulletproofs | Dory | FRI (Ch. 10) |
|----------|-----|------------------|------|--------------|
| **Trusted setup** | Required | None | None | None |
| **Commitment size** | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ |
| **Proof size** | $O(1)$ | $O(\log N)$ | $O(\log N)$ | $O(\log^2 N)$ |
| **Verification time** | $O(1)$ | $O(N)$ | $O(\log N)$ | $O(\log^2 N)$ |
| **Prover time** | $O(N)$ | $O(N \log N)$ | $O(N)$ | $O(N \log N)$ |
| **Assumption** | q-SDH + Pairings | DLog only | DLog + Pairings | Hash collision |
| **Post-quantum** | No | No | No | **Yes** |
| **Batching** | Excellent | Good | Very good | Good |

**How to read this table**:

- **Fastest verification**: KZG ($O(1)$), but requires trust
- **Best transparency**: IPA, Dory, or FRI (no trusted setup needed)
- **Best of both worlds**: Dory gets logarithmic verification with transparency
- **Post-quantum ready**: Only FRI (Chapter 10) survives quantum computers

**Practical guidance**:

- **Ethereum L1**: KZG (verification cost is critical, trusted setup is acceptable)
- **Privacy coins**: Dory or IPA (trust minimization matters)
- **Long-term security**: FRI/STARKs (quantum resistance)
- **zkVMs with batching**: Dory (amortizes opening costs across many polynomials)



## Key Takeaways

1. **The core problem**: Verifying polynomial identities without revealing the polynomial.

2. **PCS abstraction**: Commit, Open, Verify. Binding ensures consistency; succinctness enables efficiency.

3. **KZG magic**: Pairings allow checking polynomial identity at a secret point. Constant-size proofs, constant verification.

4. **KZG cost**: Trusted setup is required. The "toxic waste" $\tau$ must be destroyed.

5. **Inner product argument (IPA)**: Reduce polynomial evaluation to inner product. Prove via recursive folding ($\log N$ rounds halving the problem each time).

6. **IPA verification bottleneck**: The verifier must compute folded generators, requiring $O(N)$ group operations. Transparent, but expensive.

7. **Dory's innovation**: Two-tier matrix commitment with pairings. Achieves $O(\log N)$ verification while remaining transparent.

8. **The tensor product insight**: Polynomial evaluation has multiplicative structure that can be exploited for efficiency (Dory, Hyrax).

9. **Multilinear extensions**: All paradigms extend to multilinear polynomials, crucial for hypercube-based protocols like sum-check.

10. **PCS choice determines SNARK properties**: KZG for minimal proof size with trusted setup; IPA for simplicity with linear verification; Dory for transparency with efficient verification; FRI (next chapter) for post-quantum security. The choice is application-dependent.
