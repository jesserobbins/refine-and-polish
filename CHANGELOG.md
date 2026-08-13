# Changelog

All notable changes to this project are documented here. The format is based on
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project
follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.0.2-alpha] — 2026-08-05

### Changed

- Rename the standalone skill to `roborev-refine-and-polish`.
- Use `Iteration` consistently and record per-iteration plans and results in a
  table.

[0.0.2-alpha]: https://github.com/jesserobbins/refine-and-polish/releases/tag/v0.0.2-alpha

## [0.0.1-alpha] — 2026-06-25

This is the first alpha release. It packages the **Robo Refine and Polish**
discipline as a standalone, self-installing Claude Code plugin.

### Added

- The `refine-and-polish` skill (`skills/refine-and-polish/SKILL.md`) — the full
  discipline for tracking a multi-iteration roborev refine loop in a private
  ledger: typed findings (`NEW` / `REGRESSION` / `REPEAT` / `LOOP`), the
  deliberate-pushback list, reviewer-coverage caveats, an explicit convergence
  criterion and iteration budget, and the honest "budget stop" for an unbounded
  Low-severity tail.
- Plugin packaging: `plugin.json` and a self-contained `marketplace.json`, so the
  repo installs as its own marketplace.
- A [worked example](./docs/example-ledger.md) — a real three-iteration loop with
  a filled-in ledger, a pushback entry, and an honest convergence caption.
- README quickstart and an alpha-status note.

### Known limitations

- The two-step marketplace install matches the documented plugin flow and known
  working plugins. No new user tested it from start to finish.
- Requires the [roborev](https://roborev.io) CLI. This discipline works as a layer
  on top of roborev, not as a standalone tool.
- Assumes subscription-backed reviewers. The "reviewer unavailable" handling
  refuses API-key fallback on purpose.

[0.0.1-alpha]: https://github.com/jesserobbins/refine-and-polish/releases/tag/v0.0.1-alpha
