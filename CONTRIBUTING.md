# Contributing to Solomonoff Loop

Solomonoff Loop is a **research framework** first and a code project second. The
most valuable contributions right now are not lines of code — they are
**falsification attempts, reproductions, and sharper formulations** of the ideas
in this repository.

## What we are looking for

- **Reproduce or falsify the minimal experiment.** The single most valuable
  contribution is running the experiment in
  [`docs/experiment-design.md`](docs/experiment-design.md) and reporting
  whether the Crystal beats ablation D ("verify but do not retain"). A null
  result is *more* valuable than a positive one — it saves everyone from
  building on sand.
- **Sharpen the formalism.** Point out where a claim is under-specified,
  circular, or relies on an assumption that is not stated.
- **Add references.** The 2026 literature is moving fast and partially
  unverified. If you can confirm, correct, or replace a citation, do so.
- **Propose or implement a DSL.** A concrete, minimal domain-specific language
  for the program-induction experiment is the highest-value code contribution.

## Ground rules

1. **Flag speculation.** This repository distinguishes *grounded* claims (with a
   citation or a theorem) from *speculative* claims. Keep that discipline in
   every edit.
2. **Cite before asserting.** If you add a claim about what "the literature
   shows," link the paper. A claim without a reference is a proposal, not a fact.
3. **Prefer falsification over advocacy.** The framework is a hypothesis. Treat
   it as such.
4. **Keep the mechanical verifier mechanical.** Any proposal that turns the
   Anvil into an LLM judge is a *different framework* and should be flagged as
   such.
5. **Bilingual parity.** Substantive changes to an English document should be
   mirrored in its `*.zh-CN.md` counterpart (and vice versa). If you cannot
   translate, add a note in the untranslated file.

## Pull request checklist

- [ ] Grounded claims are cited; speculative claims are marked `[speculation]`.
- [ ] The change respects the five invariants in
      [`docs/core-mechanism.md`](docs/core-mechanism.md) (or explicitly argues
      why one should change).
- [ ] English and Chinese documents are updated together.
- [ ] No benchmark numbers are reported without a frozen control and an exact
      reproduction recipe (see the audit protocol in
      [`docs/experiment-design.md`](docs/experiment-design.md)).

## Communication

Open an issue for discussion before opening a large PR. A short "here is what I
think the loop's weakest link is" issue is worth more than a thousand lines of
unrequested code.
