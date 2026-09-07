# Solomonoff Loop

> **A framework for giving large language models *verifiable, accumulating, auditable* self-improvement — by crystallizing their latent knowledge into executable programs and using those programs as their own ground truth.**

[中文文档](README.zh-CN.md) · [Core Mechanism](docs/core-mechanism.md) · [Experiment Design](docs/experiment-design.md) · [Roadmap](docs/roadmap.md) · [References](docs/references.md) · [FAQ](docs/faq.md)

---

## The problem in one paragraph

Every attempt to make an LLM improve *itself* fails the same way: the loop is only
as strong as the signal it climbs, and that signal is usually an LLM in disguise.
Self-rewarding models over-reward their own confident mistakes (**the self-confirming
loop**). Self-training on one's own output loses the tails of the distribution until
the model collapses into repetitive plausibility (**model collapse**, provable as
entropy decay). Curriculum proposers converge to the narrow band of problems that
satisfy the reward (**diversity collapse**). And because no one runs a frozen
control, much of the reported "self-improvement" is measurement artifact rather
than real capability gain.

The root cause is always the same: **the evaluator is either learned, or
self-referential, or absent.** Solomonoff Loop is a proposal to fix that root
cause — not to tune the optimizer.

## The idea in one paragraph

Let the model repeatedly **compress** its own latent knowledge into an explicit,
executable, legible **program layer** (we call it the **Crystal**). Every program
in the Crystal is verified by a **mechanical verifier** (we call it the **Anvil**)
— an executor, a proof checker, or held-out data — *never* an LLM. Once verified,
the Crystal becomes four things at once: **(a)** exogenous grounding that stops
collapse, **(b)** a legible self-model the model can introspect, **(c)** a
non-collapsing curriculum, and **(d)** a built-in frozen control that makes
improvement *auditable* rather than merely claimed.

The name comes from **Solomonoff induction**: the ranking signal is not the model's
own likelihood (which is circular), but **description length** — "find the shortest
program that explains the data." A minimal program defines the *whole* distribution,
including the tails that finite sampling loses. That single substitution is what
converts a self-referential loop into a grounded one.

```
        ┌──────────────────────────────────────────────────────┐
        │                                                      │
        ▼                                                      │
  ┌──────────┐  1. propose   ┌──────────────┐  2. verify   ┌─────────┐
  │ Proposer  │ ───────────► │ candidate     │ ───────────► │  Anvil   │
  │  (LLM)    │              │  programs     │              │ mechanical│
  └──────────┘              └──────────────┘              └────┬────┘
        ▲                                                      │ pass only
        │                                                      ▼
  ┌──────────┐  5. ground /    ┌──────────────────────────────┐
  │ distill   │ ◄───────────── │  Crystal (verified program    │
  │ to weights│   introspect   │  library, grows over time)    │
  └──────────┘                └───────────────┬──────────────┘
                                              │ 3. rank by description length
                                              ▼
                                       ┌──────────────┐
                                       │   Guide (LLM) │  anti-triviality +
                                       └──────────────┘  spec-grounding audit
```

## Why this is different

| Existing approach | How it climbs | Why it stalls |
|---|---|---|
| Self-rewarding (Yuan et al. 2024) | LLM judges its own output | Judge is the same weights as policy → self-confirming loop |
| Self-play (SPIN, DNO, R-Zero) | Compete against a past self | No adversary in language; curriculum narrows |
| RLVR (DeepSeek-R1) | Mechanical check (code runs, answer matches) | Works, but confined to verifiable domains |
| Harness engineering (Darwin Gödel Machine, Lilian Weng) | Rewrite the scaffolding, keep weights frozen | The model never changes; no accumulation |
| **Solomonoff Loop** | **Mechanical check on a *growing, compressed, legible program layer*** | **—** (this is the proposal) |

The load-bearing claim is a single asymmetry:

> **Synthesis errors are *detected and discarded* (a program fails to verify →
> throw it away); evaluator errors are *silently absorbed* (a bad reward → get
> reinforced). A weak synthesizer only slows the loop. A weak evaluator corrupts
> it. Therefore the only irrecoverable failure mode is the evaluator — and this
> framework makes the evaluator mechanical.**

That is why the design concentrates *all* its correctness on the Anvil being
non-LLM, and treats the Proposer's weakness as merely a matter of iteration count.

## The five invariants

These are architecture **laws**, not preferences. Breaking any one of them changes
the framework.

1. **The Anvil is mechanical, never an LLM.** Executor, proof checker, or
   held-out-data match. Any place an LLM acts as judge re-opens the self-confirming
   loop.
2. **The Crystal is dual-use: distill for fluency, invoke for tails.** Distilling a
   program into weights loses the distribution tails; keep the Crystal callable at
   inference so the tails survive.
3. **The Crystal is a *resampler*, not a memory.** Each verified program is a
   generator — it can emit an infinite stream of grounded I/O on demand. This, not
   distillation, is what defeats finite-sample entropy decay.
4. **The Guide has two jobs:** (i) reject trivially-verified but useless programs;
   (ii) audit *specification grounding* — "is this test derived from reality, or
   from the model's prior?"
5. **Selection is by description length (MDL), not model likelihood.** `L =
   L(data|program) + L(program)`. This is the non-circular ranking signal.

A **meta-loop** sits on top: the specifications/tests are themselves falsifiable
programs, recursively grounded the same way. A spec survives only if the programs
it certifies keep predicting reality. This is what closes the "who verifies the
verifier" regress.

## What this repository is

- **A research framework and falsifiable hypothesis**, not a production system.
  There is no shipped model yet.
- A precise specification of a **minimal viable experiment** (in
  [`docs/experiment-design.md`](docs/experiment-design.md)) whose job is to *falsify*
  the framework, not to celebrate it.
- A catalog of the **failure modes** and **theoretical grounding** (in
  [`docs/core-mechanism.md`](docs/core-mechanism.md)) so newcomers can see exactly
  which collapse the design is meant to survive and why.

## Status

| | |
|---|---|
| Stage | Pre-experiment (design + theory complete) |
| Next milestone | Run the minimal experiment; report the ablation-D result |
| License | MIT |
| Maturity | Speculative research. See the honest limits in [`docs/core-mechanism.md`](docs/core-mechanism.md#honest-limits) |

## Quick navigation

- **[Core Mechanism](docs/core-mechanism.md)** — the three failure modes, the
  theoretical escape, the formal loop, the five invariants, the meta-loop, and the
  honest limits.
- **[Experiment Design](docs/experiment-design.md)** — the falsifiable MVP, the toy
  DSL, the four ablations, and the audit protocol (the antidote to "phantom gains").
- **[Roadmap](docs/roadmap.md)** — phases from MVP to open-domain extension, plus
  open questions.
- **[FAQ](docs/faq.md)** — the sharpest objections and the answers.
- **[References](docs/references.md)** — the full citation list.

## Citation

If you build on this, please cite:

```
Solomonoff Loop: A Framework for Verifiable, Accumulating, Auditable
Self-Improvement of Large Language Models.
https://github.com/TikatAK/solomonoff-loop
```

## License

MIT — see [LICENSE](LICENSE).
