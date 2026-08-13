# Changelog

All notable changes to this project are documented here. The format is based on
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project
follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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

[0.2.0]: https://github.com/jesserobbins/refine-and-polish/releases/tag/v0.2.0
[0.1.0]: https://github.com/jesserobbins/refine-and-polish/releases/tag/v0.1.0
[0.0.2-alpha]: https://github.com/jesserobbins/refine-and-polish/releases/tag/v0.0.2-alpha

## [0.0.1-alpha] - 2026-06-25

This is the first alpha release. It packages the **Robo Refine and Polish**
skill as a standalone, self-installing Claude Code plugin.

### Added

- The `refine-and-polish` skill (`skills/refine-and-polish/SKILL.md`), which
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

[0.0.1-alpha]: https://github.com/jesserobbins/refine-and-polish/releases/tag/v0.0.1-alpha
