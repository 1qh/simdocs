# CLAUDE.md

## Session start protocol

Every Claude session that touches this project runs in this exact order before any work:

1. Cross-project foundationals in the `book` repo (operator-local path lives in agent memory): `PHILOSOPHY.md`, `HARD-RULES.md`, `NON-GOALS.md`, `SUBSTRATE.md`, `CLAUDE.md`, `README.md`. Full read, no skipping, no subagent delegation.

2. This repo's foundationals (root, this order):
   - `VISION.md` — what gets built
   - `GOAL.md` — concrete deliverable
   - `REQUIREMENTS.md` — req breakdown
   - `USERS.md` — audience
   - `NON-GOALS.md` — project scope defenses
   - `SUBSTRATE-VS-PRODUCT.md` — split for this project
   - `AGENT-DOCTRINE.md` — project agent rules
   - `UX-DOCTRINE.md` — industrial 3D UX
   - `A11Y.md` — accessibility floor
   - `PERFORMANCE.md` — perf contract
   - `CONCURRENCY.md` — concurrency + parallelism contract
   - `DETERMINISM.md` — reproducibility contract
   - `STACK.md` — locked picks
   - `API-CONVENTIONS.md`, `GLOSSARY.md`, `OBSERVABILITY.md`
   - `AUTH.md`, `PERSISTENCE.md`, `DEPLOY.md`, `SECURITY.md`, `SCHEMAS.md`
   - `MIPS-DATAPATH.md`, `MIPS-ISA.md`, `KMAP.md`, `PIPELINE.md`, `CRITICAL-PATH.md`, `COMPARE.md`, `ASM-EDITOR.md`
   - `LEARN.md`, `OG-IMAGES.md`
   - `VERIFY.md`, `RUNBOOK.md`
   - `GOTCHAS.md`, `OPEN-QUESTIONS.md`

3. Every file under `adr/`.

4. Sibling `sim` repo's `CLAUDE.md` (short pointer back here).

5. Agent project memory directory for this project. Carries operator-local paths, cross-conversation preferences.

6. Code-repo filesystem inspection: list top-level entries, match against `adr/repo-layout.md` Required components, identify present vs missing.

7. `adr/foundation-bootstrap-order.md` — identify next Phase based on step 6.

8. Code-repo ledger: `make ledger` (gates at HEAD) + `make ledger.stale` (gates not validated at HEAD). Re-running green-at-HEAD gates without reason is a `book/HARD-RULES.md` violation.

Then send the user a single message:
- File counts read per repo
- Foundational summary in own words (PHILOSOPHY mindset + HARD-RULES enforceables + project NON-GOALS + SUBSTRATE split + UX-DOCTRINE + AGENT-DOCTRINE) in 3–5 sentences
- Bootstrap state: components present / missing / next Phase
- Gate state: known-green / known-stale
- Doc gaps flagged in ADR "Gotcha for Claude" sections
- Proposed next concrete action

Do NOT execute work until the user confirms understanding.

Subagents inherit the satisfied state; they execute scoped task in their invocation prompt, do not re-run this protocol.

## No hallucination, no assumption

Fact, path, identifier, decision, constraint not present verbatim in this repo or the code repo: do not know it. Read more files. Read code. Ask the founder. Inventing names / paths / versions / decisions is forbidden.

## Commit messages

- Conventional commits, mandatory type prefix
- No length cap on subject or body; "why" must read cleanly
- Body only when "why" non-obvious
- No AI / Claude / coauthor mentions
- Incremental, smallest meaningful unit
- Silent — no end-of-turn recap

## On learning a recurring rule or gotcha

Update the doc that owns the topic. Never duplicate to a second doc. Commit the doc update with the work that taught it.
