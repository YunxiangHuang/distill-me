# Reference — ⑥ Distill: RED→GREEN skill validation

Read this at step ⑥ when you turn verified knowledge into a triggerable skill. The point of RED→GREEN: confirm an agent **WITHOUT** the skill makes the mistake, and **WITH** it does not — i.e. the skill actually changes behavior, not just reads well. (Use `writing-skills` for how to write the skill itself.)

## Model floor

Run both the RED (no-skill) and GREEN (with-skill) agents on a model **at least as capable as the one that will use the skill in production, and never below Sonnet.**

- A **too-weak baseline** fails the trap because it's weak, not because it lacks the skill → false-positive validation. **Haiku is below the floor** — it mis-reads, or grabs vocabulary from the skill rather than applying judgment; don't mistake its incompetence for a real finding. (Real case: a Haiku RED→GREEN "found" that a skill edit misfired, but on Sonnet the same edit behaved correctly — the Haiku result was noise.)
- A **strong adversarial baseline** is doubly useful: it either confirms the flip harder, or — being capable — *exposes the skill's coverage gaps* by surfacing real alternatives the skill missed.

## Pair every case with its opposite

Each **positive** case (*does the skill fire?*) gets a **should-not** twin (*does it OVER-fire?*). A good edit fires on the positive and stays quiet on the should-not. If the twin regresses — hedges a clear-cut case, downgrades a real bug, forces a translation — the wording is too aggressive; soften it. **Paired positive + should-not cases are a standard part of RED→GREEN, not an extra.**

## Ground cases in REAL observed failures

When you have a real incident, **replay it** — don't substitute a clean synthetic proxy. A synthetic eval that does NOT discriminate while you hold a ground-truth failure **indicts the eval, not the guard.** (Real case: a glossary rule looked "non-discriminating" on a synthetic eval and was nearly deleted — but the rule existed to stop a coined umbrella that had already merged two concepts in a real session; an eval that *replayed that exact naming task* immediately discriminated.)

## Keep RED clean (no skill leakage)

- Feed the GREEN agent the skill content **inline** in its prompt — don't rely on installation.
- Run RED while the skill is **invisible** to it. Installing the skill anywhere the session can see — a global symlink **or** the project's `.claude/skills/` — registers it and can leak into RED. Uninstall before re-running RED if it's already installed. (Headless `claude -p` from a cwd outside the project isolates the project's CLAUDE.md leak but NOT a globally-installed skill.)
- **Contamination check:** grep the RED output for tokens unique to this slice (gaps numbers, PR numbers, skill-original phrasing). Any hit = leakage; the RED baseline is void.
- **Auto-capture hooks are a leak vector too:** if a persistent-memory plugin captures via hooks, it may write the slice's just-formed conclusions into the store *during* the run, which then leak into a later RED. Hold the slice's store writes until **after** the clean RED completes.

## Sequence (avoid the leak)

Write SKILL.md → run RED→GREEN (GREEN gets content inline) → **only then** symlink + write the CLAUDE.md progress line. The progress line and the installed skill both leak slice conclusions into a later RED baseline.

Close the slice with **Memory loop *reconcile*** (see `memory-loop.md`) — a slice that leaves stale store entries behind is NOT done.

## Pick a scenario the code can't reveal

In a highly **self-documenting** domain, a capable model reads the mechanism correctly from raw source with NO skill — so a *mechanism-class* RED→GREEN scenario does not discriminate (RED self-corrects; verdict is a tie). To prove the skill's real increment, pick a **business-intent / rationale** scenario the code cannot reveal (who configures this and why; a deliberate-laziness characterization; an "is this reserved or residual" call) — RED must be unable to answer or must guess wrong. Scenario *class* decides whether RED→GREEN can discriminate at all — distinct from the model floor, which is about baseline *strength*.

## Experiment design

- **Isolate one variable.** Don't handicap RED with an artificial constraint to manufacture a gap — a "no source access" limit is bypassable (a capable agent just shells out), so it isolates nothing. Give both arms **equal** access and still show the increment, i.e. prove it under conditions *least* favorable to the skill.
- **Keep the seed / triage prompt at symptom level** (method names, signals — no root-cause semantics), or the prompt itself leaks the answer into RED.

## Consumption-verification (multi-file skills)

Distinct from RED→GREEN (which tests *content* discrimination), this targets the **split form**. After splitting a skill into spine + `references/`, give an agent the spine + references and run a **cross-step task**, comparing against a fully-inlined MONOLITH baseline (model floor ≥ Sonnet). If the split agent misses a cross-stage rule the monolith honors, that rule got buried in a single-step reference and never read — **lift the cross-stage CORE back into the always-loaded spine.**

## Mistakes

| Mistake | Reality |
|---|---|
| Stop at a doc | A doc is not triggerable. Distill to a skill so it actually changes future agent behavior; verify RED→GREEN. |
| Validate on a weak model (Haiku) | Below the floor — failures are incompetence, not missing-skill. Use ≥ Sonnet / production tier. |
| One-sided eval (positive only) | Pair each positive with a should-not twin; an edit that over-fires (hedges clear cases) is a regression. |
| Treat a non-discriminating synthetic eval as proof the guard is useless | If you hold a real ground-truth failure, replay it — the synthetic eval tested the wrong failure mode. |
| Symlink the skill / write the progress line before RED→GREEN | Both leak slice conclusions into the RED baseline. Test first, install after. |
