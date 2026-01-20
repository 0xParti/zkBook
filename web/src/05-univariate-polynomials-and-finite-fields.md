# Chapter 5: Univariate Polynomials and Finite Fields

There is something deeply satisfying about roots of unity.

Take a special element $\omega$ in a field: one where $\omega^n = 1$ but $\omega^k \neq 1$ for any smaller positive $k$. Such an element is called a **primitive $n$-th root of unity**. Raise it to successive powers: $\omega, \omega^2, \omega^3, \ldots$ and you trace out exactly $n$ distinct values before cycling back to 1. Not every element has this property (most don't). But when one does, the powers form a perfect cyclic structure. In the complex plane, these values are evenly spaced around the unit circle, a perfect $n$-gon inscribed in a circle of radius one.

This cyclical structure has consequences. When you square all the $n$-th roots of unity, you get the $(n/2)$-th roots, each appearing twice. When you add a root to its "opposite" (half a cycle away), you get zero: $\omega^k + \omega^{k+n/2} = 0$. These aren't coincidences to be memorized; they're manifestations of the underlying symmetry, which we'll explore in detail below.

And this symmetry is *useful*. The Fast Fourier Transform, one of the most important algorithms in all of computing, exists because of roots of unity. Polynomial multiplication, signal processing, and now zero-knowledge proofs all exploit the same beautiful structure.

This chapter develops the univariate polynomial paradigm: finite fields, roots of unity, and the techniques that make systems like Groth16, PLONK, and STARKs possible. Where Chapter 4 explored multilinear polynomials over the Boolean hypercube, here we explore a single variable of high degree over a very different domain.



## Finite Fields: The Algebraic Foundation

Zero-knowledge proofs live in finite fields. Not the real numbers, not the integers; finite fields, where arithmetic wraps around and every division is exact.

A finite field $\mathbb{F}_p$ consists of the integers $\{0, 1, 2, \ldots, p-1\}$ with arithmetic modulo a prime $p$. Addition and multiplication work as usual, then you take the remainder when dividing by $p$:

$$3 + 5 = 8 \equiv 1 \pmod 7$$
$$3 \times 5 = 15 \equiv 1 \pmod 7$$

The magic is in division. Every nonzero element has a multiplicative inverse: this is guaranteed because $p$ is prime. (More generally, finite fields exist for any prime power $p^k$, but prime fields $\mathbb{F}_p$ are the simplest case.) In $\mathbb{F}_7$, we have $3^{-1} = 5$ because $3 \times 5 = 15 \equiv 1$. You can divide by any nonzero element, and the result is exact (no fractions, no approximations).

The nonzero elements $\mathbb{F}_p^* = \{1, 2, \ldots, p-1\}$ form a **cyclic group** under multiplication. This is fundamental: there exists a **generator** $g$ such that every nonzero element is some power of $g$.

**Example in $\mathbb{F}_7$**: The element $3$ generates everything:

| Power | $3^k \mod 7$ |
|-------|--------------|
| $3^1$ | $3$ |
| $3^2$ | $2$ |
| $3^3$ | $6$ |
| $3^4$ | $4$ |
| $3^5$ | $5$ |
| $3^6$ | $1$ |

Every nonzero element appears exactly once. The powers cycle through all of $\mathbb{F}_7^*$ before returning to 1.

For cryptographic applications, we use primes of 256 bits or more. The field is vast, roughly $2^{256}$ elements, making exhaustive search impossible.



## Roots of Unity

Because $\mathbb{F}_p^*$ is cyclic of order $p-1$, it contains subgroups of every order dividing $p-1$. The most useful are the **roots of unity**.

An element $\omega \in \mathbb{F}_p$ is an **$n$-th root of unity** if $\omega^n = 1$. It's a **primitive** $n$-th root if additionally $\omega^k \neq 1$ for any $0 < k < n$: the smallest positive power that gives 1 is exactly $n$.

If $\omega$ is a primitive $n$-th root, the complete set of $n$-th roots is:

$$H = \{1, \omega, \omega^2, \ldots, \omega^{n-1}\}$$

This is a subgroup of order $n$. It's the evaluation domain that powers univariate-based SNARKs.

### Worked Example: Fourth Roots in $\mathbb{F}_{17}$

Take $p = 17$. The multiplicative group has order $16 = 2^4$. Since $4$ divides $16$, fourth roots of unity exist.

Is $\omega = 4$ a primitive fourth root?

$$4^1 = 4$$
$$4^2 = 16 \equiv -1 \pmod{17}$$
$$4^3 = 64 \equiv 13 \equiv -4 \pmod{17}$$
$$4^4 = 256 \equiv 1 \pmod{17}$$

Yes. The fourth roots of unity are:

$$H = \{1, 4, 16, 13\} = \{1, 4, -1, -4\}$$

Notice the structure: $4$ and $-4 = 13$ are negatives of each other, as are $1$ and $-1 = 16$. This is not a coincidence.



## The Symmetries

Roots of unity have two key symmetries that enable fast algorithms.

### Symmetry 1: Squaring Halves the Group

When $n$ is even:

$$\omega^{n/2} = -1$$

**Why is this true?** Start with the defining property: $\omega^n = 1$. Taking the square root of both sides: $(\omega^{n/2})^2 = 1$. So $\omega^{n/2}$ is a square root of 1. In any field, the square roots of 1 are exactly $1$ and $-1$. But $\omega^{n/2} \neq 1$ because $\omega$ is *primitive*: its first power to equal 1 is $\omega^n$, not $\omega^{n/2}$. Therefore $\omega^{n/2} = -1$.

This has a remarkable consequence. If you square every element of $H$:

$$(\omega^k)^2 = \omega^{2k}$$

The squares form the $(n/2)$-th roots of unity. And since $(\omega^{k + n/2})^2 = (\omega^k \cdot \omega^{n/2})^2 = (\omega^k)^2 \cdot 1 = (\omega^k)^2$, each square root of unity appears exactly twice.

**In $\mathbb{F}_{17}$**: Squaring the fourth roots $\{1, 4, 16, 13\}$:

$$1^2 = 1, \quad 4^2 = 16, \quad 16^2 = 1, \quad 13^2 = 16$$

The squares are $\{1, 16\}$: the square roots of unity, each appearing twice.

### Symmetry 2: Opposite Elements are Negatives

Elements half a cycle apart are negatives:

$$\omega^{k + n/2} = \omega^k \cdot \omega^{n/2} = -\omega^k$$

**In $\mathbb{F}_{17}$**:

- $\omega^0 = 1$ and $\omega^2 = 16 = -1$
- $\omega^1 = 4$ and $\omega^3 = 13 = -4$

These two symmetries, squaring halves the group and opposites are negatives, are the engine of the Fast Fourier Transform.

### The DFT Is Polynomial Evaluation

Here is one of those facts that seems almost too good to be true.

The Discrete Fourier Transform (DFT) is defined as a matrix-vector multiplication. Given a vector $(c_0, c_1, \ldots, c_{n-1})$, the DFT produces a new vector whose $k$-th entry is:

$$\sum_{j=0}^{n-1} c_j \cdot \omega^{jk}$$

where $\omega$ is a primitive $n$-th root of unity.

Now look at polynomial evaluation. Given a polynomial $P(X) = c_0 + c_1 X + \cdots + c_{n-1} X^{n-1}$, evaluate it at $\omega^k$:

$$P(\omega^k) = \sum_{j=0}^{n-1} c_j \cdot (\omega^k)^j = \sum_{j=0}^{n-1} c_j \cdot \omega^{jk}$$

They are identical. The DFT of the coefficient vector *is* the evaluation vector at roots of unity. This is not a useful analogy or a computational trick. It is a mathematical identity.

The FFT, then, is not "like" converting between polynomial representations. It *is* converting between polynomial representations. Coefficient form and evaluation form are the two natural bases for the same vector space, and the DFT matrix is the change-of-basis matrix. The FFT is the fast algorithm for this change of basis, made possible by the recursive structure of roots of unity.

This is why the same algorithm appears in signal processing, image compression, and zero-knowledge proofs. They are not merely related applications; they are the same mathematical operation in different disguises.

### Resonance: A Physical Intuition

There's a reason the Fourier transform appears in both signal processing and cryptographic proofs: both are exploiting the same mathematical structure.

In physics, every oscillating system has **natural frequencies**: the resonant modes where energy flows most efficiently. Strike a bell, and it rings at specific pitches. Pluck a string, and it vibrates in harmonics. These aren't arbitrary; they're the **eigenfrequencies** of the system, determined by its physical structure.

Roots of unity are the eigenfrequencies of the multiplicative group $\mathbb{F}_p^*$.

Just as a physical system has modes that "fit" its boundary conditions, the finite field has elements that "fit" its cyclic structure. The $n$-th roots of unity are exactly the elements whose powers repeat with period $n$; they resonate with the group's multiplicative structure.

The FFT is decomposition into eigenmodes. A polynomial is a sum of monomials, and each monomial interacts differently with roots of unity. The FFT separates these interactions, projecting the polynomial onto each eigenmode. Evaluating a polynomial at all $n$-th roots simultaneously is like decomposing a sound into its frequency components: same mathematics, different interpretation.

This is why operations that are hard in one basis become easy in another. Multiplication of polynomials (convolution in coefficient space) becomes pointwise multiplication in evaluation space. The FFT is a change of basis to the eigenbasis, where operations decouple.



## Two Representations of Polynomials

A polynomial of degree less than $n$ can be viewed in two ways.

**Coefficient form**: The polynomial is stored as its coefficients.

$$P(X) = c_0 + c_1 X + c_2 X^2 + \cdots + c_{n-1} X^{n-1}$$

**Evaluation form**: The polynomial is stored as its values at $n$ distinct points. Using the $n$-th roots of unity:

$$[P(1), P(\omega), P(\omega^2), \ldots, P(\omega^{n-1})]$$

These two forms carry exactly the same information. A polynomial of degree less than $n$ is uniquely determined by its values at any $n$ points (this is Lagrange interpolation). The coefficient form and evaluation form are just two different coordinate systems for the same object.

Why care about evaluation form? In zero-knowledge proofs, constraints are naturally expressed as evaluations. Gate $i$ must satisfy some relation; this becomes: the constraint polynomial $C(X)$ must equal zero at $\omega^i$. The evaluation form directly represents these constraints.

### Polynomial Evaluation as Inner Product

Here's a key observation that bridges polynomials and linear algebra: **evaluating a polynomial is computing an inner product**.

In coefficient form:
$$P(z) = c_0 + c_1 z + c_2 z^2 + \cdots + c_{n-1} z^{n-1} = \langle \vec{c}, \vec{z} \rangle$$

where $\vec{c} = (c_0, c_1, \ldots, c_{n-1})$ is the coefficient vector and $\vec{z} = (1, z, z^2, \ldots, z^{n-1})$ is the "powers of $z$" vector.

In evaluation form, the same polynomial can be written via Lagrange interpolation:
$$P(z) = \sum_{i=0}^{n-1} P(\omega^i) \cdot L_i(z) = \langle \vec{P}, \vec{L}(z) \rangle$$

where $\vec{P} = (P(1), P(\omega), \ldots, P(\omega^{n-1}))$ is the evaluation vector and $\vec{L}(z) = (L_0(z), L_1(z), \ldots, L_{n-1}(z))$ is the vector of Lagrange basis evaluations.

Either way, polynomial evaluation is an inner product. This observation is surprisingly powerful: it means that **committing to a polynomial** (in either form) reduces to **committing to a vector**, and **proving an evaluation** reduces to **proving an inner product claim**. We'll exploit this connection extensively in Chapter 9.

**Two ways to commit**: This duality (coefficient form vs evaluation form) manifests directly in polynomial commitment schemes:

- **KZG** (Chapter 9) commits in coefficient form: $C = g^{f(\tau)} = \prod_i (g^{\tau^i})^{c_i}$. The commitment encodes "evaluate the coefficients at a secret point $\tau$."

- **FRI** (Chapter 10) commits in evaluation form: a Merkle tree over $[f(1), f(\omega), \ldots, f(\omega^{n-1})]$. The commitment is a hash of all the evaluations.

The FFT is what makes these equivalent: you can convert between representations in $O(n \log n)$ time. But the choice of representation affects everything: proof size, prover cost, setup requirements, and the algebraic tricks available for verification.



## The Fast Fourier Transform

Converting between coefficient and evaluation form naively takes $O(n^2)$ operations: you'd compute each of $n$ evaluations, each requiring $O(n)$ work.

The **Fast Fourier Transform (FFT)** does it in $O(n \log n)$. This speedup is essential; without it, the polynomials in modern proof systems would be computationally intractable.

The FFT exploits the symmetries of roots of unity through divide-and-conquer.

### The Core Idea

Split a polynomial into its even and odd terms:

$$P(X) = P_{\text{even}}(X^2) + X \cdot P_{\text{odd}}(X^2)$$

where:

- $P_{\text{even}}(Y) = c_0 + c_2 Y + c_4 Y^2 + \cdots$ (even-indexed coefficients)
- $P_{\text{odd}}(Y) = c_1 + c_3 Y + c_5 Y^2 + \cdots$ (odd-indexed coefficients)

Both have half the degree of $P$.

Now, when we square the $n$-th roots of unity, we get the $(n/2)$-th roots (each appearing twice). So to evaluate $P$ at all of $H$, we:

1. Recursively evaluate $P_{\text{even}}$ and $P_{\text{odd}}$ at the $(n/2)$-th roots
2. Combine the results

The combination uses the antisymmetry property:

$$P(\omega^k) = P_{\text{even}}(\omega^{2k}) + \omega^k \cdot P_{\text{odd}}(\omega^{2k})$$
$$P(\omega^{k + n/2}) = P_{\text{even}}(\omega^{2k}) - \omega^k \cdot P_{\text{odd}}(\omega^{2k})$$

Two evaluations of $P$ from one evaluation each of $P_{\text{even}}$ and $P_{\text{odd}}$: the same work computes both, with just an addition versus subtraction.

### Worked Example: 4-Point FFT

Evaluate $P(X) = 5 + 3X + X^2 + 2X^3$ at $H = \{1, 4, 16, 13\}$ in $\mathbb{F}_{17}$.

**Split**:

- $P_{\text{even}}(Y) = 5 + Y$ (coefficients $c_0 = 5$, $c_2 = 1$)
- $P_{\text{odd}}(Y) = 3 + 2Y$ (coefficients $c_1 = 3$, $c_3 = 2$)

**Evaluate on $\{1, 16\}$** (the square roots of unity):

| $Y$ | $P_{\text{even}}(Y) = 5 + Y$ | $P_{\text{odd}}(Y) = 3 + 2Y$ |
|-----|------------------------------|------------------------------|
| $1$ | $6$ | $5$ |
| $16$ | $21 \equiv 4$ | $35 \equiv 1$ |

**Combine** using $\omega^0 = 1$, $\omega^1 = 4$, $\omega^2 = 16$, $\omega^3 = 13$:

$$P(1) = P_{\text{even}}(1) + 1 \cdot P_{\text{odd}}(1) = 6 + 5 = 11$$
$$P(4) = P_{\text{even}}(16) + 4 \cdot P_{\text{odd}}(16) = 4 + 4 = 8$$
$$P(16) = P_{\text{even}}(1) - 1 \cdot P_{\text{odd}}(1) = 6 - 5 = 1$$
$$P(13) = P_{\text{even}}(16) - 4 \cdot P_{\text{odd}}(16) = 4 - 4 = 0$$

**Result**: $[P(1), P(4), P(16), P(13)] = [11, 8, 1, 0]$.

Verification: $P(4) = 5 + 3(4) + 16 + 2(64) = 5 + 12 + 16 + 128 = 161 \equiv 8 \pmod{17}$. Correct.

The inverse FFT, going from evaluations back to coefficients, uses the same algorithm with $\omega^{-1}$ instead of $\omega$ and a factor of $1/n$.



## The Vanishing Polynomial

Here is the central insight of univariate arithmetization.

The **vanishing polynomial** of a set $H$ is:

$$Z_H(X) = \prod_{h \in H}(X - h)$$

For the $n$-th roots of unity, this simplifies dramatically:

$$Z_H(X) = X^n - 1$$

This is because $\omega^n = 1$ for every root of unity $\omega$; they are precisely the roots of $X^n - 1$.

**The key theorem**: A polynomial $C(X)$ vanishes at every point of $H$ if and only if $Z_H(X)$ divides $C(X)$.

This is the compression at the heart of univariate SNARKs:

1. Encode $n$ constraints as: "$C(\omega^i) = 0$ for all $i$"
2. This is equivalent to: "$Z_H(X)$ divides $C(X)$"
3. Which is equivalent to: "There exists $Q(X)$ such that $C(X) = Q(X) \cdot Z_H(X)$"

One polynomial divisibility condition captures $n$ separate constraint checks.



## The Divisibility Check

How do we verify divisibility efficiently?

The prover computes the quotient $Q(X) = C(X) / Z_H(X)$ and commits to it. The verifier picks a random challenge $z \in \mathbb{F}$ and checks:

$$C(z) \stackrel{?}{=} Q(z) \cdot Z_H(z)$$

If $C(X) = Q(X) \cdot Z_H(X)$ as polynomials, this equation holds for all $z$, including the random one.

If $C(X) \neq Q(X) \cdot Z_H(X)$, their difference is a nonzero polynomial. By Schwartz-Zippel, a random $z$ catches this disagreement with probability at least $1 - d/|\mathbb{F}|$, where $d$ is the degree.

One random check. $n$ constraints verified. This is the magic.



## Lagrange Interpolation

Given evaluations at roots of unity, how do we recover the polynomial?

The **Lagrange basis polynomial** $L_i(X)$ equals 1 at $\omega^i$ and 0 at all other roots:

$$L_i(X) = \prod_{j \neq i} \frac{X - \omega^j}{\omega^i - \omega^j}$$

The polynomial passing through points $({\omega^i, y_i})$ is:

$$P(X) = \sum_{i=0}^{n-1} y_i \cdot L_i(X)$$

For roots of unity, the Lagrange basis has a beautiful closed form:

$$L_i(X) = \frac{\omega^i}{n} \cdot \frac{X^n - 1}{X - \omega^i}$$

The factor $\frac{X^n - 1}{X - \omega^i}$ vanishes at all roots except $\omega^i$. The prefactor $\frac{\omega^i}{n}$ normalizes to give $L_i(\omega^i) = 1$.



## Cosets: Shifting the Domain

Sometimes we need evaluation points outside $H$. **Cosets** provide them while preserving structure.

If $k \notin H$ is any nonzero field element, then:

$$k \cdot H = \{k, k\omega, k\omega^2, \ldots, k\omega^{n-1}\}$$

is a coset of $H$. It's a "shifted" copy: $n$ new points, disjoint from $H$.

**Why cosets matter in ZK:** Several proof systems crucially depend on cosets:

- **PLONK's permutation argument**: Uses multiple cosets to encode wire positions. If you have $n$ gates with 3 wires each ($a$, $b$, $c$), PLONK encodes them on $H$, $kH$, and $k^2H$ (three disjoint domains of size $n$ each). This lets the permutation polynomial distinguish "wire $a$ of gate 5" from "wire $b$ of gate 5."

- **FRI's low-degree testing**: The prover evaluates on a domain larger than the polynomial's degree (for "rate" or "blowup"). Using $H \cup kH$ doubles the evaluation domain while maintaining FFT structure.

- **Quotient degree management**: If $C(X)$ has degree $2n$ but we've only committed to evaluations on $H$ (size $n$), we need more points to pin down the quotient. Using $H \cup kH$ gives $2n$ points (enough to determine a polynomial of degree less than $2n$).

The FFT works on cosets too: just multiply each root of unity by $k$ before running the algorithm.



## The Quotient Argument

A fundamental operation: prove that $P(z) = y$ for a committed polynomial $P$.

The **factor theorem** says: $P(z) = y$ if and only if $(X - z)$ divides $P(X) - y$.

The prover computes:

$$Q(X) = \frac{P(X) - y}{X - z}$$

If $P(z) = y$, this is a polynomial. If not, the division has a remainder; $Q$ isn't a polynomial.

The verifier checks the polynomial identity:

$$P(X) - y = Q(X) \cdot (X - z)$$

at a random point. This is the foundation of KZG opening proofs (Chapter 9).



## Univariate vs. Multilinear

We now have two paradigms for polynomial proofs:

| Aspect | Multilinear | Univariate |
|--------|-------------|------------|
| **Variables** | $n$ variables, degree 1 each | 1 variable, degree $n-1$ |
| **Domain** | Boolean hypercube $\{0,1\}^n$ | Roots of unity $H$ |
| **Size** | $2^n$ points | $n$ points |
| **Constraint encoding** | Sum over hypercube | Divisibility by $Z_H$ |
| **Key algorithm** | Recursive halving | FFT |
| **Verification** | Sum-check protocol | Random evaluation |
| **Systems** | GKR, Spartan, Lasso | PLONK, Marlin, STARKs |

Both achieve the same essential goal: reduce exponentially many constraint checks to a constant number of random evaluations. They're complementary perspectives on the same phenomenon (the rigidity of low-degree polynomials).

**A note on Groth16**: Groth16 uses univariate polynomials but doesn't require roots of unity; it encodes constraints via QAP (Quadratic Arithmetic Programs) and verifies satisfaction through pairing equations, not divisibility checks at structured domains. Provers *can* use FFT as an optimization for polynomial arithmetic, but it's not fundamental to the protocol. PLONK and STARKs, by contrast, rely structurally on roots of unity: constraints are encoded as "polynomial vanishes on $H$," checked via the divisibility pattern described above.



## Key Takeaways

1. **Finite fields** provide exact arithmetic with every nonzero element invertible. The nonzero elements form a cyclic group.

2. **Roots of unity** are elements with $\omega^n = 1$. They form a subgroup of size $n$ when $n$ divides $p-1$.

3. **The key symmetries**: Squaring halves the group; opposite elements are negatives. These enable the FFT.

4. **Two representations**: Polynomials can be stored as coefficients or evaluations. The FFT converts between them in $O(n \log n)$ time.

5. **The vanishing polynomial** $Z_H(X) = X^n - 1$ captures all roots of unity. A polynomial vanishes on $H$ iff $Z_H$ divides it.

6. **Constraint compression**: $n$ constraints "$C(\omega^i) = 0$" become one divisibility "$Z_H | C$", verified by one random check.

7. **Lagrange interpolation** over roots of unity has a clean closed form exploiting the structure of $Z_H$.

8. **Cosets** extend the domain while preserving FFT-friendliness.

9. **Quotient arguments** prove evaluation claims: to show $P(z) = y$, prove $(X-z)$ divides $P(X) - y$.

10. **The FFT exists because of roots of unity.** The algorithm is a direct consequence of the symmetries $\omega^{n/2} = -1$ and $(\omega^k)^2 = \omega^{2k}$.
