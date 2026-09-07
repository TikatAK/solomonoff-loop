# FAQ

> The sharpest objections, answered. If a question here is wrong-headed, the
> answer says so directly.

---

### Q1. "You haven't escaped the evaluator bottleneck — you've relocated it into synthesis and specification."

Partially true, and that relocation is a *strict improvement*, not a dodge.

- At the specification layer, one correct spec certifies infinitely many programs,
  and specs are vastly more compressible and checkable than raw outputs. The trust
  surface shrinks from "all generated tokens" to "a small set of auditable specs."
  That is the best achievable bound — it is what humans do (we audit axioms and
  laws, not every inference).
- The anchor is **predictive error on held-out reality**, not self-consistency.
  Even a spec is a falsifiable prediction with a measurable, exogenous track
  record.
- The load-bearing asymmetry remains: a synthesis error is *detected and
  discarded*; an evaluator error is *silently absorbed*. Relocating the weakness
  from the evaluator (irrecoverable) to the synthesizer (self-lifting) is the
  whole point.

### Q2. "Who verifies the verifier?"

The **meta-loop**. Specifications are themselves falsifiable programs, recursively
grounded the same way. A spec survives only if the programs it certifies keep
predicting reality. There is no infinite tower of verifiers — there is a tower that
bottoms out in held-out samples of reality, which is the one unforgeable referee.

### Q3. "Isn't this just program synthesis / RLVR / distillation, dressed up?"

It *uses* all three, but the claim is about the **loop**, not any single component.

- **Program synthesis** is a step, not the loop; alone it has no notion of
  accumulating verified knowledge.
- **RLVR** (verifiable rewards) works but is confined to verifiable domains and
  does not retain a growing, legible self-model.
- **Distillation** loses tails unless paired with invocation (invariant 2).

The novel combination is: *a description-length-ranked, mechanically-verified,
executable program layer that serves simultaneously as grounding, self-model,
curriculum, and frozen control.* No single existing component is that.

### Q4. "How does this extend to writing, dialogue, taste — domains with no executor?"

"Verifiability" is a property of the *proposition form*, not the domain. An
open-ended competence can usually be restated as a **prediction task with a
real-world outcome variable**: "good prose" becomes "a program that predicts which
passage a human rates higher," tested against held-out *human judgments*.

Two honest caveats: (1) optimizing a proxy program can drift from true quality
(Goodhart), and that drift re-enters through the choice of outcome variable; (2) for
domains with *zero* observable outcome, no framework works — including human
practice. We state this boundary rather than paper over it.

### Q5. "Isn't algorithmic probability (CTM/BDM) uncomputable?"

Kolmogorov complexity `K(x)` is uncomputable in general, yes. CTM/BDM are
*computable approximations* for short strings, and they catch structure (e.g.
`101010…`) that compressors miss. The practical ladder is: **MDL / symbolic-
regression complexity** (what PySR already minimizes) → **gzip / NCD** (scalable
rough bound) → **CTM/BDM** (principled, small) → **Lean/Coq proof terms** (rigorous,
narrow). We use the rung that fits the scale; the toy DSL is exactly the regime
where CTM/BDM is computable.

### Q6. "Isn't this just 'harness engineering' (Lilian Weng / Darwin Gödel Machine)?"

No — and the difference is the load-bearing one. Harness engineering rewrites the
*scaffolding* around a **frozen model**; the model's weights never change and nothing
accumulates into the model. Solomonoff Loop rewrites a *growing program layer* that
is distilled back into the model (invariant 2's "distill" branch). It is the
"experience distillation" step that SPEE identifies as the missing middle — test-time
extraction without weight internalization is only half the loop.

### Q7. "What about safety?"

Real and treated as a gate, not an afterthought. The safety review in the roadmap
requires: the Anvil stays non-LLM and outside the loop it grades; the loop cannot
modify its own evaluation channel; append-only logs; human approval for anything
that touches the spec layer. The danger is not "improvement" — it is an agent
editing its own evaluator, which makes "improvement" and "scoreboard editing"
indistinguishable. The design specifically keeps the evaluator *out* of the loop's
write access.

### Q8. "How do I know reported improvement is real and not 'phantom gains'?"

The Crystal is a frozen control *by construction*. Every program is a checkable
assertion; regression = a program that used to pass now fails. The audit protocol
(experiment-design §7) additionally requires exact-match on held-out inputs,
transition-level reporting, bootstrap CIs, FDR control, and never reusing held-out
data. Improvement is auditable by a third party re-running the Anvil.

### Q9. "What is actually *new* here, honestly?"

Not the components. The **configuration** — and specifically three things no prior
system combines:

1. A *mechanical, non-LLM* verifier that makes the only irrecoverable failure mode
   (a weak evaluator) structurally impossible.
2. A *description-length-ranked* program layer that is simultaneously the
   anti-collapse grounding (2601.05280's escape), the introspectable self-model
   (2607.04277's missing primitive), and the curriculum.
3. A *recursive spec layer* (the meta-loop) that lets the evaluator itself
   co-evolve — the gap the 2025–26 frontier explicitly names as unsolved.

Whether that configuration actually compounds is precisely what the experiment is
designed to falsify.
