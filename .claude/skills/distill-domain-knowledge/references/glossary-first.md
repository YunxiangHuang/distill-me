# Reference — Glossary-first term admission (throughout) + ⑤ final-align

Read this the moment ANY new term surfaces — at every step, not just ⑤.

## Glossary-first term admission

"New term" includes words the **EXPERT** uses *and* names **YOU** coin (file names, doc titles, section headers, skill names, domain labels all count). Before the term propagates into any artifact, run admission:

- **(a) duplicate?** — a canonical term for this concept may already exist; reuse it.
- **(b) collision / ambiguity?** — does the word already mean something else in the org? ASK the expert — you cannot know org-internal vocabulary from code alone.
- **(c) admit** — concrete definition + scope-type into the glossary, with the **headword in the term's original language** (the code/domain spelling like `OrderRecord` / `payout` / `CancelTransaction`), **NOT translated into the conversation's working language**. Glossary terms are proper nouns, and a translation silently collapses distinct terms into one, re-introducing the ambiguity the glossary exists to kill. Translations live in the gloss/description, never as the headword.

Step ⑤ is the **closing audit**, NOT the first check.

*Why front-load admission*: a term that spreads into file names / skill names / cross-references before admission multiplies rename cost many-fold. Real case: an AI-coined "billing engine" collided with an existing product-area term already in use — caught late, it cost 3 file renames + a skill rename + a repo-wide reference rewrite; caught at admission, it would have been one question.

*Why original-language headwords*: a coined working-language umbrella can merge two distinct code concepts. Real case: a coined Chinese umbrella **"出款"** meant *both* `refund` (money to the customer) and `payout` (money to the merchant) — distinct receivers, triggers, and accounting — and confused even the domain expert. Keeping the code-native English headwords (`refund`, `payout`) makes the merge impossible; a single translated label invites it.

## ⑤ Glossary final-align (closing audit — admission already happened inline)

Maintain ONE living glossary (ubiquitous language). Sweep the slice's output for terms that slipped past admission; every term → one canonical definition **plus its scope-type (module / function / config)**, headword in the original language. Two ambiguity axes each force a split + re-grill:

- **(a)** a **word** with two meanings (or two words for one thing).
- **(b)** a term that's secretly a **cross-cutting capability** dressed up as a single module.

## Mistakes

| Mistake | Reality |
|---|---|
| Skip the glossary | Term ambiguity (e.g. "user" = merchant or customer?) silently inverts meaning. One canonical definition per term, always. |
| Coin a term now, glossary it later (file/skill/doc/domain names count as coining) | Admission is FRONT-loaded: duplicate-check, org-collision-check (ask the expert), define + scope-type — BEFORE the name spreads. Deferred admission multiplies secondary edits. |
| Glossary headword translated into the working language | Glossary terms are proper nouns — keep the headword in the term's original language; a translation re-merges distinct terms (e.g. one word for both refund and payout). Translations go in the description. |
