# Core Mechanism

> This document is the intellectual core of Solomonoff Loop. It states the
> problem precisely, gives the theoretical grounding, defines the loop and its
> five invariants, and — just as importantly — marks the boundary between what is
> *grounded* and what is *speculation*.

---

## 1. Terminology

| Term | Meaning |
|---|---|
| **Proposer** | The LLM that synthesizes candidate programs from its own latent knowledge. |
| **Anvil** (the verifier) | The *mechanical* verifier: an executor, a proof checker, or a held-out-data match. **Never an LLM.** |
| **Crystal** (the program layer) | The growing library of verified, executable, legible programs. |
| **Guide** | An LLM role with two jobs: anti-triviality filtering and specification-grounding audit. |
| **Distillation** | Crystal → weights (fine-tuning on Crystal-resampled I/O). |
| **Grounding** | The exogenous signal obtained by executing programs against reality. |
| **Description length** | `L = L(data|program) + L(program)` — the MDL / algorithmic-probability selection criterion. |

---

## 2. The core claim

> An explicit, executable, description-length-ranked program layer can serve
> simultaneously as **(a)** exogenous grounding that prevents collapse, **(b)** an
> introspectable functional self-model, and **(c)** a non-collapsing curriculum —
> and a loop that builds such a layer is strictly more stable than any
> self-improvement method whose evaluator is learned or self-referential.

The claim is *falsifiable*: it stands or falls on whether the Crystal produces a
measurable gain over "verify but do not retain" (ablation D in the experiment
design). If it does not, the framework is merely relocating collapse, not
escaping it.

---

## 3. The three failure modes

Every self-improvement method in the literature fails in one of three ways. All
three are *the same failure* — a weak improvement signal — wearing different masks.

### 3.1 The self-confirming loop

**Mechanism.** When the generator and the evaluator share weights, their biases
correlate. The 2025 work *Breaking the Self-Confirming Loop* diagnoses the
mechanism in self-rewarding RL: confidence-coupled rewards systematically
over-reward *high-confidence mistakes*, so the loop preferentially reinforces
exactly the errors the model is most sure about. This is the general case of
reward hacking — the explicit "game the judge" is just the adversarial special
case.

**Why it is fatal.** The bias compounds across iterations. There is no
self-correction; the loop is self-confirming by construction.

**The design's answer.** Invariant 1: the Anvil shares *zero* weights with the
Proposer because it is not a neural model at all.

### 3.2 Model collapse / entropy decay

**Mechanism.** Training on one's own output loses the tails of the distribution.
Shumailov et al. (*The Curse of Recursion*, Nature 2024) showed this is inevitable
even in near-ideal conditions, driven by two compounding error sources (statistical
approximation error from finite sampling, and functional approximation error from
capacity limits). The 2026 paper *On the Limits of Self-Improving LLMs*
(2601.05280) formalizes this as a dynamical system: as the fraction of external
grounding α → 0, the model mean follows a random walk bounded only by the support
diameter — *drift*, not just mode-dropping.

**Why it is fatal.** No practical self-improvement loop can afford to lose the
tails — the tails are where generalization and surprise live.

**The design's answer.** Invariants 2 and 3: the Crystal is a *resampler*, not a
memory. A verified program encodes the generative mechanism; resampling it
reproduces the full distribution *including the tails*. This is the concrete
instantiation of the 2601.05280 escape route (see §4).

### 3.3 Diversity collapse

**Mechanism.** In co-evolutionary loops, the *proposer* converges to the narrow band
of problems that satisfy the reward. Self-play curricula starve the solver of
genuinely new challenges. Novelty is a consumable resource that closed loops
deplete.

**Why it is fatal.** The loop becomes excellent at a shrinking region of task
space — a local optimum disguised as progress.

**The design's answer.** Invariant 4: the Guide's anti-triviality job is precisely
to reject "difficulty without value," and its grounding audit keeps the curriculum
anchored to reality rather than to the model's prior.

---

## 4. The theoretical escape

Two 2026 papers argue, from different directions, that the escape from these
failures is *neurosymbolic* — and they converge on the same primitive.

### 4.1 Algorithmic probability preserves the distribution

*On the Limits of Self-Improving LLMs* (2601.05280) proves that **purely
distributional self-training must collapse**, because it relies on the empirical
properties of finite samples. The proof's constructive escape: **program synthesis
guided by algorithmic probability** (Coding Theorem Method CTM, Block Decomposition
Method BDM). The key insight:

> A minimal program `p*` that generates the observed data implicitly defines the
> **entire** distribution — including the tails that a finite sample cannot see.
> Learning "more concise, algorithmically probable programs that explain the data"
> is a different objective from "lower perplexity," and it is the one that does not
> decay.

### 4.2 Executable symbols supply the missing self-model

*Self-reference in LLMs: the introspection threshold for RSI* (2607.04277) argues
that sustained recursive self-improvement requires a **faithful functional
self-model** — a von Neumann-style complexity threshold. But standard Transformers
are feedforward and bounded by the uniform TC⁰ circuit class, so they cannot read
their own weights or perform fixed-point iteration; they are *structurally
incapable* of true introspection. The missing primitive is a legible,
executable representation of the system's own competence — which is exactly what
the Crystal is.

The synthesis: **the Crystal is simultaneously the algorithmic-probability anchor
(§4.1) and the functional self-model (§4.2).** That is the novel combination.
Neither paper alone proposes the loop; the loop is the contribution.

---

## 5. The loop

Formalized as a per-round procedure:

```
round t:
  1. PROPOSE     For each active target, the Proposer synthesizes a batch of
                 candidate programs from k observed samples.
  2. VERIFY      The Anvil runs each candidate on m held-out inputs the model
                 never saw. Candidates that fail are discarded.  (This is the
                 exogenous signal — it forces generalization, not memorization.)
  3. RANK        Surviving candidates are ordered by description length
                 L = L(data|program) + L(program); keep the shortest per target.
  4. CRYSTALLIZE Add survivors to the Crystal.
  5. GROUND      (i)  distill: fine-tune the Proposer on I/O resampled from the
                      Crystal, including tail inputs;
                 (ii) invoke: keep the Crystal callable at inference.
  6. INTROSPECT  The Proposer reads the Crystal and may propose merges,
                 refactors, or patches. A change is accepted only if it still
                 verifies on all targets' held-out sets.
```

Steps 2, 3, and the acceptance gate of 6 are **mechanical**. Steps 1, 5, and the
proposals of 6 are **neural**. The boundary between them is the entire point.

### Why step 2 is the grounding

Step 2 is the only place fresh, non-self-generated signal enters. Execution against
held-out inputs is *not* derived from the model's soft distribution — a program
either produces the correct held-out output or it does not. This is the α > 0 term
in the collapse theorem, and it is why the loop does not drift.

---

## 6. The five invariants (the laws)

1. **The Anvil is mechanical, never an LLM.** Executor, proof checker, or
   held-out-data match. This is the anti-self-confirming-loop law. An LLM-as-judge
   anywhere re-opens failure mode 3.1.

2. **The Crystal is dual-use: distill for fluency, invoke for tails.** Distillation
   (invariant 2's "distill" branch) is how the model internalizes the mechanism's
   *structure*; invocation (the "invoke" branch) is how the *tails* survive the
   model's finite approximation. Both are required; either alone fails.

3. **The Crystal is a resampler, not a memory.** The anti-collapse law. A program
   is a generator of an infinite grounded I/O stream; resampling it (not replaying
   finite samples) is what defeats entropy decay.

4. **The Guide has two jobs: anti-triviality and spec-grounding audit.** The
   anti-diversity-collapse law. The Guide is the *only* LLM in a gatekeeping role,
   and it is itself gated by the Anvil's track record.

5. **Selection is by description length, not model likelihood.** The anti-circularity
   law. The model's own perplexity is a function of its own beliefs; description
   length is a function of the program's compressibility, which is independent of
   the model.

---

## 7. The meta-loop (who verifies the verifier)

The sharpest objection is that step 2 only pushes the trust boundary one level up:
**who writes the tests?** If the Proposer authors both program and test, execution
is exogenous to the generator's *distribution* but not to the model's *beliefs
about reality*.

The answer is recursive grounding:

> **Specifications are themselves falsifiable programs.** A spec survives only if
> the programs it certifies keep predicting reality. The spec's track record —
> "does it produce programs that generalize to held-out data?" — is measurable and
> exogenous.

Reality is the one unforgeable external referee, and it can be invoked at *whatever
level has an observable outcome*. This closes the regress: there is no infinite
tower of verifiers; there is a tower that bottoms out in held-out samples of
reality.

Formally, the meta-loop is:

```
meta-round:
  1. Specs are programs in a (possibly richer) DSL.
  2. A spec is evaluated by the predictive success of the programs it admits
     on held-out data, over time.
  3. Specs that admit reality-predicting programs persist; specs that admit
     overfit programs decay in weight.
```

This is the mechanism by which **the evaluator itself co-evolves** — filling the
gap that the 2025–26 frontier (CORAL, Aspire, Darwin Gödel Machine) explicitly
names as unsolved.

---

## 8. Why this is more stable than prior work: the fault-tolerance asymmetry

The single most important reason the design is *strictly better* than self-rewarding
or self-play, stated as an asymmetry:

| Error type | Where it occurs | Consequence |
|---|---|---|
| Synthesis error (Proposer writes a wrong program) | Step 1 | **Detected** by the Anvil → discarded. Cost: one wasted candidate. |
| Evaluator error (judge gives a wrong reward) | (in self-rewarding: the judge) | **Silently absorbed** → reinforced → compounds. |

A weak synthesizer merely *slows* the loop (fewer programs verify per round). A
weak evaluator *corrupts* the loop (bad rewards get reinforced, and the bias
compounds). Furthermore, synthesis quality is *bootstrappable* — the model gets
better at writing programs by observing which ones verify — whereas a bad-judge
bottleneck is self-reinforcing and has no escape.

**Corollary: the loop's only irrecoverable failure mode is the evaluator, and the
evaluator is mechanical.** This is why the design concentrates all its correctness
on invariant 1.

### The Crystal is a built-in frozen control

A second, independent stability property: every program in the Crystal is a
*checkable assertion of capability*. Regression = a previously-verified program
now fails its held-out set. This is a frozen control that exists by construction —
the antidote to the "phantom gains" problem (see the audit protocol in
[experiment-design.md](experiment-design.md)). Improvement is *auditable*, not
merely reported.

---

## 9. Formal sketch

*This section is a sketch; the full formalization is open work.*

**Claim (collapse theorem, paraphrase of 2601.05280).** Let the self-training
operator update a distribution `Q_{t+1} = T(P'_t) + ε_t` where `P'_t` is the
empirical sample and `α_t` is the fraction of exogenous data. If `α_t → 0`, then
the model mean `μ_t` follows `μ_{t+1} ≈ μ_t + ξ_t` (a random walk), so
`Var(μ_t)` grows linearly and the distribution drifts from the truth.

**Claim (the Crystal's escape).** The Crystal replaces the empirical sample `P'_t`
with a *program-induced* distribution `P(p*)`, where `p*` is the shortest program
consistent with the data. Because `p*` defines the support of the distribution —
including tails invisible in `P'_t` — resampling from `P(p*)` does not suffer
finite-sample entropy decay. The external-grounding term `α_t` is supplied not by
"more human data" but by **execution against held-out reality**, which is an
inexhaustible, unforgeable source.

**Open quantity.** The "exchange rate" — *how little external grounding suffices* —
is unestablished. No one has measured the minimum α below which a loop degrades.
This is a first-class target of the experiment design.

**The description-length ladder (what is actually computable):**

Kolmogorov complexity `K(x)` is uncomputable in general (Chaitin). The practical
substitutes, in order of rigor vs. scale:

1. **MDL / symbolic-regression complexity** — `L = L(data|program) + L(program)`,
   what PySR and symbolic regression already minimize. *The primary stand-in.*
2. **gzip / Normalized Compression Distance** — a rough lower bound on K, O(n),
   scalable. *The engineering default.*
3. **CTM / BDM** — exact-ish algorithmic probability for short (≤ ~12-bit) blocks;
   catches structure (e.g. `101010…`) that compressors miss. *Use in the toy DSL.*
4. **Lean/Coq proof terms** — in a dependently-typed proof assistant, the proof
   term is simultaneously the program, the verifier (type checker), and the
   description-length object (proof length). *The cleanest instantiation, confined
   to formal domains.*

---

## 10. Domain of validity: what "reality" means per domain

The framework's validity domain is bounded by the existence of **some observable,
sampled-from-reality outcome variable**. "Verifiability" is a property of the
*proposition form*, not the domain — an open-ended competence can usually be
restated as a prediction task with a real-world outcome.

| Domain | What the Anvil checks |
|---|---|
| Code | Execution against held-out unit tests |
| Math | Answer extraction / formal proof check (Lean, Isabelle) |
| Physics / games / DBs | Simulator or deterministic outcome |
| **Open-ended** (writing, dialogue, taste) | A *prediction program* — e.g. "a program that predicts which passage a human rates higher" — tested against held-out **human judgments** (exogenous samples) |

Two honest caveats for the open-ended regime:

1. **Goodhart drift.** Optimizing a proxy program (predicted engagement) can drift
   from true quality, and that drift re-enters the loop *through the choice of
   outcome variable*. This is the real, unsolved limit.
2. **Zero-outcome domains.** For a domain with *no* observable outcome (private,
   unrevealed taste), there is no exogenous ground — and no framework, including
   human practice, can self-improve on it reliably.

---

## 11. Honest limits

**Grounded** (with citation or theorem):
- The collapse-escape logic: a minimal program defines the full distribution.
- Execution as exogenous grounding (the α > 0 term).
- The MDL/gzip/CTM/BDM ranking ladder.
- The synthesis-vs-judge fault-tolerance asymmetry.
- The anti-circularity invariant (verifier ≠ LLM).

**Speculation** (mark these `[speculation]` if you build on them):
1. That CTM/BDM ranking transfers beyond small programs.
2. That the Crystal constitutes a *true* functional self-model rather than
   externalized memory — 2607.04277's claim is *not yet demonstrated* by this design.
3. That the loop actually compounds beyond one or two rounds — this is exactly what
   ablation D in the experiment is designed to settle.
4. That the Goodhart-drift limit in open-ended domains is tractable.

**Not claimed:** that this is currently *implemented* anywhere; that it is
*sufficient* for AGI; that it is *safe* without the safety review in the roadmap.
