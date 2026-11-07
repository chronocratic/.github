<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://shieldcn.dev/header/surface.svg?title=CHRONOCRATIC&amp;subtitle=Core+infrastructure+and+tools+for+time+series+research&amp;align=center&amp;font=jetbrains-mono&amp;width=1200&amp;height=600&amp;bg=0E2A44&amp;titleColor=F3F1EC&amp;subtitleColor=A8D5D1&amp;mode=dark&amp;logo=https%3A%2F%2Fraw.githubusercontent.com%2Fchronocratic%2F.github%2Fmain%2Fassets%2Fmark-round-dark.png" />
    <img alt="Chronocratic: core infrastructure and tools for time series research" src="https://shieldcn.dev/header/surface.svg?title=CHRONOCRATIC&amp;subtitle=Core+infrastructure+and+tools+for+time+series+research&amp;align=center&amp;font=jetbrains-mono&amp;width=1200&amp;height=600&amp;bg=F3F1EC&amp;titleColor=0E2A44&amp;subtitleColor=2A6A7C&amp;mode=light&amp;logo=https%3A%2F%2Fraw.githubusercontent.com%2Fchronocratic%2F.github%2Fmain%2Fassets%2Fmark-round.png" />
  </picture>
</p>

Shared GitHub configuration for the chronocratic organization.

- `.github/workflows/`: reusable workflows (`on: workflow_call`). Each
  repository keeps a short caller that owns the trigger.
- `workflow-templates/`: those callers, offered under **Actions > New workflow**
  in every organization repository.
- `CONTRIBUTING.md`, `SECURITY.md`, issue forms and the
  pull request template: defaults for repositories that do not define their own.
- `profile/README.md`: the organization profile page.

## Using a workflow

```yaml
jobs:
  ci:
    uses: chronocratic/.github/.github/workflows/build-and-test.yml@v0.1.0
    with:
      package: chronocratic/models
```

Pin an exact release tag. Dependabot in each repository opens a PR when a
newer tag is published (see below).

| Workflow | Caller trigger | Inputs | Needs in the calling repository |
| --- | --- | --- | --- |
| `build-and-test.yml` | push to main, PRs to main and dev | `package`, `python-version`, `docs` | `tests/`, `docs/`, `ty` and `docs` group |
| `auto-changelog-fragment.yml` | PRs to dev | `python-version` | towncrier, `changelog.d/`; `contents: write` |
| `release-prep.yml` | `workflow_dispatch` | `version`, `python-version` | `ff_merge` environment with `FF_MERGE_TOKEN` |
| `release-notes.yml` | release published | none | `CHANGELOG.md` on main; `contents: write` |
| `python-build.yml` | release published | `python-version` | upload job stays in the caller, see below |
| `docs-publish.yml` | release published | `docs-url` | `rtd` environment with `RTD_WEBHOOK_URL`, `RTD_WEBHOOK_TOKEN` |
| `ff-merge-check.yml` | issue comment | none | none |
| `ff-merge-do.yml` | issue comment | none | `ff_merge` environment with `FF_MERGE_TOKEN` |
| `main-pre-merge-gate.yml` | PRs to main | `forbidden` | none |
| `protected-files-check.yml` | `pull_request_target` | `contributing-doc` | `pull-requests: write` |

Notes:

- A reusable workflow gets at most the permissions of the calling job. The
  templates grant what each workflow needs.
- Environment secrets resolve in the reusable job that names the environment.
  Callers pass `secrets: inherit` and keep the environments and secrets in
  their own repository settings.
- PyPI trusted publishing does not support uploading from a reusable
  workflow. `python-build.yml` builds `dist`; the upload job in
  `workflow-templates/pypi-publish.yml` stays in each repository, and that
  file is the trusted publisher registered on PyPI.
- The branch flow is fixed: feature branches merge into `dev`, and `dev`
  fast-forwards into `main`.

## Releasing this repository

Releases are fixed semver tags, never moved:

- patch (`v0.1.1`): bug fix
- minor (`v0.2.0`): new optional input or feature
- major (`v1.0.0`, then `v2.0.0`): breaking change to inputs, secrets or behavior; while on `0.x`, a minor bump may break

Merge to `main`, then tag:

```bash
git tag v0.2.0
git push origin v0.2.0
```

## Keeping callers current

Add `.github/dependabot.yml` to each repository. Dependabot then opens a PR
that bumps `@vX.Y.Z` in the callers when a new tag appears, and keeps the
other pinned actions current too:

```yaml
version: 2
updates:
  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: weekly
```
