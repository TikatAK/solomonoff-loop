# Experiment Design

> The framework is a hypothesis. This document is the experiment that would
> **falsify** it. Its job is not to produce a flattering number — it is to find out
> whether the Crystal does any real work beyond "verify and discard."

---

## 1. The falsification criterion

**Null hypothesis (H₀):** the Crystal adds nothing. A loop that verifies programs
and then *throws them away* ("verify, don't retain") is as good as one that
retains them.

**The framework survives only if** the full loop (Crystal retained) shows:

1. **Monotonic tail-accuracy improvement** over rounds (measured on rare/adversarial
   inputs), *and*
2. **Healthy entropy** (no collapse) at every round, *and*
3. **Shrinking total description length** of the Crystal, *and*
4. **Beats ablation D** (verify-and-discard) by a margin outside bootstrap CIs.

If the full loop only *matches* ablation D, the framework is relocating collapse,
not escaping it — and the honest conclusion is that it does not work.

---

## 2. The toy DSL

Programs are small enough to synthesize reliably, yet rich enough to test the
"compression preserves tails" claim. The DSL:

```
value   ::= int | bool | list<int> | str
expr    ::= literal
          | var
          | expr + expr | expr - expr | expr * expr | expr // expr | expr % expr
          | expr == expr | expr != expr | expr < expr | expr <= expr | ...
          | and(expr, expr) | or(expr, expr) | not(expr)
          | if expr then expr else expr
          | loop n over acc  (bounded iteration, n ≤ 20)
          | recurse (bounded recursion, depth ≤ 5)
          | append(expr, expr) | index(expr, expr) | length(expr)
          | concat(expr, expr) | substr(expr, expr, expr)
          | map(func, expr) | filter(func, expr)
```

- **~15–20 primitives**, programs ≤ **~30 AST nodes**.
- The DSL is deliberately small: it is the regime where **CTM/BDM** (algorithmic
  probability over short strings) is still computable, and where the difference
  between "description length" and "perplexity" is most visible.

---

## 3. Ground-truth reality (P*)

- Sample **N = 200 hidden programs** from the DSL, uniformly. These are **P*** — the
  "reality" the loop must discover. The model *never sees P\* itself*, only samples.
- For each `P*_i`, generate:
  - **k = 5–20 training pairs** `(x_j, y_j = P*_i(x_j))` — visible to the Proposer.
  - **m = 50 held-out validation pairs** — *never* shown to the model during
    synthesis; used only by the Anvil.
  - **tail pairs**: a held-out set of *rare/adversarial* inputs (extreme values,
    empty lists, edge cases) reserved exclusively for measuring tail accuracy.

The split is the whole point: matching the k training pairs is *memorization*;
matching the m held-out pairs is *generalization*, and it is the only place
exogenous signal enters.

---

## 4. The loop, concretely

```
for round t in 0..T:
  for each target i with an unsolved-or-regressable state:
    1. PROPOSE:   Proposer sees (x_j, y_j) for j=1..k, samples B candidate
                  programs from the DSL.
    2. VERIFY:    Anvil executes each candidate on the m held-out inputs.
                  Keep only exact matches on ALL m.  (Mechanical.)
    3. RANK:      among survivors, keep the one with minimal description length
                  L = gzip_len(source) + λ·(AST node count).  (λ is a small
                  complexity penalty; MDL is the primary stand-in.)
    4. CRYSTALLIZE: add the shortest survivor to the Crystal.
    5. GROUND:
       (i)  distill — fine-tune the Proposer on I/O resampled from the Crystal,
            explicitly including tail inputs;
       (ii) invoke — keep the Crystal callable as tools at inference.
    6. INTROSPECT (phase 2 only): Proposer may merge/refactor/patch Crystal
       programs; a change is accepted iff it still verifies on all m held-out.
```

Implementation notes:

- The **Anvil** is a plain interpreter over the DSL — no neural component.
- **Proposer** is a base LLM (e.g. a 3B–8B model) fine-tuned on program-induction
  traces; it does not share weights with any judge.
- The **Guide** is a prompt-level filter (or a separate small LLM) that drops
  candidates that verify trivially (e.g. a constant program that matches m by luck)
  and flags specs that look model-authored rather than reality-derived.

---

## 5. Metrics (all reported with bootstrap 95% CIs)

| Metric | Definition | What it tests |
|---|---|---|
| **Tail accuracy** | accuracy on the reserved rare/adversarial inputs | whether compression preserves tails |
| **Entropy / coverage** | entropy of the model's output distribution (or candidate diversity) | whether the loop collapses |
| **Crystal val accuracy** | held-out accuracy of retained programs | whether the Crystal actually generalizes |
| **Description length** | total `L` of the Crystal | whether "improvement = shorter programs" |
| **Transition matrix** | per-target fail→pass / pass→fail counts between rounds | the audit protocol (below) |

---

## 6. Ablations (this is what proves the point)

| Ablation | What changes | Expected result if the framework is right |
|---|---|---|
| **A. α = 0 self-training** | No Anvil; the model trains only on its own outputs | Collapse / entropy decay (reproduces 2601.05280) |
| **B. LLM-as-judge** | Replace the Anvil with a second LLM's score | Self-confirming loop: over-rewards confident mistakes |
| **C. Perplexity ranking** | Rank candidates by the model's own likelihood instead of description length | Worse tail recovery; preference for plausible-but-wrong programs |
| **D. No Crystal** | Verify programs but do *not* retain/distill them | **The key comparison.** No compounding improvement. |

**The decisive test is D.** If the full loop does not beat D, the Crystal is dead
weight and the framework should be abandoned.

---

## 7. The audit protocol (the antidote to "phantom gains")

Recent work ("phantom gains" audits) shows that reported self-improvement is often
measurement artifact: single greedy decoding, mean accuracy, and the absence of a
frozen control all inflate apparent gains. The protocol here is designed to be
immune:

1. **Frozen control by construction.** The Crystal *is* a frozen control — every
   retained program is a checkable assertion. Regression = a program that used to
   pass now fails.
2. **Exact-match on held-out inputs only.** No judge, no soft scoring, no
   LLM-as-grader. A program passes or it doesn't.
3. **Transition-level reporting.** Report *which targets* went fail→pass and
   pass→fail, not just an aggregate. Aggregate accuracy hides "improved here,
   regressed there."
4. **Bootstrap CIs + FDR control.** All comparisons reported with bootstrap 95% CIs
   and false-discovery-rate control across targets.
5. **Never reuse held-out data.** The m validation pairs are burned once; any
   tuning that peeks at them invalidates the round.

The success criterion is therefore *auditable*: a third party can re-run the Anvil
on the Crystal and confirm the reported accuracy without trusting the authors.

---

## 8. Success criteria, in order

1. Tail accuracy improves monotonically over rounds (within CIs).
2. Entropy does not decay at any round.
3. Crystal description length shrinks.
4. Full loop beats ablation D outside bootstrap CIs.
5. (Phase 2) An introspected/patched Crystal still verifies and improves.

Failures 1–4 = framework falsified. A *null* result on D is the single most
valuable outcome: it saves the field from building on sand.

---

## 9. Scale and effort

| | |
|---|---|
| DSL | ~15–20 primitives, ≤ ~30 AST nodes |
| Ground truth | N = 200 hidden programs |
| Models | 3B–8B base LLM (Proposer); optional small LLM (Guide) |
| Compute | single GPU, a few days |
| Effort | **1–2 weeks** for a capable researcher |

Every component — program synthesis, DSL interpretation, gzip/BDM ranking,
distillation — exists today. Nothing here requires a research breakthrough; the
experiment tests whether the *composition* is a breakthrough.
