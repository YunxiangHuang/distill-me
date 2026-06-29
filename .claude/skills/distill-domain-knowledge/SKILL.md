---
name: distill-domain-knowledge
description: Use when capturing a domain expert's tacit knowledge — domain rules, decision rationale, edge cases, system logic — into reusable, verified docs and triggerable skills. Use when someone wants to "write down what's in my head" so future AI sessions get the domain right, or when building a project glossary / ubiquitous language to remove term ambiguity.
---

# Distilling Domain Knowledge

Turn an expert's tacit knowledge into **ground-truth-verified** docs + a triggerable skill, with a living glossary that removes term ambiguity.

## Core principle — two failure modes, two pillars

- **Content drift** (memory ≠ reality) → **Verify before capture**: the expert's spoken "how-it-works" claims are DEFAULT-UNVERIFIED until checked against ground truth.
- **Reference drift** (one word → many meanings) → **Ubiquitous-language glossary**: every term gets ONE canonical definition; overloaded words get split/disambiguated.

A polished knowledge base full of unverified or ambiguous claims is worse than none — it confidently misleads every future session.

- **Content before container** (a *format*-layer failure mode): don't author a skill-format spec / directory scheme before extracting real content — the dominant failure is a polished empty shell (perfect format, nothing externalized). Markdown source-of-truth comes first; prove the protocol on ONE slice end-to-end (a walking skeleton) before scaling the format out. Value priority: **content > container**.

## When to use
- An expert wants to externalize domain knowledge / decision rationale / logic.
- Documenting a subsystem so future AI sessions implement/review it correctly.
- Term ambiguity keeps causing confusion → build/maintain the glossary.

## Method

Sub-tools: **grill-me** for the interview (step ②, **mandatory** — external dependency from `github.com/mattpocock/skills`, install it to run the Grill step by the book), **writing-skills** for the final skill, the **Workflow tool / parallel agents** for verification fan-out. Degrade gracefully only if a sub-tool genuinely doesn't exist.

```
① Scaffold → ② Grill → ③ Verify → ④ Capture(verified only) → ⑤ Glossary final-align → ⑥ Distill
   ├──────────── Glossary-first term admission：贯穿全程，新名词出现的瞬间执行 ────────────┤
   └──────── Memory loop（有记忆库则必走）：recall@① · re-verify@③ · index@④ · reconcile@⑥ ────────┘
```

**This SKILL.md is the spine.** Each step's detailed discipline lives in a reference file — open it when you reach that step (don't try to run the step from the one-liner alone; the traps are in the detail):

| Step / cross-cut | Read |
|---|---|
| ② Grill (the relentless interview + its traps) | `references/grilling.md` |
| ③ Verify + ④ Capture (the gate; code-vs-prod; bug-claim reality check) | `references/verify-gate.md` |
| Glossary-first admission (runs throughout) + ⑤ final-align | `references/glossary-first.md` |
| Memory loop (runs throughout, if a store exists) | `references/memory-loop.md` |
| ⑥ Distill — RED→GREEN skill validation | `references/red-green-validation.md` |

### Cross-cutting disciplines — apply at EVERY step, not only where numbered

A term surfaces mid-Verify; an unexercised enum is classified at Capture; the store is touched at four steps. These don't belong to one step — keep their CORE in hand throughout (the per-step references give depth, but you must apply these even when you never open that reference):

- **Glossary-first admission** — the moment ANY new term surfaces (the expert's words AND names *you* coin: files, skills, domain labels): duplicate-check → org-collision-check (**ask the expert**) → admit with the **headword in its original language** (a translated/coined umbrella silently merges distinct concepts — e.g. one label for both `refund` and `payout`). → `references/glossary-first.md`
- **Memory loop** (if a store exists) — recall@① · re-verify recalled entries@③ · index on capture (**delete the stale entry first**)@④ · reconcile@⑥. A slice that leaves stale entries behind is not done. → `references/memory-loop.md`
- **Never-produced artifact → don't self-label** — a state never written / flag never set / field nothing populates is forward-reserved *vs* backward-residual ambiguous; only the expert (or a `// DEPRECATED` comment / migration) decides — default `pending` → ask, don't pick "dead" or "reserved". A *narrowly*-produced one (admin-tool-only) is **live-but-narrow**, not this trap. → `references/grilling.md`
- **Honesty over fluency** — no recorded reason? capture exactly that; never backfill a plausible "why" (future sessions inherit it as fact). Applies to recommended answers (Grill) AND to what you write (Capture).
- **Verify before capture** — every mechanical / how-it-works claim is default-unverified, and code-verified ≠ prod-verified. → `references/verify-gate.md`
- **Sub-agent fan-out discipline** — when dispatching agents for code reconnaissance/judgement (Scaffold enumeration, Verify falsification, RED→GREEN), two non-negotiables: ① **model** — judgement-class fan-out runs on a production-grade model, NOT the lightweight default; a weak model fails by being dim, not by lacking info, so its coverage call / verdict is suspect. ② **scan depth** — each agent MUST read the actual code to judge coverage/mechanism; NEVER infer from directory layout or file names alone (a shallow dir-skim makes the coverage call unreliable). Require a `scannedSummary` back, proving it covered the assigned set rather than guessed from names.

### The six steps (essence — detail in the reference files)

1. **Scaffold (light, one pass).** Build a "current understanding" skeleton from existing ground truth — code, schemas, docs, data, dashboards, tickets/PRs, prior memory (with a store, this is Memory loop *recall*). Emit a **typed entity catalog** as the spine: classify each entity as **module / cross-cutting function / config domain**, one-line definition + **upstream→downstream**. Just enough to grill against; **do NOT deep-dive** — and a *thorough* Scaffold raises, not lowers, how much grilling is left (see `references/grilling.md`). Reserve parallel agent/Workflow fan-out for **Verify** (breadth-checking mechanical claims against ground truth), NOT for mass-re-reading code at Scaffold — the high-value work is the serial, human-in-loop grill.
2. **Grill — invoke `grill-me` (MANDATORY, don't hand-roll it).** Interview relentlessly, ONE branch at a time, each question carrying YOUR recommended answer, until each branch bottoms out. The traps (scaffold-induced laziness, manufacturing reasons, forward-reserved vs backward-residual) are in `references/grilling.md` — read it.
3. **Verify — the gate.** Every mechanical/factual/how-it-works claim is UNVERIFIED; proactively go find ground truth (fan out agents/Workflow). "Why/intent" is the expert's authority. On discrepancy, **re-grill** with the evidence. End with a `pending/gaps.md` punch-list. Code-vs-prod limits, the bug-claim reality check, and the answer-mismatch signal are in `references/verify-gate.md`.
4. **Capture (verified only).** Small markdown files, one concept each, tagged `✅ verified @ path:line` (anchor REQUIRED), `⚠️ expert-asserted`, or `pending` (→ punch-list, out of the verified body). Then Memory loop *index*. Detail in `references/verify-gate.md`.
5. **Glossary final-align.** ONE living glossary; every term → one canonical definition + scope-type, **headword in the term's original language**. Sweep for terms that slipped admission. Detail in `references/glossary-first.md`.
6. **Distill.** Generate a focused **triggerable skill** (and/or ADR/runbook) from the VERIFIED knowledge; test it **RED→GREEN** per `references/red-green-validation.md`. Close with Memory loop *reconcile* — a slice that leaves stale store entries behind is NOT done.

## Output shape (per domain slice)
- `knowledge/<area>/<concept>.md` — verified source of truth; every ✅ claim carries a `path:line` code anchor.
- `<glossary>.md` — the living ubiquitous language (each term: canonical definition + scope-type, original-language headword).
- `pending/gaps.md` — punch-list of unresolved / unverifiable items (claim · why · what's-needed · owner).
- `.claude/skills/<name>/SKILL.md` — distilled, RED→GREEN-verified skill (symlink to `~/.claude/skills/` to make it live everywhere).
- **Memory-store delta** (whenever a store exists) — every new/revised file (re)indexed with stale entries deleted first; every recalled/touched/superseded entry reconciled; verified byproducts landed as point-in-time-verified entries.
