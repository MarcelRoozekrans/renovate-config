# AGENTS.md: renovate-config

A shared Renovate preset. The repo holds one file, `default.json`, which other repositories extend with
`"extends": ["github>MarcelRoozekrans/renovate-config"]`; Renovate resolves that shorthand to
`default.json` on the default branch. It sets semantic `chore(deps)` commits, auto-merge of non-major
updates, and grouping rules for packages that must move together. There is no code, no README and no CI.

## Exceptions to the org rules

- **Releases.** This repo has no release-please and no release. The org rule that commit types decide
  a release does not apply. Every merge to the default branch changes Renovate behaviour in every
  repo that extends the preset, on that repo's next Renovate run. Review a change as a production change.
- **CI and coverage.** This is an exception to the org CI rules: there is no `ci-baseline` workflow,
  no tests, no coverage gate and no Codecov. Nothing checks a PR automatically, so run the validator
  below before you open one.
- **Commit types.** Conventional Commits still hold. This repo's history uses `feat:` for a new
  grouping or merge rule and `chore:` for a plain setting change, but the type releases nothing.

The branch and PR rule holds: never push to the default branch.

## Build and test

Prerequisite: Node.js with `npx` (no repo-local install). Validate after every edit; this passed on
the current `default.json`:

```bash
npx --yes --package renovate -- renovate-config-validator default.json
```

It prints `Config validated successfully`. It cannot tell whether a rule matches the packages you meant, so check a new rule's regex by hand.

## Conventions and gotchas

- **Majors stay manual.** The first rule auto-merges only `minor`, `patch`, `digest` and `pin`.
  Do not add `major` to it, and do not set `automerge` on a group rule: the update-type rule above applies to a grouped
  PR as well, which keeps a major of a grouped package manual too.
- **`platformAutomerge` is `false` on purpose.** GitHub's native auto-merge waits for the required
  review even for an app with a ruleset bypass, so Renovate merges the PR itself once checks are green.
  The top-level `description` array records this; keep it true if you change the setting.
- **One ZeroAlloc grouping rule per family.** A generator is version-locked to its runtime, so
  `ZeroAlloc.X` and `ZeroAlloc.X.Generator` move in one PR. Renovate cannot template `groupName` per
  package, so a new `ZeroAlloc.*.Generator` needs a new rule with its own
  `"/^ZeroAlloc\\.X(\\.Generator)?$/"` regex and `"groupName": "ZeroAlloc.X + generator"`, restricted to
  `matchManagers: ["nuget"]`.
- **Anchor every regex.** Use `^` and `$`, with the dot escaped as `\\.` in JSON. Without the anchors
  `ZeroAlloc.Mediator` would also match `ZeroAlloc.Mediator.Authorization`, which has its own rule.
- **Lockstep families.** `AI.Sentinel` and `Minos.NET` packages each form one group, because each
  package needs the others at the same or a later version and split PRs never restore on their own.
- **`ci-baseline` is grouped and not digest-pinned** (`pinDigests: false`), so `v1.x` releases reach
  callers through the moving `v1` tag without a PR and only a new major opens one.
- Rule descriptions in `default.json` carry the reason for the rule. When you add or change a rule,
  write or update its `description` in the same edit; the first rule of a family holds the full reason,
  later ones may stay without.

<!-- org-core:start sha256=08a88996bc9165f888fc0260607cfd0670cae128c323e16707c6b452ee66222d (generated from docker-nuc scripts/agents-md/org-core-personal.md; do not edit here) -->
## Org rules

These rules hold in every public MarcelRoozekrans repo.

### Commits and releases

- Conventional Commits. release-please reads them on the default branch, and the changelog sections
  in `release-please-config.json` decide which types count. The standard: `feat`, `fix`, `perf`,
  `deps` and `revert` open or update the release PR; `docs`, `test`, `refactor`, `build`, `ci`,
  `chore` and `style` do not, unless the commit is breaking or has a `Release-As:` footer. The
  release happens when that PR merges. A breaking change needs `!` or a `BREAKING CHANGE:` footer.
- A commit body never contains nested parentheses. release-please cannot parse them and silently
  drops the whole commit from the changelog, while the workflow still reports success.
- After merging a commit that should release, check that it appears in the release-please PR.

### Branches and pull requests

- Every change goes through a branch and a pull request. Never push to the default branch directly.
- A PR body says what the diff does, and nothing the diff does not do.
- Fix review findings in the PR under review, including small ones. File an issue only for a finding
  that belongs to other code, needs a design decision, is its own piece of work, or waits on
  something external.

### CI

- The .NET repos call the shared workflows of `MarcelRoozekrans/ci-baseline`, pinned to `@v1`, never
  `@main`.
- Its coverage gate fails the build when line coverage drops below the floor of 80%, and Codecov
  fails a PR that lowers project coverage by more than 1 percentage point. New code comes with
  tests.
- A repo with a lower floor documents the reason and links the issue that raises it.
<!-- org-core:end -->
