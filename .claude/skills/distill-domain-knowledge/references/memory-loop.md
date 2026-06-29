# Reference — Memory loop (throughout, if a persistent store exists)

Read this whenever the project has a persistent memory store (e.g. agentmemory). **MANDATORY** when one exists; skip only if no store exists.

The store is a slice ARTIFACT this protocol maintains — these four touchpoints are **steps of the protocol itself**, NOT external project conventions you may skip:

- **(a) recall @ ①** — open Scaffold with a recall pass; recalled entries are scaffold input, and a recalled ✅ is *point-in-time*, never current truth.
- **(b) re-verify @ ③** — every recalled entry whose mechanism or anchors the slice touches is one more claim for the Verify gate (recall-is-reverify); a drifted entry is a discrepancy like any other AND gets queued for (d).
- **(c) index @ ④** — Capture is not done until each new **or revised** knowledge file is (re)indexed into the store; revision = **DELETE the stale entry FIRST, then write** — old snapshots outrank and outcompete fresh knowledge at recall. Verified architecture/topology byproducts that don't merit a knowledge file land in the store as point-in-time-verified entries with code anchors — they are RESOLVED facts, so the punch-list (home of UNRESOLVED items) is the wrong place for them. **Embed the re-verify reminder inside the entry text itself** (e.g. a `verified-@<date> · re-verify before use` header): recall often happens in a foreign session that does NOT load this protocol, so the recall-is-reverify discipline can't be relied on externally — the entry has to carry it.
- **(d) reconcile @ ⑥** — the slice is not done until every store entry it recalled, touched, or superseded is **re-dated, upgraded to knowledge, or retired**.

*Why*: an unmaintained store doesn't just rot — it actively *outcompetes* fresh knowledge. Real case: a slice revised two knowledge files but skipped re-indexing; the store's stale snapshots kept ranking #1 on every recall for those topics, propagating an assertion the slice itself had just falsified. (And a *fabricated* entry — an entity name that never existed in code — must be **retired**, not just re-dated, or it keeps misleading every recall.)

## Output: the memory-store delta (per slice)

Every new/revised knowledge file (re)indexed with stale entries deleted first; every recalled/touched/superseded entry reconciled (re-dated / upgraded / retired); verified byproducts landed as point-in-time-verified entries.

## Mistakes

| Mistake | Reality |
|---|---|
| "Memory-store hygiene is a project convention, not a step of this skill" — declare done with stale entries left | The Memory loop IS protocol: recall @ ①, re-verify @ ③, index @ ④, reconcile @ ⑥. A slice closed over stale entries ships an assertion it just falsified at recall rank #1. |
| Trust a recalled ✅ entry as current truth | ✅ is point-in-time; assertions drift with refactors. Re-verify on use, then update the entry at reconcile. |
| Revise a knowledge file but write the new store entry without deleting the old | Old snapshots outrank fresh knowledge at recall. Delete the stale entry FIRST, then write. Fabricated entries get retired, not re-dated. |
