# USERS

## Primary

Learners exploring low-level digital logic and computer architecture. Self-paced, self-directed. Reading textbooks, watching lectures, working through exercises elsewhere; come here to *see the thing*.

Concrete shapes:
- Building intuition for a MIPS instruction by stepping through its datapath animation
- Comparing two instructions side-by-side to see what changes
- Practicing K-map grouping on a truth table and self-checking against the solver
- Exploring how a 5-variable K-map's wraparound actually works (the headline 3D win)
- Deriving the Control unit truth table and seeing it minimized in K-map, then watching the same signals fire in the datapath

## Anonymous-first

Default state. Every feature works without identity:
- Pick instruction, step, run, reset
- Author truth table, group cells, see PIs
- Save snapshot → content-addressed permalink works
- Share permalink → recipient opens, sees exact same sim state, no signup wall
- Side-by-side compare, pipeline view, critical path — all anon

## Signed-in

Optional accent for cross-device persistence:
- "My saves" list across devices
- Anonymous snapshots saved in localStorage get claimed silently on signin
- No feature is locked behind signin

## Not addressed

- Real-time multiplayer collaboration
- Classroom / instructor dashboards / grading
- Curriculum-shaped lesson sequencing
- Per-user analytics surface

Listed for clarity; deferred items live in `NON-GOALS.md` with explicit trigger.

## Tone and copy

Audience is technical learners. Copy is direct, accurate, terse. No infantilizing, no gamification, no "great job!" toasts. Pedagogy is the product, not retention loops.

No reference to institution, course code, semester, or program in product surface, copy, docs, OG cards, metadata. Domain language only — "instruction", "control signal", "Boolean minimization", "prime implicant", "datapath".
