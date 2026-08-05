# Robo Refine and Polish

> **Status: `0.0.2-alpha`.** Early release. The discipline is real and
> battle-tested in practice, but the packaging is new: the two-step install
> below follows the documented plugin-marketplace flow and matches working
> plugins, but hasn't been run end-to-end by a fresh user yet. Expect rough
> edges in the plumbing, not the method. Feedback welcome.

A Claude Code skill that keeps a long [`roborev`](https://roborev.io) refine loop
honest.

`/roborev-refine` runs the review → fix → re-review cycle. **Robo Refine and
Polish** is the bookkeeping that turns that cycle into a *converging* process: a private
ledger that tracks every finding across iterations and reviewers, so you can
tell a regression from a repeat from a loop, defend a deliberate design call
without re-litigating it every iteration, handle a reviewer going offline
without faking coverage, and decide — honestly — when to stop.

Without the ledger, a long loop is how you end up "fixing" the same finding
three times, missing a regression you introduced two iterations ago, and
silently re-arguing a decision the next agent run can't see.

## What it gives you

- **A typed ledger table** — one row per finding, across all iterations and all
  reviewer agents, with a `Type` column that forces the distinction between
  `NEW`, `REGRESSION`, `REPEAT`, and `LOOP`.
- **Multi-reviewer discipline** — run `claude-code`, `codex`, and `pi` as
  separate jobs so "one reviewer flagged it, another didn't" stays as signal.
- **A deliberate-pushback list** — the mechanism that lets a multi-agent loop
  converge when reviewers can't see prior turns; a defended decision stops being
  an infinite loop.
- **An explicit convergence criterion and iteration budget** — including the
  honest "budget stop" for an unbounded tail of Low-severity findings.
- **Coverage caveats** — when a subscription reviewer goes down, that's a
  recorded limitation, never a silent drop to one agent.

## Requirements

This skill builds on **[roborev](https://roborev.io)** — continuous code review
for AI coding agents — and its `/roborev-refine` loop. roborev is the loop runner
this discipline sits on top of: the ledger is keyed to roborev's job ids, agent
names, and Pass/Fail verdicts. Install it first (see the
[roborev installation guide](https://roborev.io/installation/) and
[quick start](https://roborev.io/quickstart/)); without it, the methodology still
reads, but the commands in this skill won't run.

## Installation

This repo is its own plugin marketplace, so install it in two steps — add the
marketplace, then install the plugin:

```sh
/plugin marketplace add jesserobbins/refine-and-polish
/plugin install refine-and-polish@refine-and-polish
```

The skill activates automatically when relevant — when you run a multi-iteration
roborev refine loop — and you can also invoke it explicitly with
`/refine-and-polish:roborev-refine-and-polish` (plugin skills are namespaced by plugin).

## Quickstart: your first loop

Once installed, a two-reviewer loop looks like this. The skill drives these
steps; you don't have to memorize them.

```sh
# 1. See which subscription reviewers are live — record them in the ledger header.
roborev check-agents

# 2. Create the ledger (private notes — never committed to a public repo):
#    header, convergence criterion, and empty tables. The skill writes these for you.

# 3. Iteration 1 — run each reviewer as its own job, same scope:
roborev review --branch --agent claude-code --wait
roborev review --branch --agent codex --wait

# 4. Add one row per finding to the ledger and TYPE each one
#    (NEW / REGRESSION / REPEAT / LOOP). Fix what's real; defend what isn't.

# 5. Before each next iteration, add an "Iteration N" report row — predict the run.
#    Re-review, update the tables, check the convergence criterion. Repeat.
```

You stop when an iteration produces zero findings outside the deliberate-pushback
list — or when you hit your iteration budget and caption the remaining Low tail
honestly. See the [worked example](./docs/example-ledger.md) for a real
three-iteration loop with a filled-in ledger, a pushback entry, and an honest
convergence caption.

## Usage

Use it whenever a refine loop will run for more than a couple of iterations —
especially with two or more reviewer agents, or when you've been asked to "loop
until convergent", "address every finding", or "iterate to convergence". The
skill walks you through creating the ledger, typing each finding, planning each
iteration before you run it, and writing an honest closing when the loop
converges or hits its budget.

For the full discipline, read
[`skills/roborev-refine-and-polish/SKILL.md`](./skills/roborev-refine-and-polish/SKILL.md); for a
filled-in loop, read the [worked example](./docs/example-ledger.md).

## Known limitations (alpha)

- **Install not yet verified end-to-end.** The two-step marketplace install
  matches the documented flow and working plugins, but no fresh user has run it
  start to finish. If `/plugin install` misbehaves, that's the most likely spot.
- **roborev is required.** This is a discipline *on top of* roborev, not a
  standalone tool — see [Requirements](#requirements).
- **The discipline assumes subscription-backed reviewers.** It deliberately
  refuses to route a downed reviewer through an API key; if your setup is
  API-key-only, the "reviewer unavailable" handling won't match your situation.

## License

[MIT](./LICENSE)
