# Worked example: a three-iteration loop

This is a real (lightly trimmed) ledger from a short loop, to show the shape
filled in. It reviewed a small docs-and-config change with two reviewers,
`claude-code` and `codex`, run as separate jobs each iteration. Read it
alongside [the skill](../skills/roborev-refine-and-polish/SKILL.md) — every section here
maps to a section there.

The point it illustrates: **two reviewers caught different real bugs, and the
loop converged honestly — not by declaring "zero findings," but by fixing what
was real and refuting what wasn't.**

---

## Loop header

- **Branch / scope:** the change under review (full content of the branch).
- **Instruction (verbatim):** "review this and loop until it's clean."
- **Live reviewers (`roborev check-agents`):** claude-code OK, codex OK; pi
  available, held in reserve.
- **Convergence criterion:** zero High/Medium and zero Low from any agent, **or**
  the only remaining findings are on the deliberate-pushback list.
- **Budget:** 3 iterations.

## Ledger table

| Iteration | Agent | Sev | Location | Type | Status / Commit |
|------|-------|-----|----------|------|-----------------|
| 1 | claude-code | L | intro link points at a same-page anchor, not the tool's home | NEW | Fixed `71d9cfc` |
| 1 | claude-code | L | install slug stale after a rename | NEW | Already fixed at HEAD (review ran on a pre-rename SHA) |
| 1 | claude-code | L | quick-ref table omits a conditional remedy the prose carries | NEW | Deferred → pushback list |
| 1 | codex | — | (whole diff) | — | "No issues found" — clean |
| 2 | claude-code | — | (full content) | — | "No issues found" |
| 2 | codex | **L** | doc tells users to invoke a bare slash command that won't resolve | NEW | **Verified real** → Fixed `c0e17a2` |
| 3 | claude-code | — | (full content) | — | "No issues found" (verified every CLI claim against the live tool) |
| 3 | codex | L | re-raises the slash line with an escalated, incorrect claim | REPEAT of 2.codex.L | **Refuted with evidence** → pushback |

## Deliberate-pushback list

**1.claude-code.L — quick-ref row omits the conditional remedy.**
Reviewer wanted a one-line remedy added to a summary table. Not fixing, two
reasons: (1) the remedy is *conditional* — it only applies to one specific
error signature, and the table row is generic, so flattening it in would make
the cheat-sheet recommend the remedy in cases where it doesn't work; the
condition lives in prose because it can't survive compression to a row. (2) the
row ends in a deliberately absolute rule, and diluting that anchor for a Low
isn't worth it. *Iteration N+1 may re-raise this verbatim — that is NOT a loop; the
convergence criterion is "zero findings outside this pushback list."*

**3.codex.L — "the slash command won't resolve without a separate command
file."** Verified and **refuted**: an installed, working plugin in the same
environment exposes its skill as a namespaced slash command with no such file,
and the official docs confirm the namespaced form. No code change — adding a
file to satisfy a confirmed-false flag would manufacture complexity. Recorded as
a *verified false positive*, distinct from a design-pushback.

## Closing — convergent at Iteration 3 (budget 3/3)

Convergence rests on: **claude-code clean and verified** (Iteration 3 actively checked
every CLI flag and path against the live tool), and **codex's one remaining
finding evidenced-false**. Note the honest caption — codex *did* produce an
Iteration 3 finding; it was refuted, not absent. "Zero findings" would have been a
lie; "one finding, refuted with evidence" is the truth.

**The two-reviewer payoff, concretely:**

| Iteration | Reviewer results | Findings and decisions | Watch next |
|-----------|------------------|------------------------|------------|
| 1 | claude-code caught three Lows codex missed | Fixed or deferred them | Check codex's independent coverage |
| 2 | **codex caught one real bug claude-code missed** | Fixed the broken invocation instruction | Re-check the command against live tooling |
| 3 | Reviewers split on a refuted claim; live-tool verification settled it | Recorded the false positive as pushback | Confirm coverage and close |

A single-reviewer loop would have shipped the broken instruction. That is the
whole argument for running two reviewers as separate jobs and keeping their
disagreement as signal.

**Coverage:** full — both reviewers live all three iterations, no waiver, no
API-key temptation triggered. **H/M trajectory:** zero High, zero Medium every
iteration; all findings Low.
