# Contributing

Thank you for helping. Each repository describes its development setup in
`docs/contributing.md`; this file covers what all chronocratic repositories
share.

## Branches

- Open pull requests against `dev`. Pull requests to `main` come only from
  `dev` and are merged by fast-forward (`/fast-forward`) by a maintainer.
- Title pull requests with Conventional Commits, for example
  `feat(models): add TS2Vec encoder` or `fix: seed random crops`.

## Changelog

Every pull request to `dev` needs a towncrier fragment in `changelog.d/`.
For pull requests from a branch in the same repository, CI writes it from the
title and the `## TL;DR` section of the description. From a fork, add it by
hand. Set the type with a `changelog:<type>` label (`added`, `fixed`,
`changed`, `removed`, `deprecated`, `security`) or opt out with
`skip-changelog`.

## Protected files

Changes to `pyproject.toml`, `ruff.toml`, `.pre-commit-config.yaml` and
`.vscode/` get a maintainer review comment. Explain why in the description.

## Agent and planning files

Pull requests to `main` must not contain `.claude/`, `.remember/`,
`.planning/`, `CLAUDE.md` or similar local agent files. Keep them out of
commits.
