# Reference — ② Grill (the relentless interview)

Read this when you reach step ②. The whole value of distillation lives here: Scaffold/Verify give you *mechanism*; only grilling gives you *intent*, and intent is where the dangerous, plausible-looking errors hide.

## Invoke `grill-me` — MANDATORY, do not hand-roll the interview

`grill-me` enforces the discipline that stops a grill from going thin: relentless, **ONE branch at a time**, each question carrying YOUR recommended answer for the expert to confirm/correct, walking the decision tree until each branch **bottoms out** — the expert hits *undecided / can't-recall / no-recorded-reason*, or the answer opens no new branch. Capture corrections and the *why*.

One-at-a-time is load-bearing: each answer spawns the next question, so **batching a list of questions fails** — you can't see where answer N leads before asking N+1, so you can't follow the tree.

## Trap 1 — a rich Scaffold induces grill laziness

Once the mechanism is mapped, "mechanism is clear" masquerades as "understanding is complete" and you ask a few questions and stop. **Mechanism ≠ intent.** The thicker the Scaffold, the *higher* the share of remaining value that is intent-only — so a strong Scaffold means grill **deeper**, not less. (Real case: a thorough 5-agent Scaffold led to a 4-question grill that missed that a rule which looked like deliberate policy was actually a removed historical limitation — the expert had to flag the thin grill.)

## Trap 2 — the recommended-answer habit manufactures reasons

Carrying a recommended answer is good, but it pressures you to invent a plausible *why* for every rule — and the truth is often **historical / residual / no recorded reason**. So always offer "or this is a leftover / there's no recorded reason" as a live candidate; when the expert confirms it, **capture exactly that**. A fluent wrong reason misleads worse than an honest "unknown", because future sessions build on it. (Real case: an AI rationalized three domain rules in a row and the expert corrected all three to *historical baggage / residual field / product policy with no recoverable reason*.)

## Trap 3 — forward-reserved vs backward-residual

An artifact that is **never produced** — a state never written, a flag never set, a field nothing populates — is ambiguous between *reserved for the future* and *leftover from the past*. Code structure alone cannot tell them apart.

- **Default: unknown → ask the expert and hold `pending`; do NOT self-assign a label.** A never-produced artifact is neither automatically "dead code" nor automatically "reserved"; guessing either is the mistake. forward/backward is the *expert's* classification after they answer, not yours to pick.
- **If the code itself documents the answer** — a `// DEPRECATED, migrated to X` comment, a matching migration, an author note — that IS the expert's answer; commit it with the anchor. The trap is the *undocumented* never-produced artifact.
- **Scope:** only **never-produced** artifacts. One that is narrowly produced (e.g. written only by an admin tool) is **live-but-narrow**, not this trap — don't apply forward/backward to it.

## Answer-mismatch signal (also a Verify trigger — see verify-gate.md)

When the expert answers about a *different* mechanism than the one you asked (a sibling method/path/service — e.g. you asked about `updateRecord`, they answered about `updateRecordIncremental`), the mismatch itself is a claim — expert mental models often merge sibling paths. Verify the relationship (does A route through / share code with B?) **immediately, in the same exchange**, and present the resolved topology back; deferring it costs an extra grill round.

## Mistakes

| Mistake | Reality |
|---|---|
| Rich Scaffold → ask a few questions, declare grill done | Scaffold answers *what/how*; the grill's job is *why/intent*, which lives in the branches each answer opens. Invoke `grill-me` and go until branches bottom out (undecided / unknowable / no-new-branch), not until you've asked a batch. |
| Backfill a plausible reason for every rule; self-assign "dead" or "reserved" to a never-produced artifact | Truth is often historical / residual / no-recorded-reason — capture "unknown" honestly. A never-produced state/flag is forward-reserved vs backward-residual ambiguous: only the expert (or a `// DEPRECATED`-style comment/migration) decides — default to unknown→ask→`pending`, don't pick a label. (A narrowly-produced state, e.g. admin-tool-only, is live-but-narrow, not this trap.) |
| Expert answers about mechanism B when you asked about A — and you move on | The mismatch IS a finding: verify the A↔B relationship right then, in the same exchange. |
