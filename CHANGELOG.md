# Changelog

All notable changes to this project are documented here. The format is based on
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project
follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Changed

- Closed a gap where `LOOP` was described as a finding type in the Type
  rules prose since `0.0.1-alpha` but was missing from the ledger's
  formal, permitted `Type` value list. Also drew a line between `LOOP`
  and `REPEAT`: `LOOP` is a fixed finding recurring in the first review
  by the same agent to run against that fix, `REPEAT` is everything else
  (a deferred finding's expected re-raise, or a fixed finding recurring
  after at least one review by that agent already ran against the fix
  without re-raising it).
- Added `N/A` as a permitted ledger `Type` value, for a non-finding row
  (agent unavailable or errored). **Breaking for existing ledgers:** the
  `Sev` column's non-finding marker also changed. A clean pass (agent
  ran, found nothing) now uses `—`, and `N/A` is reserved strictly for an
  unavailable or errored agent. A `—` row in a ledger written before this
  change meant "unavailable"; retype any such row to `N/A` before relying
  on `—` to mean "clean" in that ledger.
- Gave `REGRESSION` precedence over `NEW`: a finding that is technically
  its first appearance, but was caused by your own fix in a prior
  iteration, is typed `REGRESSION`, not `NEW`.
- Closed a gap where a same-finding recurrence that was not a
  word-for-word match fell into neither `LOOP` nor `REPEAT`: both now
  cover the same underlying finding recurring, whether reworded or
  escalated by the reviewer the second time, not only an identical
  re-statement. A genuinely different symptom on the same code path is
  `REGRESSION`, not `LOOP` or `REPEAT`.
- Anchored `LOOP` and `REPEAT` on the first review by the same agent to
  run against a fix, not on iteration count: a reviewer unavailable for
  one or more iterations after a fix landed used to make its eventual
  first review back match neither type. The escalation rule (three
  consecutive recurrences) is now stated the same way, for consistency.
- Added a fourth admission criterion to the deliberate-pushback list for
  a finding verified false on investigation: record the evidence, not a
  code change. It counts toward convergence the same way a design
  pushback does.
- Fixed the two-reviewer examples (README, skill, quick reference), which
  ran sequential `--branch --wait` calls, a real race that could review
  different ranges if a commit landed between launches. They now resolve
  a commit range once and pass the identical `<start> <end>` pair to both
  jobs.
- Clarified that the ledger's `private/` subdir location must be
  gitignored, not merely a subdirectory, and that a gitignored ledger is
  never committed (adjusted the workflow's commit steps to match).
- Required recording each review job's scope (single commit vs. full
  branch/history) when its row shares a report table entry with another
  job, and required every reviewer job used for a convergence decision to
  target the identical scope.
- Distinguished a true budget-exhaustion stop from a genuine convergence
  stop in the Low-tail guidance: under the default criterion an
  unresolved new Low is budget exhaustion, not convergence; under the
  Lows-pushback-by-default variant it is convergence.

## [0.2.1] - 2026-08-14

### Changed

- Described the skill by what it does, not why: "a generic skill that runs
  multiple roborev reviewers in a loop and tracks every finding in a
  private ledger," replacing "keeps a long loop honest," in the README,
  the skill's triggering description, and the plugin manifests.
- Fixed a stale `0.1.0` status badge in the README.

## [0.2.0] - 2026-08-13

### Changed

- Rewrote the README to lead with who the skill is for, what it does, and
  how it works, with a concrete example pulled from the worked example.
- Replaced "discipline" throughout the README, the skill (including its
  triggering description), the changelog, and the plugin manifests with
  plainer language: this is a skill that enhances roborev, not a
  discipline layered on top of it.
- Renamed the skill's "Type discipline" section to "Type rules".

## [0.1.0] - 2026-08-13

This is the first release out of alpha. The two-step marketplace install has
been run end to end.

### Added

- A scope note in the skill that states plainly what the ledger reads: only
  roborev's own review findings, tied to a specific job ID, never an
  outsider-authored feed, queue, or text channel.
- An Author section in the README.

### Changed

- Rewrote all prose in the README, this changelog, the worked example, and
  the skill itself in Simplified Technical English: short sentences, active
  voice, no banned modals, no filler.
- Removed the alpha status note and the "install not yet verified" known
  limitation, now that the install has been verified end to end.

## [0.0.2-alpha] - 2026-08-05

### Changed

- Rename the standalone skill to `roborev-refine-and-polish`.
- Use `Iteration` consistently and record per-iteration plans and results in a
  table.

## [0.0.1-alpha] - 2026-06-25

This is the first alpha release. It packages the **Robo Refine and Polish**
skill as a standalone, self-installing Claude Code plugin.

### Added

- The `refine-ledger` skill (`skills/refine-ledger/SKILL.md`, later renamed
  to `roborev-refine-and-polish`), which
  holds the full method for tracking a multi-iteration roborev refine loop
  in a private ledger: typed findings (`NEW` / `REGRESSION` / `REPEAT` /
  `LOOP`), the deliberate-pushback list, reviewer-coverage caveats, an
  explicit convergence criterion and iteration budget, and the honest "budget
  stop" for an unbounded Low-severity tail.
- Plugin packaging: `plugin.json` and a self-contained `marketplace.json`, so the
  repo installs as its own marketplace.
- A [worked example](./docs/example-ledger.md): a real three-iteration loop
  with a filled-in ledger, a pushback entry, and an honest convergence
  caption.
- README quickstart and an alpha-status note.

### Known limitations

- The two-step marketplace install matches the documented plugin flow and known
  working plugins. No new user tested it from start to finish.
- Requires the [roborev](https://roborev.io) CLI. This skill enhances
  roborev. It does not work as a standalone tool.
- Assumes subscription-backed reviewers. The "reviewer unavailable" handling
  refuses API-key fallback on purpose.

[Unreleased]: https://github.com/jesserobbins/refine-and-polish/compare/v0.2.1...HEAD
[0.2.1]: https://github.com/jesserobbins/refine-and-polish/releases/tag/v0.2.1
[0.2.0]: https://github.com/jesserobbins/refine-and-polish/releases/tag/v0.2.0
[0.1.0]: https://github.com/jesserobbins/refine-and-polish/releases/tag/v0.1.0
[0.0.2-alpha]: https://github.com/jesserobbins/refine-and-polish/releases/tag/v0.0.2-alpha
[0.0.1-alpha]: https://github.com/jesserobbins/refine-and-polish/releases/tag/v0.0.1-alpha
