# Robo Refine and Polish

> **Status: `0.0.2-alpha`.** This is an early release. The discipline is
> proven and tested in practice, but the packaging is new. The two-step
> install below follows the documented plugin-marketplace flow and matches
> other working plugins. No fresh user has run the install from start to
> finish yet. Expect rough edges in the installation steps, not in the
> method. Feedback is welcome.

A Claude Code skill that keeps a long [`roborev`](https://roborev.io) refine loop
honest.

`/roborev-refine` runs the review, fix, and re-review cycle. **Robo Refine
and Polish** adds the bookkeeping that turns this cycle into a process that
converges. A private ledger tracks every finding across iterations and
reviewer agents. The ledger lets you tell a regression from a repeat from a
loop. It lets you defend a deliberate design choice without arguing it
again each iteration. It lets you handle a reviewer that goes offline
without a false claim of full coverage. It helps you decide, with honesty,
when to stop.

Without the ledger, a long loop causes problems. You fix the same finding
three times. You miss a regression that you introduced two iterations ago.
You re-argue a decision that the next agent run cannot see.

## What it gives you

- **A typed ledger table**: one row per finding, across all iterations and
  all reviewer agents. A `Type` column forces the distinction between `NEW`,
  `REGRESSION`, `REPEAT`, and `LOOP`.
- **Multi-reviewer discipline.** `claude-code`, `codex`, and `pi` run as
  separate jobs. If one reviewer flags an issue and another does not, that
  difference stays visible as signal.
- **A deliberate-pushback list**, the mechanism that lets a multi-agent loop
  converge when reviewers cannot see prior turns. A defended decision stops
  the loop from repeating without end.
- **An explicit convergence criterion and iteration budget**, including the
  honest "budget stop" for an unbounded tail of Low-severity findings.
- **Coverage caveats**: when a subscription reviewer goes down, the ledger
  records the limitation. It never drops silently to one agent.

## Requirements

This skill builds on **[roborev](https://roborev.io)** (continuous code review
for AI coding agents) and its `/roborev-refine` loop. roborev is the loop
runner that this discipline sits on top of. The ledger uses roborev's job
IDs, agent names, and Pass/Fail verdicts as keys. Install roborev first (see
the [roborev installation guide](https://roborev.io/installation/) and
[quick start](https://roborev.io/quickstart/)). Without roborev, the
methodology still applies, but the commands in this skill do not run.

## Installation

This repo is its own plugin marketplace. Install it in two steps: add the
marketplace, then install the plugin.

```sh
/plugin marketplace add jesserobbins/refine-and-polish
/plugin install refine-and-polish@refine-and-polish
```

The skill activates automatically when you run a multi-iteration roborev
refine loop. You can also invoke it directly with
`/refine-and-polish:roborev-refine-and-polish`. Plugin skills use this
namespaced form.

## Quickstart: your first loop

After installation, a two-reviewer loop looks like this. The skill drives
these steps. You do not have to memorize them.

```sh
# 1. See which subscription reviewers are live, then record them in the ledger header.
roborev check-agents

# 2. Create the ledger (private notes, never committed to a public repo):
#    header, convergence criterion, and empty tables. The skill writes these for you.

# 3. Iteration 1: run each reviewer as its own job, same scope:
roborev review --branch --agent claude-code --wait
roborev review --branch --agent codex --wait

# 4. Add one row per finding to the ledger and TYPE each one
#    (NEW / REGRESSION / REPEAT / LOOP). Fix what's real; defend what isn't.

# 5. Before each next iteration, add an "Iteration N" report row that predicts the run.
#    Re-review, update the tables, check the convergence criterion. Repeat.
```

You stop the loop when an iteration produces zero findings outside the
deliberate-pushback list. You also stop when you reach your iteration
budget. In that case, describe the remaining tail of Low-severity findings
with honesty. See the [worked example](./docs/example-ledger.md) for a
three-iteration loop with a filled-in ledger, a pushback entry, and an
honest convergence caption.

## Usage

If a refine loop runs for more than two iterations, use this skill. Use it
especially with two or more reviewer agents. Also use it when someone asks
you to "loop until convergent," "address every finding," or "iterate to
convergence." The skill helps you create the ledger, type each finding, and
plan each iteration before you run it. When the loop converges or reaches
its budget, it also helps you write an honest closing note.

Read [`skills/roborev-refine-and-polish/SKILL.md`](./skills/roborev-refine-and-polish/SKILL.md)
for the full discipline. Read the [worked example](./docs/example-ledger.md)
for a filled-in loop.

## Known limitations (alpha)

- **Install not yet verified end-to-end.** The two-step marketplace install
  matches the documented flow and working plugins, but no fresh user has run
  it start to finish. If `/plugin install` fails, this step is the most
  likely cause.
- **roborev is required.** This is a discipline *on top of* roborev, not a
  standalone tool. See [Requirements](#requirements).
- **The discipline assumes subscription-backed reviewers.** It deliberately
  does not route a downed reviewer through an API key. If your setup uses
  only an API key, the "reviewer unavailable" handling will not match your
  situation.

## Author

[Jesse Robbins](https://jesserobbins.com) (@jesserobbins) built this skill on top of [roborev](https://roborev.io).

## License

[MIT](./LICENSE)
