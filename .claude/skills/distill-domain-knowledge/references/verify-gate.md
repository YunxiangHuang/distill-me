# Reference — ③ Verify (the gate) + ④ Capture

Read this at steps ③/④. Verify is the gate that stops content drift; Capture is what's allowed through it.

## ③ Verify — the gate

Treat every mechanical / factual / how-it-works claim as **UNVERIFIED**. **Proactively go FIND and check it** against ground truth — don't wait for it to be handed to you; the dangerous errors look plausible. Fan out agents/Workflow for breadth.

- **"Why / intent / decision rationale" is the expert's authority** — mark it `⚠️ expert-asserted`, don't force external proof.
- **On any discrepancy, RE-GRILL the expert with the evidence** — never silently overwrite either side. Conflicts are often layering / version differences, and the resolution is itself knowledge.
- **Answer-mismatch** (asked about A, answered about B — a sibling path): the mismatch is a claim; verify the A↔B relationship immediately, in the same exchange (see `grilling.md`).
- **Recalled memory entries are claims too**: re-verify any entry whose mechanism or anchors this slice touches, and queue it for the ⑥ reconcile (see `memory-loop.md`).

### Ground truth isn't always in the repo

Some claims — **DB schema, runtime / production behavior, prod config, data distribution** — can't be settled by reading code. A repo-scoped agent confirms "the code says X", never "production does X". Tag those `✅ code-verified` vs *needs-prod-evidence*; don't conflate.

### Bug-claim reality check

If a verify pass (especially a focused sub-agent told to "confirm or refute this claim") concludes **"this is a production bug"** but that would contradict observable reality — e.g. "refunds are broken" while refunds demonstrably work in prod — **downgrade to provisional → punch-list; do not capture it as a confirmed bug.** A confident verdict from repo-only evidence over-claims. (Real case: a verify sub-agent reported a schema-mismatch as a `is_bug=true` runtime bug; since the feature works in prod, the column almost certainly exists via an out-of-repo migration — the right capture was a `needs-prod-evidence` punch-list item, not a confirmed bug.)

### Close Verify with a gap punch-list

Emit `pending/gaps.md` rows — **claim · why-unresolved · what's-needed · owner** — so unresolved / unverifiable items are an active worklist, not silent gaps.

## ④ Capture (verified only)

Write verified content into a markdown source of truth — small files, one concept each — tagging status:

- **`✅ verified @ path:line`** — a code/source anchor is REQUIRED for ✅ (evidence, not just an assertion; it also makes the doc AI-jumpable).
- **`⚠️ expert-asserted`** — intent / why, or a mechanism not yet code-confirmed.
- **`pending`** — → punch-list; stays OUT of the verified body until resolved.

Capture ends with **Memory loop *index*** (see `memory-loop.md`): (re)index every new/revised file into the store — on revision DELETE the stale entry first — and land verified architecture/topology byproducts as point-in-time-verified entries (they are resolved facts; do NOT park them on the punch-list).

## Mistakes

| Mistake | Reality |
|---|---|
| Only verify when a contradiction is obvious | Verify **proactively** — go find ground truth even when nothing looks wrong. |
| Accept a repo-scoped agent's "this is a production bug" as confirmed | Code-verified ≠ prod-verified. Some ground truth (schema, runtime, prod data) isn't in the repo. If the "bug" contradicts observable reality (the feature works in prod), downgrade to provisional → punch-list. |
| Silently pick code-over-expert (or vice versa) on a conflict | Re-grill with the evidence; the discrepancy's resolution is itself valuable knowledge. |
| Capture everything, reconcile "later" | Don't land unverified / ambiguous content. Gate capture on verify + glossary. |
| Park a VERIFIED byproduct fact on the punch-list (or drop it) | The punch-list holds UNRESOLVED items. A verified architecture/topology byproduct is resolved — it goes into the memory store as a point-in-time-verified entry with anchors. |
