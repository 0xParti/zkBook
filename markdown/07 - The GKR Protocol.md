# Chapter 7: The GKR Protocol: Verifying Circuits Layer by Layer

The sum-check protocol is extraordinary. It transforms exponentially large sums ($2^n$ terms) into verification that runs in $O(n)$ time, logarithmic in the sum size. But every application we've seen (#SAT, triangle counting, matrix multiplication) requires a custom polynomial tailored to that specific problem. Each new computation demands a new arithmetization.

What if we want to verify *any* computation, not just counting problems?

Real programs don't come as polynomial identities. They come as algorithms: loops, conditionals, data structures, function calls. Before sum-check can help, someone must translate the computation into polynomial form. For every new program, we'd need a mathematician to design a custom reduction.

The GKR protocol, named after Goldwasser, Kalai, and Rothblum, solves this problem elegantly. It provides a *universal* framework for verifying any computation that can be expressed as an arithmetic circuit (which turns out to be everything). Rather than designing a new protocol for each problem, GKR gives us a machine: feed in a circuit, get out an efficient verification protocol.



## From Sum-Check to General Computation

Let's understand the conceptual leap. The sum-check protocol verifies claims of the form:

$$H = \sum_{x \in \{0,1\}^n} g(x)$$

Given a polynomial $g$, it checks whether the claimed sum $H$ is correct. The polynomial $g$ encodes the problem, and sum-check verifies the encoding.

But computation is more than summation. A real computation involves:

- Input values
- Intermediate calculations (additions, multiplications)
- Data dependencies (the output of one step becomes the input to another)
- A final output

The insight of GKR is that these computations have *layered structure*. A circuit consists of gates organized into layers, where each layer's outputs feed into the next layer's inputs. And crucially, the relationship between adjacent layers can be expressed as a polynomial identity: one that sum-check can verify.

> **Remark (GKR as a chain of sum-checks).** GKR is a sequence of sum-checks, each reducing a claim about layer $i$ to a claim about layer $i+1$. This is a special case of a more general pattern: sum-checks composing into directed graphs, where each sum-check is a node and evaluation claims are edges. GKR's graph is a path (linear chain from output to input). More complex protocols like Spartan (Chapter 19) have branching structure: one outer sum-check spawns multiple inner sum-checks. The graph perspective, where depth determines sequential stages and width enables batching, becomes crucial for understanding prover efficiency in Chapters 19-20.



## Layered Arithmetic Circuits

GKR operates on **layered arithmetic circuits**: directed acyclic graphs (graphs with edges that have direction, and no cycles; you can never follow edges back to where you started) where:

1. **Layers**: Gates are organized into layers $0, 1, \ldots, d$

   - Layer $d$ is the input layer
   - Layer $0$ is the output layer
   - Wires only connect adjacent layers (from layer $i+1$ to layer $i$)

2. **Gate operations**: Each gate performs either addition or multiplication, with exactly two inputs

3. **Indexing**: Gates within each layer are numbered using binary strings
   - If layer $i$ has $S_i$ gates, we use $k_i = \lceil \log_2 S_i \rceil$ bits to index them
   - Gate $j$ in layer $i$ has label $j \in \{0,1\}^{k_i}$

Any circuit can be transformed into this layered form. If a wire spans multiple layers, we insert "pass-through" gates (identity gates that output their input unchanged).

**Example Circuit**: Let's trace through a simple circuit computing $(x_1 + x_2) \cdot x_3$.

```
Layer 2 (Inputs):    x_1      x_2      x_3
                      \       /          |
                       \     /           |
Layer 1 (Middle):      [+]           [pass]
                         \             /
                          \           /
Layer 0 (Output):           [*]
                             |
                          output
```

**Gate labeling**:

- Layer 2 (inputs): $k_2 = 2$ bits needed for 3 gates
  - $x_1 \to (0,0)$, $x_2 \to (0,1)$, $x_3 \to (1,0)$
- Layer 1: $k_1 = 1$ bit for 2 gates
  - Addition gate $\to (0)$, pass-through $\to (1)$
- Layer 0 (output): $k_0 = 1$ bit for 1 gate
  - Multiplication gate $\to (0)$



## The Wiring Predicates

The circuit's structure is encoded by **wiring predicates**: functions that describe which gates connect to which.

For layer $i$, we define:

$$\text{add}_i(a, b, c) = \begin{cases} 1 & \text{if gate } a \text{ in layer } i \text{ is an addition gate with inputs } b, c \text{ from layer } i+1 \\ 0 & \text{otherwise} \end{cases}$$

$$\text{mult}_i(a, b, c) = \begin{cases} 1 & \text{if gate } a \text{ in layer } i \text{ is a multiplication gate with inputs } b, c \text{ from layer } i+1 \\ 0 & \text{otherwise} \end{cases}$$

For our example circuit, look at layer 0. It contains a single multiplication gate, labeled $(0)$. This gate multiplies the outputs of gate $(0)$ (the addition gate computing $x_1 + x_2$) and gate $(1)$ (the pass-through carrying $x_3$) from layer 1. The wiring predicate encodes exactly this:

$$\text{mult}_0(a, b, c) = \begin{cases} 1 & \text{if } a = 0, b = 0, c = 1 \\ 0 & \text{otherwise} \end{cases}$$

Reading this: "Gate $a=0$ in layer 0 is a multiplication gate whose left input comes from gate $b=0$ in layer 1, and whose right input comes from gate $c=1$ in layer 1." The predicate returns 1 only for this specific triple; all other combinations yield 0.

The layer has no addition gates, so $\text{add}_0$ is identically zero.

**The key observation**: These predicates depend only on the *circuit structure*, not on the input values. The verifier, who knows the circuit, can compute these predicates efficiently.



## Gate Values as Polynomials

Let $W_i : \{0,1\}^{k_i} \to \mathbb{F}$ denote the function mapping each gate label in layer $i$ to its output value. The prover, having evaluated the circuit on specific inputs, knows all of $W_0, W_1, \ldots, W_d$.

We extend these to multilinear polynomials $\tilde{W}_i$ over $\mathbb{F}^{k_i}$. Similarly, we extend the wiring predicates to multilinear polynomials $\widetilde{\text{add}}_i$ and $\widetilde{\text{mult}}_i$.

For our example with inputs $x_1 = 2$, $x_2 = 3$, $x_3 = 4$:

**Layer 2 values** (inputs):
$$W_2(0,0) = 2, \quad W_2(0,1) = 3, \quad W_2(1,0) = 4, \quad W_2(1,1) = 0$$

(The fourth entry is padding: we have 3 inputs but need $2^2 = 4$ slots for 2-bit indexing. Unused slots are set to 0.)

The MLE is:
$$\tilde{W}_2(y_1, y_2) = 2(1-y_1)(1-y_2) + 3(1-y_1)y_2 + 4 \cdot y_1(1-y_2)$$

**Layer 1 values**:
$$W_1(0) = x_1 + x_2 = 5, \quad W_1(1) = x_3 = 4$$

The MLE is:
$$\tilde{W}_1(z) = 5(1-z) + 4z = 5 - z$$

**Layer 0 values** (output):
$$W_0(0) = (x_1 + x_2) \cdot x_3 = 20$$

The MLE is:
$$\tilde{W}_0(u) = 20(1-u)$$



## The Layer Reduction Lemma

The heart of GKR is a beautiful algebraic identity that links adjacent layers:

**GKR Lemma**: For any point $z \in \mathbb{F}^{k_i}$:

$$\tilde{W}_i(z) = \sum_{b \in \{0,1\}^{k_{i+1}}} \sum_{c \in \{0,1\}^{k_{i+1}}} f_i(z, b, c)$$

Here $k_{i+1} = \lceil \log_2 S_{i+1} \rceil$ is the number of bits indexing gates in layer $i+1$, so the sum ranges over all $2^{k_{i+1}} \times 2^{k_{i+1}}$ possible pairs of gate indices from that layer. The polynomial $f_i$ is defined as:
$$f_i(z, b, c) = \widetilde{\text{add}}_i(z, b, c) \cdot (\tilde{W}_{i+1}(b) + \tilde{W}_{i+1}(c)) + \widetilde{\text{mult}}_i(z, b, c) \cdot (\tilde{W}_{i+1}(b) \cdot \tilde{W}_{i+1}(c))$$

**Why does this work?** The sum ranges over all possible pairs of input gates $(b, c)$. For most pairs, the wiring predicates are zero: gate $z$ doesn't receive input from those gates. Only the *actual* input pair contributes, and for that pair:

- If $z$ is an addition gate: $\widetilde{\text{add}}_i = 1$, contributing $\tilde{W}_{i+1}(b) + \tilde{W}_{i+1}(c)$
- If $z$ is a multiplication gate: $\widetilde{\text{mult}}_i = 1$, contributing $\tilde{W}_{i+1}(b) \cdot \tilde{W}_{i+1}(c)$

The sum collapses to exactly what gate $z$ should compute.

**The magic**: This identity expresses the output of layer $i$ as a *sum*, and we know how to verify sums efficiently using sum-check!



## The Protocol

The GKR protocol reduces verification of the entire circuit to a single check on the input layer.

**Initial Setup**:

1. The prover evaluates the circuit and sends the claimed output $W_0$ to the verifier
2. The verifier picks a random point $r_0 \in \mathbb{F}^{k_0}$ and computes $V_0 = \tilde{W}_0(r_0)$
3. The goal: verify that $V_0$ is correct

**Layer-by-Layer Reduction** (for $i = 0, 1, \ldots, d-1$):

At the start of round $i$, the verifier holds a claim: "$\tilde{W}_i(r_i) = V_i$"

1. **Invoke sum-check**: Using the Layer Reduction Lemma, the verifier expresses $V_i$ as a sum:
   $$V_i = \sum_{b \in \{0,1\}^{k_{i+1}}} \sum_{c \in \{0,1\}^{k_{i+1}}} f_i(r_i, b, c)$$

   The prover and verifier run sum-check on this polynomial. The number of variables is $2k_{i+1}$.

2. **Sum-check conclusion**: Sum-check runs for $2k_{i+1}$ rounds. In each round, the verifier sends a random field element as a challenge. The first $k_{i+1}$ challenges become $s_b \in \mathbb{F}^{k_{i+1}}$; the next $k_{i+1}$ become $s_c \in \mathbb{F}^{k_{i+1}}$. At the end, the verifier must verify:
   $$f_i(r_i, s_b, s_c) = \widetilde{\text{add}}_i(r_i, s_b, s_c) \cdot (\tilde{W}_{i+1}(s_b) + \tilde{W}_{i+1}(s_c)) + \widetilde{\text{mult}}_i(r_i, s_b, s_c) \cdot (\tilde{W}_{i+1}(s_b) \cdot \tilde{W}_{i+1}(s_c))$$

3. **The problem**: The verifier can compute the wiring predicates (she knows the circuit), but she doesn't know $\tilde{W}_{i+1}(s_b)$ and $\tilde{W}_{i+1}(s_c)$; those depend on intermediate gate values only the prover knows.

4. **Reduce two claims to one**: The prover sends the claimed values $\tilde{W}_{i+1}(s_b)$ and $\tilde{W}_{i+1}(s_c)$. But now the verifier has *two* claims to verify in the next round. To maintain efficiency, we reduce them to one:

   - The verifier picks a fresh random challenge $\alpha \in \mathbb{F}$
   - Define $r_{i+1} = s_b + \alpha(s_c - s_b)$ (a random point on the line $\ell(t) = s_b + t(s_c - s_b)$ through $s_b$ and $s_c$)
   - The prover sends a univariate polynomial $q(t) = \tilde{W}_{i+1}(\ell(t))$ of degree $k_{i+1}$
   - The verifier checks $q(0) = \tilde{W}_{i+1}(s_b)$ and $q(1) = \tilde{W}_{i+1}(s_c)$ against the prover's earlier claims
   - Set $V_{i+1} = q(\alpha)$, which equals $\tilde{W}_{i+1}(r_{i+1})$

   The key insight: restricting a multilinear polynomial to a line yields a low-degree univariate polynomial. The random $\alpha$ serves double duty: (1) it tests consistency (if the prover lied about either $\tilde{W}_{i+1}(s_b)$ or $\tilde{W}_{i+1}(s_c)$, the polynomial $q$ won't pass through both claimed values); (2) it produces a fresh random point $r_{i+1}$ that combines both claims into one for the next round

   *Alternative: random linear combination.* Some implementations (Church-Forbes-Spooner 2017) instead use $V_{i+1} = \alpha_1 \cdot \tilde{W}_{i+1}(s_b) + \alpha_2 \cdot \tilde{W}_{i+1}(s_c)$ for fresh random $\alpha_1, \alpha_2$, verifying via a single combined claim. Both approaches achieve the same goal with similar security.

**Final Check**:

After $d$ reductions, the verifier holds a claim: "$\tilde{W}_d(r_d) = V_d$"

But layer $d$ is the input layer! The verifier knows the inputs. She computes $\tilde{W}_d(r_d)$ herself and checks if it equals $V_d$.



## Worked Example: Verifying $(x_1 + x_2) \cdot x_3$

Let's trace through the protocol with $x_1 = 2$, $x_2 = 3$, $x_3 = 4$.

**Honest computation**:

- Layer 2: $W_2(0,0) = 2$, $W_2(0,1) = 3$, $W_2(1,0) = 4$
- Layer 1: $W_1(0) = 5$, $W_1(1) = 4$
- Layer 0: $W_0(0) = 20$

The prover claims the output is 20.

**Round 0: Reducing Layer 0 to Layer 1**

The verifier picks $r_0 = 7$ (say). Recall from earlier that $\tilde{W}_0(u) = 20(1-u)$ (the MLE of the single output value 20). She computes:
$$V_0 = \tilde{W}_0(7) = 20(1-7) = -120$$

The sum to verify (by the GKR Lemma):
$$-120 = \sum_{b,c \in \{0,1\}} \widetilde{\text{mult}}_0(7, b, c) \cdot (\tilde{W}_1(b) \cdot \tilde{W}_1(c))$$

(The $\widetilde{\text{add}}_0$ term vanishes since layer 0 has no addition gates.)

The wiring predicate's MLE: Since $\text{mult}_0(0, 0, 1) = 1$ and is 0 elsewhere:
$$\widetilde{\text{mult}}_0(u, v, w) = (1-u)(1-v)w$$

At $u = 7$:
$$\widetilde{\text{mult}}_0(7, v, w) = (1-7)(1-v)w = -6(1-v)w$$

The sum becomes:
$$\sum_{b,c \in \{0,1\}} -6(1-b)c \cdot (\tilde{W}_1(b) \cdot \tilde{W}_1(c))$$

**Sum-check on this polynomial** proceeds for 2 rounds (one for $b$, one for $c$). The verifier sends random challenges after each round. Suppose these random challenges result in evaluation points $s_b = 3$ and $s_c = 5$; these are where the verifier needs to know $\tilde{W}_1$.

The prover claims:
$$\tilde{W}_1(s_b) = \tilde{W}_1(3) = 5 - 3 = 2, \quad \tilde{W}_1(s_c) = \tilde{W}_1(5) = 5 - 5 = 0$$

Now the verifier has two claims to verify. To reduce to one, she picks random $\alpha = 2$ and considers the line $\ell(t) = s_b + t(s_c - s_b) = 3 + 2t$ passing through $s_b$ (at $t=0$) and $s_c$ (at $t=1$). The prover sends the univariate polynomial $q(t) = \tilde{W}_1(\ell(t)) = 5 - (3 + 2t) = 2 - 2t$. The verifier checks:

- $q(0) = 2$ matches the claimed $\tilde{W}_1(s_b) = 2$ $\checkmark$
- $q(1) = 0$ matches the claimed $\tilde{W}_1(s_c) = 0$ $\checkmark$

The next round's claim becomes $r_1 = \ell(\alpha) = 3 + 2(2) = 7$ with value $V_1 = q(\alpha) = 2 - 2(2) = -2$.

**Round 1: Reducing Layer 1 to Layer 2**

The verifier now holds the claim: $\tilde{W}_1(7) = -2$.

Using the GKR Lemma for layer 1:
$$\tilde{W}_1(7) = \sum_{b,c \in \{0,1\}^2} \left[\widetilde{\text{add}}_1(7, b, c) \cdot (\tilde{W}_2(b) + \tilde{W}_2(c)) + \widetilde{\text{mult}}_1(7, b, c) \cdot (\tilde{W}_2(b) \cdot \tilde{W}_2(c))\right]$$

Another sum-check reduces this to claims about $\tilde{W}_2$ at random points.

**Final Check (Layer 2)**:

Eventually, the verifier holds a claim about $\tilde{W}_2(r_2)$ for some random $r_2$. She computes:
$$\tilde{W}_2(r_2) = 2(1-r_{2,1})(1-r_{2,2}) + 3(1-r_{2,1})r_{2,2} + 4 \cdot r_{2,1}(1-r_{2,2})$$

using the known inputs $x_1 = 2$, $x_2 = 3$, $x_3 = 4$. If this matches the prover's claim, she accepts.



## Why GKR Works

**Completeness**: If the prover is honest, all polynomials they send in sum-check are correct, and all claimed evaluations are accurate. Every check passes.

**Soundness**: Suppose the prover claims a wrong output. Then $\tilde{W}_0$ is incorrect. By the Layer Reduction Lemma, either:

- The sum-check protocol catches a lie (soundness of sum-check), or
- The prover's claimed values for layer 1 are inconsistent

The lie propagates backward through the layers. By induction, if the original claim is false, either some sum-check fails, or the final claim about the input layer is false (which the verifier catches by direct computation).

The soundness error is bounded by:
$$\epsilon \leq \frac{d \cdot \deg(f)}{|\mathbb{F}|}$$

where $d$ is the circuit depth and $\deg(f)$ is the degree of the sum-check polynomial.



## Efficiency Analysis

**Verifier's work**:

- For each layer, participate in a sum-check with $O(\log S)$ rounds (where $S$ is the layer size)
- Evaluate wiring predicates at random points (depends on circuit structure)
- Final check: compute $\tilde{W}_d(r_d)$ in time $O(n)$ where $n$ is the number of inputs

Total: $O(d \log S + n)$ for a depth-$d$ circuit with layers of size at most $S$.

For circuits with "regular" wiring (like FFT butterflies or matrix multiplication), evaluating wiring predicates takes $O(\log S)$ time. The verifier achieves **polylogarithmic verification** in the circuit size!

**Prover's work**:

- Must compute the univariate polynomials for each sum-check round
- Requires summing over all gate values in each layer
- Total: $O(S \log S)$ where $S$ is the total number of gates

The prover does work linear in the circuit size: roughly the cost of evaluation itself, with logarithmic overhead.



## The Circuit Model: Power and Limitations

GKR works for any layered arithmetic circuit. This is remarkably general: any polynomial-time computation can be expressed as a polynomial-size arithmetic circuit.

**Why addition and multiplication suffice**: Over a finite field, these two operations generate all polynomial functions. And any Boolean function can be computed by polynomials: represent true as 1, false as 0, then AND becomes multiplication ($a \cdot b$), NOT becomes subtraction from 1 ($1 - a$), and OR follows from De Morgan ($1 - (1-a)(1-b)$). Since Boolean circuits are universal for computation (any Turing machine can be simulated), arithmetic circuits inherit this universality. The overhead is polynomial: a computation with $T$ steps and $S$ space becomes a circuit of size $O(T \cdot S)$.

**What circuits capture well**:

- Numerical computations (matrix operations, polynomial evaluation)
- Field arithmetic (cryptographic operations)
- Regular patterns (FFT, convolutions)

**Challenges**:

- Data-dependent control flow (if-then-else based on inputs) requires unrolling all branches
- Memory access patterns: Random access memory is expensive to arithmetize
- Bit operations: Non-arithmetic operations require special encoding

Chapter 8 will explore **arithmetization**, the art of expressing computations as circuits, in depth. We'll see how R1CS and QAP provide systematic ways to convert programs into the algebraic form that protocols like GKR can verify.



## The Bigger Picture

GKR represents a conceptual leap in verifiable computation. Instead of designing a custom protocol for each problem:

1. Express the computation as a circuit (a general, mechanical process)
2. Apply GKR (a universal verification protocol)
3. Achieve efficient verification (polylogarithmic in circuit size for regular circuits)

This modularity is powerful. The "frontend" (how to express a computation as a circuit) separates from the "backend" (how to verify circuit evaluation). Improvements to either benefit all applications.

But GKR as originally described is an **interactive** protocol. The prover and verifier exchange messages over multiple rounds. For practical applications (blockchain verification, privacy-preserving credentials) we want **non-interactive** proofs that anyone can verify without interaction.

Chapter 11 will show how to compile interactive protocols like GKR into non-interactive SNARKs using polynomial commitment schemes and the Fiat-Shamir transformation. The journey from sum-check to practical zero-knowledge proofs passes through GKR as a crucial waypoint.

**Is GKR actually used?** For years, GKR was primarily of theoretical interest; the prover overhead and circuit structure requirements made pairing-based SNARKs (Groth16, PLONK) more practical. But GKR is experiencing a resurgence. Modern systems like Lasso and Jolt use GKR-style sum-check reductions as their core verification mechanism, achieving state-of-the-art prover performance for certain computations.

The key insight is that GKR's prover is *native*, working directly with the computation's structure rather than reducing to generic polynomial arithmetic. To see why this matters, consider the alternative. In R1CS-based systems (Groth16, Spartan), every computation, no matter how structured, gets flattened into a uniform constraint system: thousands of equations of the form $a \cdot b = c$. A 256-bit multiplication, a hash function, a simple addition: all become rows in the same homogeneous matrix. The prover then does generic linear algebra over this matrix, blind to the original structure.

GKR is different. The prover traverses the actual circuit layer by layer, computing the sum-check polynomials from the wiring predicates and gate values directly. If your circuit has repeated structure, say 1000 copies of the same subcircuit, the prover can exploit that. If a layer is sparse (few gates), the work is proportionally smaller. The algorithm "sees" the computation's shape.

This becomes dramatic for certain operations. Lookup tables, for instance: proving "this value appears in that table" via R1CS requires encoding the entire table as constraints. GKR-based approaches (like Lasso) can instead prove lookups with work proportional to the number of lookups, not the table size. For memory operations, range checks, and other structured primitives, native provers can be orders of magnitude faster.

GKR is also transparent (no trusted setup) and plausibly post-quantum when instantiated with hash-based commitments. The protocol you've learned here isn't a historical curiosity; it's foundational to an active and growing family of proof systems.



## Key Takeaways

1. **GKR generalizes sum-check**: Instead of custom polynomials per problem, GKR handles any layered arithmetic circuit.

2. **Layer-by-layer reduction**: A claim about layer $i$'s output becomes a claim about layer $i+1$'s output, via sum-check.

3. **Wiring predicates encode structure**: The functions $\text{add}_i$ and $\text{mult}_i$ describe which gates connect to which (known to both prover and verifier).

4. **The Layer Reduction Lemma**: $\tilde{W}_i(z)$ equals a sum over products of wiring predicates and next-layer values (perfect for sum-check).

5. **Two claims become one**: Random linear combinations reduce checking two points to checking one, maintaining efficiency.

6. **Final check on inputs**: The chain of reductions terminates at the input layer, which the verifier can evaluate directly.

7. **Polylogarithmic verification**: For regular circuits, the verifier runs in time $O(d \log S + n)$ (exponentially faster than evaluating the circuit).

8. **Prover overhead is modest**: The prover works in time $O(S \log S)$, only logarithmically more than circuit evaluation.

9. **Circuits are universal**: Any polynomial-time computation has a polynomial-size circuit representation.

10. **GKR is interactive**: Achieving non-interactive proofs requires additional machinery (polynomial commitments, Fiat-Shamir), covered in later chapters.
