# Unified ZK Book - 2026 Edition
## Writing Guide and Style Reference

This document captures the vision, style, and sources for the unified Zero-Knowledge Proofs book. Use this as a reference when writing or reviewing chapters.

---

## Vision

Create a **unified, cohesive book** on Zero-Knowledge Proofs that synthesizes the best content from multiple sources into a single narrative. The result should read as one coherent story, not stacked sections from different documents.

### Primary Sources

1. **Thaler's Survey** - Both the original textbook and the 2025 sum-check update
2. **Protocol Compilation v2** - Excellent worked examples, deep Groth16/PLONK content
3. **Foundational Notes** - Folders 1-4: Sigma protocols, polynomials, core techniques
4. **Philosophical Documents** - "The Philosophy of Polynomials in ZK"
5. **2025 Thaler Sum-Check Survey** - Modern insights on sum-check renaissance

### Output Location
`ZK Theory/Thaler/2026/`

---

## Writing Style

### The Standard: Beautiful Technical Prose

This is a graduate-level text. The audience holds or is pursuing advanced degrees. They don't need hand-holding; they need clarity, precision, and intellectual engagement.

The goal is prose that reads like the best scientific essays: Richard Feynman's lectures, Dijkstra's EWDs, Terence Tao's expository writing, or the opening chapters of Sipser's *Theory of Computation*. These writers achieve elegance not through simplification but through *crystalline clarity*: every sentence earns its place, every abstraction is grounded, and the reader feels they're discovering ideas alongside the author.

### What to Avoid

**Naive analogies**: Cooking recipes, building houses, sealed rooms with notes under doors. These are clichés that signal "I don't trust my reader to handle abstraction." A PhD student doesn't need polynomial commitments explained as "sealing envelopes." They need to understand *why* binding is the right formalization of commitment.

**Excessive scaffolding**: "In this section, we will discuss..." "As we mentioned earlier..." "Let's now turn to..." These are verbal tics, not prose. Cut them.

**False enthusiasm**: "This remarkable result..." "The beautiful insight..." Let the mathematics speak. If an insight is beautiful, the reader will feel it. You don't need to announce it.

**Repetitive structure**: Every section opening with a rhetorical question. Every transition with "But here's the catch." Vary your rhythms.

### Core Principles

1. **Open with intellectual stakes** - Not definitions, but the *tension* that makes the concept necessary.
   - ✓ "The verifier's dilemma is asymmetric: they cannot afford to redo the prover's work, yet they must achieve certainty about its correctness."
   - ✓ "A function evaluated at $2^n$ points contains exponential information. How can a logarithmic number of checks suffice?"
   - X "Imagine you're cooking a meal and need to verify the ingredients..."

2. **Failure-driven progression** - Each technique exists because the previous one hit a fundamental barrier. Show the barrier precisely.
   - "Deterministic verification requires reading the entire witness. Randomness breaks this barrier: not by luck, but by amplifying local inconsistencies into global ones."
   - "The interactive protocol achieves what we want, but interaction itself is a cost. Eliminating it requires replacing the verifier's entropy source."

3. **Concrete before abstract** - Real computations with actual field values first, then the general theorem.
   - Work through $\mathbb{F}_{17}$ before $\mathbb{F}_p$. Show specific matrices before abstract definitions.
   - But don't belabor the concrete. One well-chosen example, fully traced, beats three superficial ones.

4. **Precision over approximation** - Graduate readers can handle nuance. Don't round off the truth.
   - X "This is basically the same as..."
   - ✓ "This differs from the previous construction in that the binding property now relies on computational hardness rather than information-theoretic impossibility."

5. **Grounded abstraction** - When introducing an abstraction, explain what it *captures* and what it *ignores*.
   - "The oracle model abstracts away *how* the verifier accesses polynomial evaluations. It captures the essential constraint: the prover's commitment precedes the verifier's queries. It ignores the mechanism (whether commitments use pairings, hash functions, or trusted hardware)."

6. **Worked examples as discovery** - Examples should feel like guided research, not rote demonstration.
   - Set up a specific instance with concrete values
   - Trace the computation step by step
   - Pause to note where things could go wrong (where a cheater would fail)
   - End with verification: does our answer satisfy the claimed property?

### Prose Rhythms

**Vary sentence length.** A sequence of complex sentences exhausts the reader. Follow a long sentence with a short one. The short sentence lands harder.

**Use paragraph breaks for emphasis.** A single-sentence paragraph draws the eye. Use sparingly.

**Let mathematics carry weight.** When a formula captures an idea precisely, state it and let it breathe. Don't immediately paraphrase in words what the symbols already say clearly.

**Transitions should advance, not summarize.** Instead of "Having established X, we now turn to Y," try "X leaves open the question of Y" or simply begin discussing Y. The reader can follow.

---

### The Deeper Aesthetic: A Book That Resonates

This is a technical book. It is also more than that.

The aspiration is something like *Gödel, Escher, Bach*: not in structure (we need not alternate dialogues with chapters), but in spirit. Hofstadter's book succeeds because it treats mathematical ideas as worthy of the same attention we give to art and music. The strange loops aren't just clever; they illuminate something about minds, meaning, and the boundaries of formal systems. The reader finishes the book changed, not just informed.

We want that quality. A graduate student who reads this book should come away not merely knowing how STARKs work, but feeling something about what it means that local corruptions become global inconsistencies, that randomness can create certainty, that a polynomial's rigidity is a kind of crystalline integrity.

**The literary sensibility.** Good mathematical writing shares something with the writers who handle ideas that resist easy expression: Borges with his labyrinths and infinite libraries, Kafka with his parables of incomprehensible systems, Dostoevsky with his characters thinking aloud about the unbearable, Pessoa fragmenting identity into heteronyms, Cărtărescu building cathedrals of dream-logic, Saer circling the same events from ever-shifting angles. These writers don't explain; they evoke, circle, return.

What does this mean for technical prose? Not that we write fiction. But that we recognize:

- **Ideas have emotional weight.** The moment you grasp why the Schwartz-Zippel lemma works, why a polynomial *must* disagree with any incorrect claim almost everywhere, is not just intellectual. There's something like relief, or recognition. Good writing doesn't announce this ("The beautiful insight...") but creates the conditions for the reader to feel it.

- **History is story.** The 1960 Reed-Solomon paper for spacecraft. Ben-Sasson's two decades working on PCPs before STARKs. The trusted setup ceremonies and their paranoid security. These are not ornaments. They are how ideas become real: how abstractions acquire weight by being connected to human effort, contingency, and time.

- **Recursion and return.** The best mathematical concepts rhyme with each other. Merkle trees and polynomials both turn local changes into global signatures. The simulator paradigm in zero-knowledge mirrors the prover-verifier dance. When we notice these echoes, we should mark them, briefly and without belaboring, because they're part of what makes the subject cohere.

- **Silence and space.** Not every paragraph needs to *do* something. Sometimes a short statement followed by a section break lets an idea settle. "The error doesn't hide; it creates a global inconsistency." Full stop. Move on. The reader can sit with it.

**What this permits (when it works):**

- Opening a section with a historical anecdote that illuminates the problem before formalizing it
- A single evocative sentence between technical paragraphs, if it genuinely clarifies
- Vivid concrete detail: "the Voyager spacecraft still transmitting from interstellar space"
- Occasional poetic register: "corrupt even one evaluation, and the spell breaks"
- Noticing when a mathematical fact is uncanny or paradoxical, and naming the feeling

**What this does NOT permit:**

- Forced whimsy or manufactured wonder
- Analogies that condescend or trivialize
- Digressions that don't earn their place
- Emotional language that substitutes for understanding
- Anything that reads like marketing or hype

The test: would a thoughtful reader feel manipulated, or would they feel the writer respects both the material and the reader's intelligence? Feynman passes this test. Quanta Magazine passes it. GEB passes it. We should aim there.

**A concrete example.** Consider how to introduce Reed-Solomon codes:

*Merely technical*: "Reed-Solomon codes are error-correcting codes based on polynomial evaluation. To encode $k$ symbols, interpret them as polynomial coefficients..."

*Better*: "The key insight comes from an unexpected source: a 1960 paper by Irving Reed and Gustave Solomon, written not for cryptography but for sending data to spacecraft. The problem they faced was corruption. Radio signals degrade over interplanetary distances. Cosmic rays flip bits..."

The second version takes thirty more words to reach the same definition, but the reader now *cares*. The abstraction is grounded in a real problem, a specific historical moment, a concrete physical reality. When we later say "This is the magic that makes Reed-Solomon codes work on CDs, DVDs, QR codes, and the Voyager spacecraft still transmitting from interstellar space," the sentence lands because the reader has been prepared to feel the scope of what polynomials made possible.

This is the spirit. Use it when it works. Omit it when it would feel forced. Trust your judgment.

### Structural Elements

**Section breaks**: Use `---` between major sections for visual breathing room.

**Emphasis patterns**:

- **Bold** for key terms on first introduction
- *Italics* for emphasis and technical terms in running text
- `Code blocks` for algorithms or ASCII diagrams

**Mathematical notation**:

- Use LaTeX: `$inline$` and `$$display$$`
- Prefer explicit notation: $\mathbb{F}_p$ not $GF(p)$
- Define notation on first use

**Worked examples format**:
```
**Worked Example:** [Brief description]

[Setup with specific values]

[Step-by-step computation]

Verification: [Check the result] ✓
```

**Key takeaways**: End each chapter with a numbered list of 6-10 key points.

**Punctuation**:

- Avoid em-dashes (—) mid-sentence. Use commas, colons, or parentheses instead.
- Hyphenated compound terms are fine: sum-check, zero-knowledge, multi-scalar.

---

### Opportunities for Resonance (Chapter-Specific Notes)

The following are places where the deeper aesthetic might naturally emerge. These are suggestions, not requirements: the right moment depends on the material and on whether the addition genuinely illuminates rather than decorates.

**Chapter 2 (Polynomials)**: The Schwartz-Zippel lemma has an almost moral quality: a polynomial cannot lie consistently. It must betray itself almost everywhere. This is worth lingering on (not explaining further, but letting the reader feel the strangeness that certainty can emerge from randomness, that structure *compels* honesty).

**Chapter 3 (Sum-Check)**: The courtroom analogy already gestures at this, but the deeper idea is that *lies propagate*. A single false claim about a sum forces the prover into a cascade of falsehoods, each increasingly likely to be detected. There's something almost Kafkaesque here: the bureaucracy of consistency traps the liar.

**Chapter 6 (Commitments)**: Commitment is philosophically rich. We ask: what does it mean to be *bound* to a claim you haven't yet revealed? The binding property is a kind of time travel: your present self constrains your future self. Borges would appreciate this.

**Chapter 10 (FRI)**: The folding process, reducing a polynomial to half its degree again and again until only a constant remains, has a meditative quality. Each round strips away complexity until the essence is exposed. "What survives the folding?"

**Chapter 12 (Groth16)**: The trusted setup ceremony is inherently dramatic. People genuinely destroyed hard drives, used radioactive decay for randomness, broadcast parameters from remote locations. This paranoid opera deserves its moment: not as spectacle, but as acknowledgment that cryptographic security sometimes requires rituals of destruction.

**Chapter 15 (STARKs)**: Already enhanced with Reed-Solomon history. The broader theme: transparency as a design principle. No secrets, no ceremonies. The proof that trust can be eliminated, that verification can stand alone, self-contained, answering to nothing but mathematics.

**Chapter 17 (Zero-Knowledge)**: The simulator paradigm is philosophically vertiginous. The proof convinces precisely because it could have been faked, but the prover didn't fake it, and the verifier can't tell the difference. What does it mean that indistinguishability from a lie is the guarantee of truth?

**Chapter 22 (Composition/Recursion)**: A proof that verifies a proof that verifies a proof... The infinite regress has a Borgesian flavor. "The Library of Babel" contains every possible proof; recursive SNARKs compress them into a single page.

**General principle**: The moments of resonance should emerge from the mathematics itself, not be imposed upon it. The best passages will feel like *noticing* something the equations already imply: not adding commentary, but giving voice to what was always there.

---

## Depth and Length

### Target: Graduate Textbook Level

- **Word count**: 5,000-7,000 words per chapter
- **Mathematical rigor**: Full proofs for key theorems, proof sketches for standard results
- **Prerequisites assumed**: Undergraduate algebra (groups, fields, polynomials), basic probability, computational complexity basics (P, NP)

### Depth Calibration
Each chapter should include:

- Formal definitions with motivation
- At least 2-3 worked examples with actual field values
- Intuitive explanations alongside formal statements
- Connections to other chapters (forward and backward references)
- Historical context where relevant

### What to Include vs. Omit
**Include:**

- Full protocol specifications
- Soundness and completeness proofs (or detailed sketches)
- Complexity analysis
- Security parameter discussions
- Practical trade-offs

**Omit:**

- Implementation details (code)
- Benchmarks and performance numbers (change too quickly)
- Detailed security proofs in random oracle model (reference instead)
- Exhaustive coverage of all variants

---

## Two Narrative Arcs

### Bottom-Up (for foundations, Parts I-II)
"What tools do we need?" → "Why that approach?" → "How does it work?"

Used for: Polynomials, multilinear extensions, finite fields, commitment basics

### Goal-Driven (for protocols, Parts III-IV)
Start with the goal, work backwards to explain each component.

Used for: GKR, Groth16, PLONK, STARKs

---

## Chapter-by-Chapter Sources and Focus

### Part I: The Problem and The Insight (Chapters 1-3)

**Chapter 1: The Trust Problem in Computation** ✓ COMPLETE

- Sources: Thaler Ch. 1, Introduction notes
- Focus: Complexity classes (P, NP, co-NP, #P, PSPACE), verification asymmetry, GMR model, Graph Non-Isomorphism protocol, IP=PSPACE significance, proofs vs arguments, Fiat-Shamir

**Chapter 2: The Alchemical Power of Polynomials** ✓ COMPLETE

- Sources: Philosophy of Polynomials, Thaler Ch. 2-3
- Focus: SAT/#SAT motivation, Lagrange interpolation, Schwartz-Zippel, Reed-Solomon codes, Freivald's algorithm, polynomial rigidity, vanishing polynomial technique, holographic principle connection

**Chapter 3: The Sum-Check Protocol**

- Sources: Thaler Ch. 4, Protocol compilation sum-check section, Sum-check Protocol doc
- Focus: The protocol specification, courtroom analogy (lies propagate), full worked example over small field, #SAT application, why this enables everything else
- Key worked example: 3-variable polynomial sum over boolean hypercube

### Part II: The Polynomial Toolkit (Chapters 4-6)

**Chapter 4: Multilinear Extensions**

- Sources: Protocol compilation v2 Parts I-II, Polynomials folder
- Focus: Boolean hypercube, MLE existence and uniqueness, Lagrange basis for hypercube, efficient evaluation algorithms, tensor product structure, FWHT connection
- Key worked example: MLE of a 3-bit function

**Chapter 5: Univariate Polynomials and Finite Fields**

- Sources: Protocol compilation v2, Polynomials folder
- Focus: Multiplicative subgroups, roots of unity, FFT/IFFT, coset structure, Lagrange over roots of unity, when to use univariate vs multilinear
- Key worked example: FFT over $\mathbb{F}_{17}$ with 4th roots of unity

**Chapter 6: Commitment Schemes**

- Sources: Commitment Schemes folder, Thaler Ch. 12-13
- Focus: Binding and hiding properties, Pedersen commitments, homomorphic properties, setup for polynomial commitments
- Key worked example: Pedersen commitment with discrete log

### Part III: Building Proofs (Chapters 7-11)

**Chapter 7: The GKR Protocol**

- Sources: Thaler Ch. 4 GKR section, Protocol compilation
- Focus: Layered arithmetic circuits, wiring predicates as polynomials, layer-by-layer reduction, combining with sum-check
- Key worked example: 3-gate circuit verification

**Chapter 8: From Circuits to Polynomials (Arithmetization)**

- Sources: Protocol compilation v2 (Groth16 arithmetization section), Thaler Ch. 6
- Focus: R1CS constraint system, $(A \cdot s) \circ (B \cdot s) = (C \cdot s)$, QAP transformation, divisibility condition
- Key worked example: $x^3 + x + 5 = 35$ full arithmetization

**Chapter 9: Polynomial Commitment Schemes**

- Sources: KZG doc, IPA doc, Thaler Ch. 14-15, Protocol compilation
- Focus: KZG (pairings, trusted setup, batch opening), IPA/Bulletproofs (recursive folding), trade-off comparison
- Key worked examples: KZG commitment and opening proof, IPA folding step

**Chapter 10: Hash-Based Commitments and FRI**

- Sources: STARKs folder, Thaler, 2025 survey Section 6
- Focus: Merkle trees for polynomials, FRI folding technique, query complexity, soundness analysis, post-quantum advantage
- Key worked example: FRI folding on a degree-7 polynomial

**Chapter 11: The SNARK Recipe**

- Sources: Thaler Ch. 10, Protocol compilation overview
- Focus: PIOP + PCS + Fiat-Shamir = SNARK, modularity benefits, the compilation pipeline
- This is a synthesis chapter, showing how pieces fit together

### Part IV: Complete Proof Systems (Chapters 12-15)

**Chapter 12: Groth16**

- Sources: Protocol compilation v2 (Groth16 section), Thaler Ch. 17
- Focus: Linear PCPs, the full Groth16 protocol, trusted setup (Powers of Tau + circuit-specific), why Phase 2 can't be universal, security in Generic Bilinear Group Model
- Key worked example: Complete Groth16 proof for small circuit

**Chapter 13: PLONK**

- Sources: PLONK folder, Protocol compilation
- Focus: Permutation argument, copy constraints, grand product check, universal SRS, comparison with Groth16
- Key worked example: Copy constraint verification

**Chapter 14: Lookup Arguments**

- Sources: Lookup Arguments.md, Plookup.md, Jolt Theory (concepts)
- Focus: The lookup problem (proving values exist in a table), grand product argument, Plookup protocol, subset-to-permutation reduction, modern variants (Lasso, LogUp)
- Key worked example: Full Plookup verification with F and G polynomials

**Chapter 15: STARKs**

- Sources: STARKs folder, Protocol compilation, Thaler
- Focus: AIR (Algebraic Intermediate Representation), STARK pipeline using FRI, transparency, post-quantum claims
- Key worked example: AIR for Fibonacci computation

**Chapter 16: Sigma Protocols**

- Sources: Sigma Protocols folder (7 files), Thaler Ch. 12
- Focus: 3-message structure, Schnorr protocol, special soundness, honest-verifier ZK, Fiat-Shamir, OR-compositions
- Key worked example: Schnorr identification

### Part V: Zero-Knowledge Properly (Chapters 17-18)

**Chapter 17: The Zero-Knowledge Property**

- Sources: Thaler Ch. 11, ZK notes
- Focus: Simulator paradigm, "learns nothing" formalization, perfect/statistical/computational ZK, HVZK vs malicious verifier, common misconceptions
- Key example: Simulator for Graph Non-Isomorphism

**Chapter 18: Making Proofs Zero-Knowledge**

- Sources: Thaler Ch. 13, Protocol compilation
- Focus: Commit-and-prove, masking polynomials, blinding in Groth16, ZK in random oracle model
- Key worked example: Adding ZK to a sum-check instance

### Part VI: The Sum-Check Renaissance (Chapters 19-21)

**Chapter 19: Fast Sum-Check Proving**

- Sources: 2025 Thaler survey Sections 1.1, 4.1-4.2
- Focus: Historical detour (why SNARKs moved away), Fiat-Shamir insight (removing interaction twice is wasteful), linear-time dense proving, sparse sum-check with prefix-suffix decomposition
- Conceptual chapter with complexity analysis

**Chapter 20: Minimizing Commitment Costs**

- Sources: 2025 survey Sections 5.1-5.3, Twist and Shout (concepts only)
- Focus: Two-stage paradigm (commit then prove well-formed), batch evaluation, virtual polynomials, small-value preservation
- Conceptual chapter focused on efficiency principles

**Chapter 21: The Two Classes of PIOPs**

- Sources: 2025 survey Sections 3.1-3.2, 5.4
- Focus: Quotienting-based (univariate, vanishing polynomials) vs sum-check-based (multilinear, RLC), worked comparison, memory checking problem, brief Lasso/Jolt mention
- Key worked example: $a \circ b = c$ in both paradigms

### Part VII: Advanced Topics (Chapters 22-24)

**Chapter 22: Composition and Recursion**

- Sources: Thaler Ch. 18, Recursive Proofs doc
- Focus: SNARK composition (inner + outer), IVC, folding schemes (Nova)
- Conceptual with high-level examples

**Chapter 23: Choosing a SNARK**

- Sources: Thaler Ch. 19, Bird's Eye View notes
- Focus: Taxonomy (trusted/transparent, pre/post-quantum), comparison table, matching systems to requirements
- Decision-oriented chapter

**Chapter 24: MPC and ZK: Parallel Paths**

- Sources: MPC folder
- Focus: MPC problem and adversary models, secret-sharing MPC, Beaver triples, garbled circuits, oblivious transfer, MPC-in-the-head as ZK construction technique, threshold cryptography
- Rationale: MPC-in-the-head directly constructs ZK proofs, making this a core technique rather than supplementary material

**Chapter 25: Frontiers and Open Problems**

- Focus: Post-quantum, formal verification, hardware acceleration, zkVM landscape (Jolt, RISC0, SP1 brief survey)
- Forward-looking, less depth needed

**Chapter 26: ZK in the Cryptographic Landscape**

- Focus: Programmable cryptography overview, four pillars (ZK, MPC, FHE, iO), FHE with LWE math, program obfuscation with iO definitions, witness encryption, why ZK succeeded first, convergence of approaches
- Book conclusion: places ZK in broader context

### Appendices

**Appendix A: Cryptographic Primitives**

- Sources: Cryptographic Primitives folder (Ch. 1-6)
- Focus: Secure random sampling, modulo bias, NUMS constructions, Shamir secret sharing, Feldman VSS

**Appendix B: Historical Timeline**

- Focus: Development of ZK proofs 1985-present, GMR to modern zkVMs, visual timeline, key themes (theory to practice, trusted setup debate, sum-check renaissance)

**Appendix C: Field Equations Cheat Sheet**

- Focus: Quick reference for core equations (Schwartz-Zippel, MLE formulas, sum-check, R1CS/QAP, KZG, FRI, PLONK, Groth16, lookup arguments, IPA, Fiat-Shamir), complexity summary table

---

## Cross-Reference Patterns

### Forward References (create anticipation)

- "We'll see exactly how in Chapter 3 when we study the sum-check protocol."
- "The sum-check protocol (Chapter 3) exploits exactly this structure."
- "This multilinear extension is the key to Chapter 4."

### Backward References (reinforce connections)

- "Recall from Chapter 2 that polynomials are rigid..."
- "Using the Schwartz-Zippel lemma we established earlier..."
- "This is the same distance-amplifying property we saw with Reed-Solomon codes."

---

## Quality Checklist

Before considering a chapter complete:

- [ ] Opens with problem/paradox, not definitions
- [ ] At least 2-3 worked examples with actual field values
- [ ] Formal definitions for all key concepts
- [ ] Soundness/completeness analysis where applicable
- [ ] Forward references to upcoming chapters
- [ ] Backward references to earlier chapters
- [ ] Key takeaways section (6-10 points)
- [ ] Word count in 5,000-7,000 range
- [ ] All LaTeX renders correctly
- [ ] Consistent notation with other chapters

---

## File Naming Convention

```
01 - The Trust Problem.md
02 - The Alchemical Power of Polynomials.md
03 - The Sum-Check Protocol.md
04 - Multilinear Extensions.md
05 - Univariate Polynomials and Finite Fields.md
06 - Commitment Schemes.md
07 - The GKR Protocol.md
08 - From Circuits to Polynomials.md
09 - Polynomial Commitment Schemes.md
10 - Hash-Based Commitments and FRI.md
11 - The SNARK Recipe.md
12 - Groth16.md
13 - PLONK.md
14 - Lookup Arguments.md
15 - STARKs.md
16 - Sigma Protocols.md
17 - The Zero-Knowledge Property.md
18 - Making Proofs Zero-Knowledge.md
19 - Fast Sum-Check Proving.md
20 - Minimizing Commitment Costs.md
21 - The Two Classes of PIOPs.md
22 - Composition and Recursion.md
23 - Choosing a SNARK.md
24 - MPC and ZK Parallel Paths.md
25 - Frontiers and Open Problems.md
26 - ZK in the Cryptographic Landscape.md
Appendix A - Cryptographic Primitives.md
Appendix B - Historical Timeline.md
Appendix C - Field Equations Cheat Sheet.md
```

---

## Progress Tracking

| Chapter | Status | Word Count | Notes |
|---------|--------|------------|-------|
| 00 - Book Guide | ✓ Complete | - | This document |
| 01 - Trust Problem | ✓ Complete | ~6,800 | Expanded with complexity classes, GNI worked example |
| 02 - Polynomials | ✓ Complete | ~6,500 | Expanded with SAT, Reed-Solomon, Freivald |
| 03 - Sum-Check | ✓ Complete | ~4,800 | Consolidated examples, added sum-check renaissance forward ref |
| 04 - Multilinear | ✓ Complete | ~4,500 | Full MLE theory with worked examples |
| 05 - Univariate | ✓ Complete | ~4,800 | FFT, roots of unity, vanishing polynomial |
| 06 - Commitments | ✓ Complete | ~4,200 | Pedersen, binding/hiding, homomorphic properties |
| 07 - GKR | ✓ Complete | ~4,500 | Layer reduction, wiring predicates, worked example |
| 08 - Arithmetization | ✓ Complete | ~6,200 | Enhanced: witness depth, Spartan, bit costs |
| 09 - PCS | ✓ Complete | ~5,500 | KZG, IPA with verification, Dory, comprehensive comparison |
| 10 - FRI | ✓ Complete | ~3,800 | Split-and-fold, worked example |
| 11 - SNARK Recipe | ✓ Complete | ~3,500 | IOP + PCS + Fiat-Shamir synthesis |
| 12 - Groth16 | ✓ Complete | ~5,800 | QAP, Linear PCP, trusted setup, verification equation derivation |
| 13 - PLONK | ✓ Complete | ~5,200 | Universal setup, permutation argument, accumulator polynomial |
| 14 - Lookup Arguments | ✓ Complete | ~4,800 | Plookup, grand product, subset-to-permutation reduction |
| 15 - STARKs | ✓ Complete | ~5,100 | AIR, state machine model, composition polynomial, FRI link |
| 16 - Sigma Protocols | ✓ Complete | ~4,500 | Three-move structure, Schnorr, Pedersen, Fiat-Shamir |
| 17 - ZK Property | ✓ Complete | ~4,600 | Simulator paradigm, flavors, HVZK vs malicious, rewinding |
| 18 - Making ZK | ✓ Complete | ~4,400 | Commit-and-prove, masking polynomials, Groth16/PLONK techniques |
| 19 - Fast Sum-Check | ✓ Complete | ~4,200 | Halving trick, sparse sums, prefix-suffix, streaming provers |
| 20 - Commitment Costs | ✓ Complete | ~4,000 | Batch evaluation, virtual polynomials, Shout/Twist, small values |
| 21 - Two PIOP Classes | ✓ Complete | ~4,300 | Quotienting vs sum-check, domain choice, wiring constraints |
| 22 - Composition | ✓ Complete | ~4,500 | SNARK composition, IVC, folding schemes, Nova |
| 23 - Choosing SNARK | ✓ Complete | ~3,200 | Comparison table, application guidance, trade-off dimensions |
| 24 - MPC and ZK | ✓ Complete | ~3,200 | MPC problem, secret-sharing, garbled circuits, OT, MPC-in-the-head |
| 25 - Frontiers | ✓ Complete | ~3,000 | Post-quantum, zkVMs, formal verification, open problems |
| 26 - Landscape | ✓ Complete | ~3,500 | Programmable cryptography landscape, FHE, iO, witness encryption |
| Appendix A | ✓ Complete | ~3,500 | Random sampling, NUMS, Shamir, Feldman VSS, hash functions |
| Appendix B | ✓ Complete | ~1,800 | Historical timeline 1985-2024, visual timeline, key themes |
| Appendix C | ✓ Complete | ~1,500 | Field equations cheat sheet, complexity summary, quick reference |

**Total: 26 chapters + 3 appendices**
**Completed: 29/29** ✓
