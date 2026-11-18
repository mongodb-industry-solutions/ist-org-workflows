# ist-org-workflows

Central home for **organization-wide required workflows** used by
`mongodb-industry-solutions` org rulesets.

> ⚠️ Do not delete this repo or its workflows. They are referenced by the
> org-level **Main branch protection** ruleset and are required for pull
> requests into `main` across the whole organization.

## What's here

| Path                                               | Purpose                                                                                          |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| `.github/workflows/enforce-main-source-branch.yml` | Required workflow that enforces the gitflow rule: `main` only accepts promotions from `staging`. |

## How it's wired

The org ruleset **Main branch protection** (`16847092`) has a `workflows`
rule pointing at `.github/workflows/enforce-main-source-branch.yml` on `main`
of this repo. GitHub runs that workflow on **every pull request targeting
`main` in every repo of the org** (except the repos explicitly excluded by the
ruleset). If the workflow fails, the PR cannot be merged.

Because this repo is the **source** of a required workflow, it must:

- keep **Actions** enabled, and
- allow **Actions sharing** within the organization
  (Settings → Actions → General → Access: _Allow access to actions and
  reusable workflows from other repositories in the organization_).

Without the second setting, attaching the ruleset fails with:

```
Workflow source repository 'mongodb-industry-solutions/ist-org-workflows' has actions sharing disabled
```

The repo is **public** on purpose: required workflows from a private/internal
source repo do not run on PRs in public target repos, and this org has public
repos. Nothing sensitive lives here — the workflow is a trivial branch check.

## What the workflow does

`enforce-main-source-branch.yml` fails any pull request whose base is `main`
unless its head branch is exactly `staging`:

```yaml
on:
  pull_request:
    branches: [main]
jobs:
  source-branch:
    runs-on: ubuntu-latest
    steps:
      - name: PRs to main must originate from staging
        run: |
          if [ "${{ github.head_ref }}" != "staging" ]; then
            echo "::error::PRs to main must come from the 'staging' branch (got '${{ github.head_ref }}')."
            exit 1
          fi
```

That's the whole thing. It exists so the gitflow is enforced by GitHub rather
than by convention.

## What to do when the `source-branch` check fails

The check failing means the PR is trying to merge into `main` from a branch
that isn't `staging`. **Do not try to bypass it.** Instead:

1. Retarget the PR's base branch to `staging` (Edit → base), **or**
2. Fold the changes into a branch created off `staging`, open the PR against
   `staging`, and close the original PR (referencing it).

Then promote with the normal `staging → main` flow.

### Dependabot PRs

Dependabot opens its PRs against the repository's default branch (`main`) by
default, so **they will fail this check — that is expected**. Do not merge them
into `main`. Retarget them to `staging` (or consolidate them into a branch off
`staging`) and let them flow through the normal promotion. An alert/PR that
cannot be retargeted cleanly can be handled with the `fix-gh-security` skill in
[`ist-ai-skills`](https://github.com/mongodb-industry-solutions/ist-ai-skills).

## Changing these workflows

This repo is protected like every other repo in the org: direct pushes to
`main` are blocked and commits must be signed. Make your change on a branch and
open a pull request (an org admin can merge it; `main` requires no approvals).

> Note: creating/updating files under `.github/workflows/` through the GitHub
> API requires the `workflow` OAuth scope. Committing them via `git` does not.
