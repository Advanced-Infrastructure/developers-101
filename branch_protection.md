# Branching and Promotion (backend)

**Promote features forward. Never promote environments backwards.**

Every change starts on its own branch cut from `master`. That same branch is promoted to `dev`,
then `sit`, then `pre-prod`. Production is released only through `pre-prod` → `master`.

This page is the short version for developers. The source of truth is the
[Feature-Based Branching Strategy: Quick Guide](https://aitl.atlassian.net/wiki/spaces/TECH/pages/2955968521)
and its detail pages on Confluence. For commits, PR titles and descriptions, see the
[Pull Request Guidelines](./pr_guideline.md).

**Scope:** all backend (`be-*`) repositories. Frontend repositories cut their branches from the
latest `dev`, not `master`. Data team repositories keep their `DAT-<story_id>` branch names.

## Branches

| Branch | What it is | Rule |
| --- | --- | --- |
| `master` | Production | Protected. Changes arrive only from `pre-prod` (releases) or `hotfix/*` (P1–P3 incidents) |
| `dev` | Development validation | `master` + features in development |
| `sit` | QA and SIT validation | `master` + features in SIT |
| `pre-prod` | Next release candidate | Recreated from `master` after every release |
| `story/<TICKET>`, `bug/<TICKET>` | One Jira story or bug | Always cut from the latest `master`. Short-lived |
| `hotfix/<TICKET>` | Emergency production fix | Cut from `master`, incident process only |

`master`, `dev`, `sit` and `pre-prod` are protected: no direct commits, no force pushes,
changes only through PRs and the automation.

## Naming

- `story/LM-12345` for a story, `bug/AI-2648` for a bug, `hotfix/LM-12345` for an emergency fix.
- Lowercase prefix, the Jira key exactly as in Jira, nothing else. **No environment segment**
  (`story/dev/...`, `bug/sit/...`) and no `feature/` prefix: the same branch goes to every environment.
- Allowed suffixes, only for these cases:
  - `story/LM-12345-api`: a sub-branch when several developers share one ticket. Its PR goes
    into `story/LM-12345`, never into an environment branch.
  - `story/LM-12345-dev-promotion` / `-sit-promotion`: a disposable branch for resolving a
    conflict with `dev` or `sit` (see below).

## The flow

```text
master
  │  git switch -c story/LM-12345
  ▼
story/LM-12345 ── build and test locally (LocalStack, local DBs, SQL validation)
  │
  ├──► dev        you raise the PR · repository owner approves · CI passes
  │               test on Dev against every acceptance criterion → Jira: Release
  ├──► sit        automation raises and merges the PR when the ticket moves to Release
  │               SIT QA + Product QA
  └──► pre-prod   only if selected for the release
          ▼
       master ──► Production
```

**You raise one PR: into `dev`.** Automation raises the PRs into `sit` and `pre-prod`, and the
release owner handles `pre-prod` → `master`. Keep the branch after the `dev` merge, because it's
promoted again later. The only PRs you raise elsewhere are:

- a `-sit-promotion` branch into `sit`, when the automated SIT PR hits a conflict (see [Conflicts](#conflicts));
- a P1–P3 `hotfix/*` into `master`, then into `dev`, `sit` and `pre-prod` to back-propagate it (see [Incidents](#incidents)).

```bash
git fetch origin
git switch master
git pull --ff-only origin master
git switch -c story/LM-12345
git push -u origin story/LM-12345
```

## Never do this

- Cut a branch from `dev`, `sit`, `pre-prod`, `legacy-dev` or `legacy-sit`.
- Merge `dev`, `sit` or `pre-prod` into your feature branch.
- Merge `dev` → `sit`, `sit` → `master`, `dev` → `master`, or any environment into another.
- Rebase or force-push a branch once any of its commits are in `dev`.

The PR check being rolled out looks at a branch's history, not just its name: a branch carrying
commits from `dev` or `sit` that aren't in `master` will fail it.

## Conflicts

| Your branch conflicts with | Do this |
| --- | --- |
| `master`, or a change already in production | `git merge origin/master` into your branch, test, push |
| A feature in `dev` or `sit` that isn't in production yet | Resolve on a disposable branch: `git switch -c story/LM-12345-dev-promotion origin/story/LM-12345`, `git merge origin/dev`, resolve, push, raise the PR from it into `dev`. Never merge it back into your original branch |
| Unsure | Stop and ask the repository owner or DevOps. Don't force anything |

## Rework

| The change is… | Do this |
| --- | --- |
| Not released yet | Same ticket, same branch. Push the fix and raise a new PR into `dev` |
| Already released | New bug ticket and a new `bug/<TICKET>` branch from the latest `master`, linked to the original |

## Incidents

- **P1–P3:** `hotfix/<TICKET>` from `master` → `master` → Production, then back-propagate the fix to
  `dev`, `sit` and `pre-prod` through PRs. The developer on the incident owns the back-propagation.
- **Everything else:** the normal flow through `dev` and `sit`. After SIT QA, automation
  updates `pre-prod` and `master`.

## Reviews and approvals

- Every PR into `dev` needs the **repository owner's** approval (Primary, else Secondary, else
  Tertiary) and passing CI. A change across several repos needs each repo's owner.
- You own the quality of your change. Approval is a checkpoint, not a hand-over.
