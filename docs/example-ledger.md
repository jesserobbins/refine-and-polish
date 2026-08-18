# Worked example: a three-iteration loop

This ledger is a real, lightly trimmed example from a short loop. It shows
the ledger format filled in. It reviewed a small docs-and-config change with
two reviewers, `claude-code` and `codex`, run as separate jobs each
iteration. Read it alongside [the skill](../skills/roborev-refine-and-polish/SKILL.md).
Every section here maps to a section there.

**The point: two independent reviewers found different real bugs. The loop
converged honestly. It did not converge by declaring "zero findings." It
converged by fixing real problems and refuting false ones.**

---

## Loop header

- **Branch / scope:** the change under review (full content of the branch).
- **Instruction (verbatim):** "review this and loop until it's clean."
- **Live reviewers (`roborev check-agents`):** claude-code OK, codex OK, with
  pi available, held in reserve.
- **Convergence criterion:** zero High/Medium and zero Low from any agent, **or**
  the only remaining findings are on the deliberate-pushback list.
- **Budget:** 3 iterations.

## Ledger table

| Iteration | Agent | Sev | Location | Type | Status / Commit |
|------|-------|-----|----------|------|-----------------|
| 1 | claude-code | L | intro link points at a same-page anchor, not the tool's home | NEW | Fixed `71d9cfc` |
| 1 | claude-code | L | install slug stale after a rename | NEW | Already fixed at HEAD (review ran on a pre-rename SHA) |
| 1 | claude-code | L | quick-ref table omits a conditional remedy the prose carries | NEW | Deferred → pushback list |
| 1 | codex | N/A | (whole diff) | N/A | "No issues found" (clean) |
| 2 | claude-code | N/A | (full content) | N/A | "No issues found" |
| 2 | codex | **L** | doc tells users to invoke a bare slash command that won't resolve | NEW | **Verified real** → Fixed `c0e17a2` |
| 3 | claude-code | N/A | (full content) | N/A | "No issues found" (verified every CLI claim against the live tool) |
| 3 | codex | L | re-raises the slash line with an escalated, incorrect claim | LOOP of 2.codex.L | **Refuted with evidence** → pushback |

## Deliberate-pushback list

**1.claude-code.L: quick-ref row omits the conditional remedy.**
The reviewer wanted a one-line remedy added to a summary table. This ledger
does not fix it, for two reasons. First, the remedy is *conditional*. It
applies to one specific error signature only. The table row is generic. If
the remedy is flattened into the row, the cheat-sheet recommends it in cases
where it does not work. The condition stays in the prose because it cannot
survive compression to a row. Second, the row ends in a deliberately absolute
rule. This ledger does not dilute that anchor for a Low finding. *Iteration
N+1 can re-raise this finding word for word. That is not a loop. The
convergence criterion is "zero findings outside this pushback list."*

**3.codex.L: "the slash command won't resolve without a separate command
file."** This ledger verified the claim, then refuted it. An installed,
working plugin in the same environment exposes its skill as a namespaced
slash command, with no separate command file. The official docs confirm the
namespaced form. This ledger made no code change. Adding a file to satisfy a
confirmed-false finding creates unnecessary complexity. This ledger records
the finding as a *verified false positive*, distinct from a design pushback.

## Closing (convergent at Iteration 3, budget 3/3)

Convergence rests on two facts. **claude-code is clean and verified**:
Iteration 3 actively verified every CLI flag and path against the live tool.
**codex's one remaining finding is evidenced-false**. The caption stays
honest about this history. codex *did* produce an Iteration 3 finding. The
finding was refuted, not absent. A caption of "zero findings" misrepresents
this iteration. "One finding, refuted with evidence" states the truth.

**The two-reviewer payoff, concretely.** This is the per-iteration report
table the skill asks for, with each job's id and scope so the coverage
claims below are checkable, not asserted:

| Iteration | Plan | Reviewer results | Findings and decisions | Watch next |
|-----------|------|------------------|------------------------|------------|
| 1 | Baseline pass, full branch, both agents | claude-code job 701 (full branch): 3 Lows. codex job 702 (full branch): clean | claude-code caught three Lows codex missed; fixed or deferred them | Verify codex's independent coverage |
| 2 | Re-review after Iteration 1 fixes, full branch, both agents | claude-code job 703 (full branch): clean. **codex job 704 (full branch): 1 Low, a real bug claude-code missed** | Fixed the broken invocation instruction | Verify the command against live tooling |
| 3 | Re-review after Iteration 2 fix, full branch, both agents | claude-code job 705 (full branch): clean, verified every CLI claim live. codex job 706 (full branch): 1 Low, a refuted claim | Recorded the false positive as pushback | Verify coverage and close |

With a single reviewer, the loop ships the broken instruction. This result is
the argument for two separate reviewer jobs, because their disagreement is
useful signal.

**Coverage:** full. Both reviewers stayed live for all three iterations, with
no waiver and no API-key temptation triggered. **H/M trajectory:** zero High
and zero Medium every iteration. All findings were Low.
