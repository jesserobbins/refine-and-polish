# Robo Refine and Polish

This is for anyone who commits to open source and wants to hold a high
quality bar before merging.

It's a generic Claude Code skill that runs multiple
[roborev](https://roborev.io) reviewers in a loop: review, fix, re-review,
and keep going until independent reviewers stop finding real problems. A
private ledger tracks every issue and every change along the way, so the
loop actually converges instead of going in circles.

> **Status: `0.2.1`.** The skill is proven and tested in practice, and the
> two-step marketplace install has been run end to end. Feedback is
> welcome.

## The problem it solves

A long review-fix-review loop sounds simple, but without bookkeeping it
falls apart in familiar ways. You "fix" the same finding twice because
nobody remembers it already came up. You miss a regression you introduced
two iterations back. You re-argue a design decision every time a new
reviewer run can't see the reasoning from last time.

The ledger is what stops all three. It's a plain Markdown file, kept
alongside your private notes, that records:

- **Every finding, typed.** Each one gets marked `NEW`, `REGRESSION`,
  `REPEAT`, or `LOOP`, so you can immediately tell "this is new" from "I
  already fixed this and it broke again."
- **A pushback list.** When you deliberately decide not to fix something,
  you write down why, once. When the reviewer raises it again next
  iteration, that's expected, not a failure.
- **Coverage caveats.** If a reviewer goes offline mid-loop, the ledger
  says so, out loud. It never quietly drops to one agent and calls it full
  coverage.
- **A real stop condition.** You're done when an iteration comes back with
  zero findings outside the pushback list, or when you hit an honest
  budget stop on a tail of small, low-severity nitpicks.

In the [worked example](./docs/example-ledger.md), running two reviewers
instead of one is what catches a real bug: `codex` flags a broken slash
command that `claude-code` missed entirely. With a single reviewer, that
bug ships. That disagreement between reviewers is the whole point, and the
ledger is what keeps it from getting lost.

## How it works

```sh
roborev check-agents                                  # which reviewers are live
BASE_BRANCH=<your-base-branch>                        # e.g. main
MERGE_BASE=$(git merge-base "$BASE_BRANCH" HEAD)
START=$(git rev-list --reverse "$MERGE_BASE"..HEAD | head -1)
HEAD_SHA=$(git rev-parse HEAD)
roborev review "$START" "$HEAD_SHA" --agent claude-code --wait
roborev review "$START" "$HEAD_SHA" --agent codex --wait
```

1. Run each reviewer as its own job, against the same resolved commit range.
2. Log every finding in the ledger, typed and dated.
3. Fix the real findings. Defend, in writing, the ones you won't fix.
4. Re-review. Repeat until findings run out or you hit your iteration
   budget.

Read the [worked example](./docs/example-ledger.md) to see a real
three-iteration loop, ledger and all. Read the
[full skill](./skills/roborev-refine-and-polish/SKILL.md) for every rule
behind it.

## Requirements

This skill sits on top of [roborev](https://roborev.io), continuous code
review for AI coding agents. Install roborev first: see its
[installation guide](https://roborev.io/installation/) and
[quick start](https://roborev.io/quickstart/). Without roborev, this skill's
methodology still makes sense to read, but its commands won't run.

## Installation

This repo is its own plugin marketplace, so installing it is two commands.

```sh
/plugin marketplace add jesserobbins/refine-and-polish
/plugin install refine-and-polish@refine-and-polish
```

Once installed, the skill activates automatically whenever you run a
multi-iteration roborev refine loop. You can also invoke it directly with
`/refine-and-polish:roborev-refine-and-polish`.

## Known limitations

- **Requires roborev.** This skill enhances it. It does not replace roborev
  or run on its own.
- **Assumes subscription-backed reviewers.** It deliberately won't route a
  downed reviewer through an API key. See the skill for why.

## Author

[Jesse Robbins](https://jesserobbins.com) (@jesserobbins) built this on top
of [roborev](https://roborev.io).

## License

[MIT](./LICENSE)
