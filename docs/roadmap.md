# Roadmap

> The phases below are ordered by *dependence*, not by desirability. Each phase
> has an explicit exit criterion; a phase is not "done" until its criterion is
> met or the framework is falsified.

---

## Phase 0 — Formalization (now)

- [ ] State the collapse theorem and the Crystal's escape as a precise claim with
      explicit assumptions.
- [ ] Define the DSL formally (grammar, semantics, complexity measure).
- [ ] Nail down the "exchange rate" question (minimum α) as a measurable quantity.

**Exit criterion:** a reviewer can check every grounded claim against a citation or
a theorem, and every speculative claim is marked.

---

## Phase 1 — The minimal experiment (1–2 weeks)

Run [`experiment-design.md`](experiment-design.md) exactly as specified.

- [ ] Implement the toy DSL and its interpreter (the Anvil).
- [ ] Generate P* and the k/m/tail splits.
- [ ] Implement the loop (propose → verify → rank → crystallize → ground).
- [ ] Run all four ablations (A–D).

**Exit criterion:** the decisive ablation-D result is known and reported with
bootstrap CIs. A null result here ends the project honestly.

---

## Phase 2 — Introspection

If Phase 1 survives:

- [ ] Let the Proposer read the Crystal and propose merges/refactors/patches.
- [ ] Accept changes only under the mechanical verification gate.
- [ ] Measure whether introspection reduces description length *without* reducing
      tail accuracy.

**Exit criterion:** a patched Crystal that verifies *and* is shorter than the
unpatched one, with no tail-accuracy loss.

---

## Phase 3 — The meta-loop (verifier evolution)

If Phase 2 survives:

- [ ] Treat specifications as first-class programs in a richer DSL.
- [ ] Weight specs by the predictive success of the programs they admit.
- [ ] Let specs decay/prune when they admit overfit programs.

**Exit criterion:** the evaluator (the spec layer) demonstrably improves over
rounds — the gap the 2025–26 frontier (CORAL, Aspire, DGM) names as unsolved.

---

## Phase 4 — Open-domain extension

If Phase 3 survives:

- [ ] Restate open-ended competences (writing, dialogue, taste) as prediction
      programs with real-world outcome variables.
- [ ] Ground them against held-out *human* judgments (exogenous samples).
- [ ] Characterize Goodhart drift explicitly, and bound it.

**Exit criterion:** a demonstration in at least one open-ended domain where the
Crystal improves a real, held-out, human-measured outcome — *and* the drift risk is
measured, not assumed away.

---

## Cross-cutting: safety review (not a phase, a gate)

Before any phase that lets the loop *edit its own verifier or its own training
harness*, a safety review must pass:

- The **Anvil remains non-LLM** and sits outside the loop it grades.
- The loop cannot modify its own evaluation channel (tests, metric code, held-out
  sets).
- Append-only logs of every accepted change.
- A human-approval gate for any change that touches permissions or the spec layer.

This is borrowed from the harness-engineering literature's "evaluator-integrity"
lesson: the moment an agent can rewrite its own evaluator, "improvement" and
"scoreboard editing" become indistinguishable.

---

## Open questions (first-class research targets)

1. **The exchange rate.** What is the minimum exogenous-grounding fraction α below
   which a loop degrades? Unmeasured anywhere.
2. **Synthesis as the true ceiling.** How much of the loop's capability is bounded
   by program-synthesis quality, and how fast is that bootstrappable?
3. **CTM/BDM at scale.** Does algorithmic-probability ranking transfer beyond toy
   DSLs, or is gzip the practical ceiling?
4. **True introspection vs. externalized memory.** Does the Crystal constitute a
   functional self-model (2607.04277's claim), or merely legible storage?
5. **Goodhart drift in open domains.** Can the drift induced by optimizing a proxy
   outcome-variable program be bounded, or is it a hard wall?
