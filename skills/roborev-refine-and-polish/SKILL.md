---
name: roborev-refine-and-polish
description: A generic skill that runs multiple roborev reviewers in a loop and tracks every finding in a private ledger, so you can detect regressions, repeats, and loops, defend deliberate pushback, handle a reviewer going offline, and decide when to stop. Use whenever running roborev refine for more than a couple of iterations, especially with multiple subscription-backed reviewers (claude-code + codex + pi), and ALWAYS when the user says "loop until convergent", "address every finding", "track progress", "iterate to convergence", or asks for budget/extension reasoning. Layer on top of /roborev-refine: that skill runs the loop, and this one runs the multiple reviewers and the ledger that tracks them.
---

# Robo Refine and Polish

A long roborev refine loop without bookkeeping causes problems. You "fix"
the same finding three times. You miss a regression that you introduced two
iterations ago. You re-litigate a deliberate design decision, because the
next agent run cannot see your reasoning. This skill is the bookkeeping.

`/roborev-refine` runs the review-fix-rereview cycle. This skill keeps the
ledger that turns the cycle into a converging process.

Use this skill when:

- The loop is expected to take more than two iterations.
- Two or more reviewer agents are involved, for example claude-code and codex.
- The user has asked for convergence ("loop until clean", "address every
  finding", "max 10 iterations"), or for the loop to be tracked privately.
- A finding has come back after a fix, and you must decide whether to
  re-fix, defend, or escalate it.

If the loop is one quick iteration, this skill is overhead. Skip it.

## Requirements

This skill assumes that [roborev](https://roborev.io) (continuous code
review for AI coding agents) and its `/roborev-refine` command are
installed and configured. roborev is the loop runner this skill enhances.
The ledger uses roborev's job ids, agent names, and Pass/Fail verdicts as
keys. See the [installation guide](https://roborev.io/installation/) to set
it up. Without roborev, the ledger method still makes sense to read, but
the commands in this skill do not run.

## Scope: reviewer output only, not arbitrary third-party text

This skill reads only roborev review findings. Reviewer agents generate
these findings from this repository's code diffs and job logs. You get
them with explicit commands: `roborev review` creates a job and returns its
ID, and `roborev log <job>` and `roborev show <job>` reference that ID.

This skill does not monitor, poll, or read any other feed, queue, or
text channel, for example issue trackers, chat, email, or web content.

The ledger records reviewer verdicts. It does not record third-party
input.

## Reviewers are subscription-backed agents

The reviewers are roborev's local agent CLIs, each on its own subscription:
`claude-code` (Claude), `codex` (ChatGPT), `pi`. roborev's default is a single
agent (`roborev config get default_agent` → usually `claude-code`).
**Multi-reviewer convergence is something you run on purpose:** one review per
agent, each its own job:

```bash
BASE_BRANCH=<your-base-branch>  # e.g. main
MERGE_BASE=$(git merge-base "$BASE_BRANCH" HEAD)
START=$(git rev-list --reverse "$MERGE_BASE"..HEAD | head -1)
HEAD_SHA=$(git rev-parse HEAD)
roborev review "$START" "$HEAD_SHA" --agent claude-code --wait
roborev review "$START" "$HEAD_SHA" --agent codex --wait
```

Both jobs must target the identical scope, so that a convergence decision
covers the whole intended target for every reviewer it rests on. A
per-commit job from one agent does not stand in for a full-branch job
from another: mixing scopes can make a partial review look like it
satisfies two-agent convergence when it does not. Pin an explicit commit
range with positional arguments (`roborev review <start> <end>`), not a
relative selector: `--branch` alone resolves against HEAD at launch
time, and `--wait` blocks until the job finishes, so two sequential
`--branch` calls can end up reviewing different ranges if anything
lands on the branch in between (that gap can be minutes with `--wait`,
not just the moment between two launches). Resolving both ends once and
passing the same pair to both jobs removes the timing dependency
entirely: both calls review that exact range, no matter how long
either job takes or what lands afterward. The positional form is
inclusive of `<start>` (`<start>^..<end>`), so pass the first commit
*unique to your branch*, not the merge-base itself: the merge-base is
shared history with the base branch, and including it can report
pre-existing, unrelated findings as if they belonged to this review.
`git rev-list --reverse "$MERGE_BASE"..HEAD | head -1` resolves that
first-unique commit generically, for any base branch. Do not combine
`--since` with `--sha` expecting it to pin the endpoint, since `--sha`
is silently ignored whenever `--since`
is set. Their findings land in the
ledger under distinct `Agent` values. "claude-code flagged X but
codex did not" is signal, so keep them as separate jobs, not one merged
run.

**Never restore a downed reviewer through the API.** A reviewer's subscription
auth can break mid-loop, most often codex's CLI rejecting under a
ChatGPT-account plan. When this happens, that reviewer is *unavailable*.
Unavailable is a recorded coverage caveat (see below), not a problem to route
around. Do **not** set `OPENAI_API_KEY` / `ANTHROPIC_API_KEY`. Do **not**
pass `--provider openai` with a key. Do **not** "temporarily" swap the
subscription reviewer for an API-keyed one to reach two-agent convergence.
Subscription-only is the constraint. A single-agent pass honestly captioned
beats a two-agent pass faked with an API key. If the user wants API
reviewing, they will ask for it. Never propose it yourself.

At loop start, run `roborev check-agents` and record which reviewers are live in
the ledger header. That one line up front is what turns "codex was down" from a
mid-loop surprise into a known constraint.

## What the ledger captures

Keep one Markdown file per refine loop in the project's private notes
location, `reviews/<pr-or-branch-slug>.md`. Where `reviews/` lives depends
on the project layout: a sibling `<project>-private/` repo that is itself
never published, or a `private/` subdir that is gitignored (never committed)
in an otherwise-public repo. For a repo that is itself private, use a
top-level `reviews/` directory instead. In every case, the ledger must live
somewhere that is never published: a subdirectory alone does not make
content private if it gets committed to a public repo. Follow the project's
instructions file (CLAUDE.md or AGENTS.md). This file is private. Never link
to it from a public PR.

The file holds these, in this order:

1. **Loop header**: branch, PR, the user's instruction in their own
   words, the convergence criterion, and **which reviewers are live**
   (`roborev check-agents` output: for example "claude-code + codex; pi
   available").
2. **Ledger table**: one row per finding, across all iterations and
   all agents.
3. **Open design questions**: anything the loop surfaced that the user
   has decided needs an answer before the loop can converge.
4. **Per-iteration report**: a table that records what you expected to fix,
   what actually happened, what you decided, and what you are watching next.
5. **Deliberate-pushback list**: findings you have decided NOT to fix,
   with the reasoning, so a future re-raise is not a loop.

## The ledger table

This table is the most load-bearing artifact in the ledger. Columns:

| Iteration | Agent | Sev | Location | Type | Status / Commit |

- **Iteration**: `1`, `2`, … or `pre` for findings that were already on
  `main` before the branch, and got picked up incidentally. Use a letter
  suffix (`1b`) for an extra or retry review inside the same iteration.
  Examples: a reviewer coming back online, or a re-run under a different
  model. Do not bump to the next iteration's number for these.
- **Agent**: roborev's agent name, `claude-code`, `codex`, or `pi`.
  This matters because agents disagree, and "claude-code flagged X but
  codex did not" is itself signal. Record the **roborev job id** for each
  agent's run in the per-iteration results (for example "claude-code, job
  699"). This id is the index back into `roborev log <job>` and `roborev
  show <job>`. The ledger row stays a one-line distillation. The job holds
  the full text.
- **Sev**: `H`, `M`, `L`, or a non-finding marker for a row with no
  finding to type. Two different things can produce a non-finding row,
  and they use different markers: a **clean pass** (the agent ran fine
  and found nothing) uses `—`; an **unavailable or errored agent** uses
  `N/A` and must be recorded so the coverage gap is visible, not silent.
  Do not use `N/A` for a clean pass: it reads as a coverage gap when the
  reviewer actually ran. Bold the High rows.
- **Location**: file plus symbol, or a short description. Make it specific
  enough that the next iteration's reviewer output can be matched against it
  without rereading the diff.
- **Type**: `NEW`, `PRE` (pre-existing on main), `REGRESSION of
  <iteration>.<agent>.<sev>` with the closing commit, `REPEAT of <…>`,
  `LOOP of <…>` (a fix re-raised by the first review from that agent
  after it landed; see Type rules below), `—` for a clean-pass row, or `N/A` for an
  unavailable-or-errored-agent row. This skill depends on the type
  column. Do not skip it.
- **Status / Commit**: `Fixed <sha>`, `Fixed <sha> (+ regression
  test)`, `Deferred (see pushback list)`, or `Escalated to user`.

A few filled-in rows show the shape. They include a regression caught two
iterations later, a reviewer offline for an iteration, and a finding sent
to the pushback list:

| Iteration | Agent | Sev | Location | Type | Status / Commit |
|------|-------|-----|----------|------|-----------------|
| 1 | claude-code | **H** | `auth.py` token refresh races | NEW | Fixed `a1b2c3d` |
| 1 | codex | M | `cache.py` unbounded key growth | NEW | Deferred (see pushback list) |
| 2 | claude-code | N/A | (agent unavailable: auth rejection) | N/A | Waived this iteration |
| 2 | codex | M | `auth.py` refresh now drops on 401 | REGRESSION of 1.claude-code.H | Fixed `e4f5a6b` (+ regression test) |
| 3 | codex | L | `cache.py` unbounded key growth | REPEAT of 1.codex.M | No change, see pushback list |

Each project's `reviews/` directory accumulates these files over time.
Reading a recent one, in the project that you are working in, is the
fastest way to see a full loop's worth of rows in context.

## Type rules: regression vs repeat vs loop

These three words are not interchangeable. Get them right or the
ledger lies.

`REGRESSION` takes precedence over `NEW`: a finding that is technically
its first appearance, but was caused by your own fix in a prior
iteration, is a `REGRESSION`, not a `NEW`. Reserve `NEW` for a first
appearance not attributable to a prior fix in this loop.

- **REGRESSION of `<iteration>.<agent>.<sev>`**: *my fix in a prior iteration
  caused this*. Same code path, broken in a new way. The fix needs a
  regression test at the boundary that broke. If you keep regressing
  the same area, slow down. Use smaller commits and paired tests.
- **NEW**: everything else that is a first appearance: not attributable
  to a prior fix in this loop.

`LOOP` and `REPEAT` are mutually exclusive by timing, not by cause: check
whether the finding's prior status was "Fixed", and whether this is the
*first review by this same agent* since that fix landed, whichever
iteration that turns out to be. Both describe the *same underlying
finding* recurring (same symptom, same location), just possibly worded
differently or escalated by the reviewer the second time. A recurrence
that is not the same underlying finding, a genuinely new or different
symptom on the same code path, is not `LOOP` or `REPEAT` at all: it is
`REGRESSION`, which already takes precedence above. Do not require the
recurrence to be a word-for-word match: "identical" means the same
finding, not the same sentence.

- **LOOP**: the same finding, marked "Fixed," recurs in **the first
  review by that same agent to run against that fix**, whichever
  iteration that is. Usually that is the very next iteration, but if the
  agent was unavailable for an iteration or two after the fix landed
  (see "When a reviewer is unavailable" below), its first review back is
  still the LOOP check, not a REPEAT check, since no review by that agent
  ever confirmed the fix held. This is how you discover that your fix and
  the reviewer's expectation disagree on what "fixed" means. Stop fixing
  and reconcile the difference, usually by writing a more pointed test or
  by moving the finding to the deliberate-pushback list with explicit
  reasoning. If reconciliation does not hold and the same finding keeps
  coming back fixed-then-reflagged for three consecutive reviews by that
  agent, that is the escalate-to-user signal in the budget section below:
  more iterations will not help.
- **REPEAT of `<iteration>.<agent>.<sev>`**: everything else. Same
  symptom, same location, as a finding you already recorded, but not the
  first-review-by-that-agent case above. This covers two situations: (a)
  the prior status was "Deferred (see pushback list)": this is the
  expected pushback re-raise, confirm the pushback reasoning still holds
  and leave the code unchanged; (b) the prior status was "Fixed," and at
  least one review by that same agent already ran against that fix
  without re-raising it, before this recurrence: either the reviewer is
  seeing a stale snapshot, the reviewer is wrong, or something later
  reintroduced the symptom. Investigate which, before you change code.
  Case (b) often turns out to be a regression from a *different*, more
  recent commit, not the original fix failing.

Why the distinction matters: a regression rate above about 30%
iteration-to-iteration means your commits are too coarse, and need paired
tests at the regression boundary. A repeat means you must defend the
finding, not patch it. A loop means the *meaning* of the finding is
unsettled, and more code will not help.

## When a reviewer is unavailable

A loop scoped for two agents routinely runs as one. The recurring cause
is codex's CLI rejecting under a ChatGPT-account plan, but any reviewer
can error out. This is the single most common way the loop's assumptions
break, so handle it explicitly instead of quietly dropping to one agent:

1. **Record it as a row**: `N/A` severity, "agent unavailable this iteration,"
   with the actual error (auth rejection, harness incompat). The coverage
   gap belongs in the ledger, not only in your head.
2. **First check for a subscription-only restore.** A `400 ... <model> not
   supported with a ChatGPT account` error is usually a *model-slug
   mismatch*, not a dead subscription. roborev asked codex for a model
   that the plan does not serve. The CLI's own default (for example
   `gpt-5.5`) works fine. Repoint it at a plan-served model, `roborev
   review --agent codex --model <plan-served-slug>`, which keeps the
   reviewer on its subscription. This is the legitimate way to *keep* a
   second reviewer. The API key is still never the answer.
3. **Then one retry, then waive**: if it is not a slug fix, retry once
   (a re-run) as a `1b` sub-iteration. If it still fails, waive that
   reviewer for the loop. Do **not** spend the budget babysitting a
   broken harness, and do **not** reach for an API key to revive it
   (see "Reviewers are subscription-backed agents").
4. **Caption the convergence**: when you declare convergent, a single
   active reviewer is honest *only* with a **reviewer-coverage caveat** in
   the closing. State which reviewer was down, why, and that convergence
   rests on the agent or agents that did run. "Convergent (claude-code
   only; codex structurally unavailable all iters)" is honest. Silently
   treating a one-agent pass as the scoped two-agent cross-check is the
   failure that this caveat exists to prevent.

A waived reviewer is a known, recorded limitation. It is never a silent one.

## roborev's verdict is not your convergence criterion

`roborev review` returns **Fail on any finding at all**, including a lone
Low. It is a mechanical gate, not a judgment about whether the loop is
done. Your convergence criterion (below) decides when to stop. roborev's
Pass/Fail only tells you that findings exist. A re-review that you expect
to come back clean is still a real review. A "confirmatory" pass can
surface a real correctness bug, and a genuine new finding there is real
work, not noise to wave through so that you can declare convergence. Read
every finding on its merits, regardless of which pass produced it.

## The deliberate-pushback list

You must not fix some findings. Document each one in a list at the
bottom of the ledger file, with:

- The finding (severity + location + reviewer's wording).
- **Why you are not fixing it**: at least two reasons, ideally one
  about the code (state of this branch / contract / call sites) and
  one about future direction (what the next planned change does to
  this surface).
- An explicit note: *"Iteration N+1 is expected to re-raise this verbatim.
  That is NOT a loop; convergence criterion is 'zero findings outside
  the deliberate-pushback list.'"*

This is the only mechanism that lets a multi-agent loop converge
when reviewers cannot see prior turns. Without it, every defensible
design call becomes an infinite loop.

A finding belongs on the pushback list when fixing it does one of the
following:

- Contradict a deliberate design decision recorded in a spec or
  proposal.
- Add a transient artifact that an already-planned refactor will
  remove.
- Trade real cost (architectural change, performance, schema
  migration) against a Low-severity hardening with no triggering
  scenario on this branch.
- Turn out, on verification, to be false: you checked the code, the
  call sites, or the live tool, and the claimed problem does not exist.
  Record the evidence, not a code change. This is not a design pushback
  (there is no tradeoff to defend), but it belongs on the same list and
  counts the same way toward the convergence criterion, since the
  finding is fully resolved, just not by editing code.

A finding does **not** belong on the pushback list because the fix
"feels like effort." Fix cosmetic and Low findings too when the fix is
low-effort. Low-effort means roughly under 10 minutes, isolated to one
function or comment, with no new test surface beyond existing patterns.
Fixing is the default. Pushback is the exception.

## Convergence criterion

State it explicitly at the top of the ledger. Do not move it mid-loop.
In practice, this default works well:

> Iteration N produces zero High/Medium and zero Low findings from any
> agent, **or** the only remaining findings are on the deliberate-
> pushback list documented in this file.

Two consequences:

- A pushback re-raise is not a failure. It is the convergence signal.
- A single new Low from any agent breaks convergence. Decide between
  fixing it (default) and pushback (with reasoning).

**Variant: Lows pushback-by-default.** On a branch where a motivated
reviewer produces an endless drip of Lows, gate convergence on
**High/Medium only**. Make Lows pushback-by-default: document them, fix
them only when trivially low-effort (the under-10-minute, one-function,
no-new-test bar), and otherwise defer them. If you use this variant,
state it in the header. This variant changes the stop condition to
"zero H/M outside the pushback list." It keeps the loop from chasing an
unbounded Low tail (see below).

## Budget and extension

Set a budget up front. Five iterations is a reasonable default for
non-trivial branches. Ten is the upper end. Past ten, something is
structurally wrong with the branch, and you must escalate.

When you hit the budget without convergence, **do not silently
continue**. Reassess:

- If the only remaining finding is the expected pushback re-raise,
  declare convergent and stop.
- If material findings remain (any High/Medium, or any Low that the user
  judges worth fixing), extend by another batch of iterations (commonly
  five more, capped at ten). Record the extension decision in the ledger
  so the trajectory is visible.
- If the same finding is fixed and re-flagged across three consecutive
  reviews by the same agent, this is a loop. Escalate to the user with
  the per-iteration trajectory. More iterations will not help.
- If the regression rate trends up across the extended iterations, slow
  down. Use smaller commits and paired regression-boundary tests. A worse
  trajectory at Iteration 7 than at Iteration 3 means the loop is no longer
  converging.

**The Low tail: why budget, not "is the Low surface empty," is the
stop rule.** A motivated reviewer, re-reviewing the previous batch's fixes,
produces a roughly steady stream of *new* Lows every iteration (one observed
run: 5, 3, 4, 4). These are not REPEATs and not regressions. They are a
genuinely unbounded supply, because each fix is itself new surface to
critique. Tell-tale sign: the reviewer's own suggested one-liner draws
the next iteration's finding (a `(x or "").strip()` that still throws on a
non-string `x`). When H/M is clean and only this Low tail remains, stop
on the **budget**, not the convergence criterion. Under the default
criterion, an unresolved new Low still means the criterion is not met:
this is a **budget-exhaustion stop** (see "Budget and extension" above),
captioned honestly in the closing as such, not as "convergent." Under the
**Lows pushback-by-default** variant, new Lows never gate convergence in
the first place, so the same stop genuinely is convergence: caption it
"convergent (Lows pushback-by-default): H/M clean, remaining Lows are a
reviewer tail, not individually pushback-documented." Either way, say so
plainly: "the remaining Lows are a reviewer tail, not an exhausted
surface," never "no Lows remain."

## Per-iteration reporting

Record one row per iteration in the private ledger, using this table:

| Iteration | Plan | Reviewer results | Findings and decisions | Watch next |
|-----------|------|------------------|------------------------|------------|
| 1 | What you expect to fix and what must not recur | Job IDs, reviewers, and concise results | Findings fixed, deferred, refuted, or escalated | Regression hazards and expected re-raises |

Write the plan before re-running the reviewers. Add the results as they land,
including each agent's job id and that job's scope (a single commit, or the
full branch/history). Per-commit reviews and branch-level reviews can go in
the same row, but only if each job's scope is recorded next to its id: a
per-commit job does not stand in for full-branch coverage, and blending them
without saying so can make a partial review look like it satisfied the
convergence criterion for the whole branch. An extra review inside an
iteration (a reviewer retry or a re-run) is a `1b`-style sub-iteration, not a
new iteration. Update the ledger table as part of writing the report row,
not afterward.

When you act on a finding:

- **Fix the surface, not the reviewer's literal line.** A reviewer's
  suggested one-liner is a hint, not a patch to paste. If its fix still
  fails a neighboring case, fix the actual surface, and add the test that
  the one-liner missed. Pasting the literal suggestion is how you
  manufacture the next iteration's finding. See
  superpowers:receiving-code-review.
- **Verify-flagged findings.** When a reviewer says "verify, not a
  confirmed bug," *verify it*: read the call sites, and verify or refute
  the claim. Record the result. A no-change verification is a valid
  outcome, for example "verified no consumer keys on this field →
  pushback." Do not change code to silence a flag that you have not
  verified.

## Post-convergence follow-ups

For deferred Lows that you still want done after the loop closes, land them
in a **single follow-up commit**. This is explicitly not a refine
iteration: no re-review, no new ledger iteration. Note it in the closing ("deferred Lows
landed in `<sha>` post-convergence; not a 4th iteration"). This keeps the
trajectory honest: the loop converged at Iteration N, and this cleanup on
top is not evidence that the loop needed to run longer.

## Workflow

1. **At the start of the loop**: run `roborev check-agents` to see which
   subscription reviewers are live. Create the ledger file: header (with
   the live-reviewer line), the convergence criterion, an empty ledger
   table, and (if you already know of any) the deliberate-pushback list.
   If the ledger lives in a sibling private repo or a private repo's own
   top-level `reviews/`, commit the file there before Iteration 1 starts.
   If it lives in a gitignored `private/` subdir of an otherwise-public
   repo, it is never committed at all: rely on the filesystem, not git
   history, for that trajectory.
2. **Run Iteration 1** via `/roborev-refine` (or its underlying commands).
   When findings land, add a row per finding to the ledger. Type each
   one (NEW or PRE for Iteration 1).
3. **Before each subsequent iteration**: write the next report row's plan.
   This forces you to predict the run, instead of only consuming it.
4. **After each iteration**: fill in the results, type each finding,
   update the ledger and report tables, and, if convergence is in sight,
   check the criterion before you start the next iteration.
5. **At convergence or budget exhaustion**: write a closing section.
   For convergence: list the pushback findings that justified stopping,
   the H/M trajectory, and a reviewer-coverage caveat if any reviewer
   was down. For exhaustion: the iteration-by-iteration table, plus what
   is still surfacing, ready to show the user.
6. **Commit the ledger file at every iteration boundary, when it lives in
   a repo that is committed at all** (a sibling private repo, or a private
   repo's top-level `reviews/`). This is private notes, not public
   artifacts, so commits cost nothing there, and the reconstructable
   trajectory is the whole point. A gitignored `private/` subdir is never
   committed, by construction; skip this step for that layout.

## Things to avoid

- Do not paste reviewer output verbatim into the ledger. Distill it to
  one row per finding. The reviewer's full output already exists in
  the daemon's job log. The ledger is the index.
- Do not link to the ledger file from a public PR or commit message.
  That leaks the path and the private repo's existence. If a finding
  needs public discussion, write it up fresh in the PR.
- Do not retroactively rewrite past iteration rows. If a prior call was
  wrong, append a correction. The trajectory is the diagnostic.
- Do not move a finding from "deliberate pushback" to "fixed" without
  noting why the prior reasoning no longer applies. Changing your
  mind is fine. Silently reversing it is the thing that turns a
  defended decision into a loop.
- Do not reach for an API key to revive a downed subscription reviewer.
  Do not quietly drop to one agent without a coverage caveat. Both
  fake a convergence that the loop did not actually earn.

## Quick reference

| Situation | What to do |
|-----------|------------|
| Start of loop | `roborev check-agents`; record live reviewers in the header |
| Run two reviewers | `roborev review <start> <end>` with the same resolved range, `--agent claude-code` and `--agent codex`, separate jobs |
| Reviewer errored / auth-broke | `N/A` row, "unavailable"; one retry as `1b`; waive + coverage caveat. **No API key.** |
| roborev verdict = Fail, only Lows | roborev fails on any finding; your criterion governs, not its verdict |
| Typing a recurrence | `REGRESSION` (your fix broke it) / `REPEAT` (defend) / `LOOP` (reconcile), see Type rules |
| Reviewer says "verify, not a bug" | verify it, record the result (no-change is valid) |
| Reviewer hands you a one-liner fix | fix the surface, not the literal line |
| H/M clean, Lows keep dripping | default criterion: budget-exhaustion stop. Lows-pushback-by-default variant: genuine convergence. Either way, caption it as a reviewer tail, not "no Lows left" |
| Deferred Lows you still want done | single follow-up commit, not a refine iteration; no re-review |

## See also

- `/roborev-refine`, the underlying refine mechanics this skill sits
  on top of.
- `/roborev-fix` for a single-pass fix without re-review (no ledger
  needed).
- `roborev check-agents` to see which subscription reviewers are live
  before you scope the loop.
- superpowers:receiving-code-review, which covers how to weigh a finding
  before you act on it: verify it, do not perform agreement, do not
  blind-apply it. Use this if you have the superpowers skill set
  installed. Otherwise the principle stands on its own.
- A recent ledger in this project's `reviews/` directory, showing the
  shape filled in, with a full loop's worth of typed rows in context.
