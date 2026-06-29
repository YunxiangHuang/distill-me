---
name: find-domain-knowledge
description: Discover and load distilled domain knowledge before reasoning about a business domain. Use when a sub-agent — especially one dispatched inside an orchestrated workflow (e.g. a multi-agent investigation / RCA swarm), where the proactive skill-invocation discipline is skipped — is about to investigate, conclude about, or reason over a business domain (orders, payments, billing, lifecycle status, refunds, discounts, deposits, or any domain a project has distilled). Finds the matching `recall-*-knowledge` entry-point skill for whatever domain the current task touches and loads it, so you build on verified distilled mechanics instead of re-deriving them from raw source. Invoke this BEFORE concluding, not after.
---

# Find Domain Knowledge

You were invoked because the current task may touch a business domain that has **distilled
knowledge** available — knowledge a previous session verified and wrote down so you don't have to
re-derive it from raw source. Find it and load it, then continue your original task.

This matters most when you are a sub-agent inside a workflow: the normal "scan skills and invoke
what's relevant" reflex is suppressed for dispatched sub-agents, so distilled knowledge that would
change your conclusion sits unused unless you deliberately go get it. That is what this skill does.

## Procedure

1. Identify the domain entities / keywords in your current task (the system, the ids, the
   operation — e.g. "this computed amount looks wrong", "this record won't change state",
   "this job is stuck", "this value diverged after an event").
2. Scan the available skills list for entries whose **name matches `recall-*-knowledge`**. Each
   one is the recall entry point for a distinct knowledge base.
3. Read those entries' descriptions and pick the one(s) whose described domain matches your task's
   entities. More than one may apply.
4. Invoke the matching `recall-*-knowledge` skill(s) via the **Skill tool**, then follow what they
   tell you to load (they route you to the specific domain skills and memory for their KB).
5. If **no** `recall-*-knowledge` skill matches your task, say so explicitly and continue — do not
   fabricate domain knowledge, and do not force an unrelated KB.

## Why it's built this way

- This skill holds **zero domain detail** by design. All routing lives in the per-knowledge-base
  `recall-*-knowledge` skills, which evolve with their knowledge base. Adding a new knowledge base
  requires no change here — it just appears as another `recall-*-knowledge` entry for step 2 to
  find. That keeps this discovery layer from rotting.
- Loading a recall skill costs a round-trip and some context. Invoke this only when your task
  genuinely touches a domain you'd otherwise reason about from scratch — not for pure telemetry
  lookups or mechanical search.
- **No registry / manifest by design.** Discovery is by naming convention (`recall-*-knowledge`), deliberately NOT a manifest file — a registry would reintroduce the manual-sync rot point this layer exists to avoid.
- **Wiring into an orchestrated workflow: inject the recall trigger ONLY in the deep reasoning phases, never as a global broadcast.** Sub-agent contexts aren't shared, so a global injection makes every one of dozens of agents reload domain knowledge (the recall increment is ~2-4× resources, × every agent = cost blowup), and most phases (triage / scout / reconcile) never touch a domain. Spend it where conclusion-correctness matters most and agent count is smallest. (Orchestrator-pushed domain facts and per-agent pull-recall are orthogonal and can stack.)
- **Match recall isolation to the consuming scenario.** Filtering recall by a project / namespace key is right for *in-distillation* recall (only this KB matters; auto-captured hook traffic is noise) — but **wrong for cross-project root-cause investigation**, where the cause may live in another project's observations and a hard filter hides the evidence. There, lean on semantic ranking to suppress noise instead of a hard project filter.

## Building a `recall-*-knowledge` (L1) entry

If you distill a new knowledge base, give it its own L1 recall entry — and keep it a **thin router**:

- It copies **no** domain-skill facts (the domain skills stay the single source of truth, so the router can't rot). Its value-add is the **cross-skill connections** no single skill's description can give — e.g. "this symptom needs domain-skill A *and* B loaded together."
- It MAY hardcode its **own** KB coordinates (its domain-skill names, its memory namespace) — it *is* that KB's recall entry and evolves with it.
- **Cross-repo reachability:** an L1 router is often invoked from a *foreign* repo that can't see the KB's raw `knowledge/*.md`. Depend only on **globally-installed domain skills** (Skill tool) + a **portable memory MCP tool** — never repo-relative knowledge-file paths, never a hardcoded `host:port` REST endpoint (ports / hosts drift per machine; an MCP tool is session-registered and portable).
