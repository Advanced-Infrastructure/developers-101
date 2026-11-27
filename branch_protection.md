# Branching and Promotion (backend)

**Promote features forward. Never promote environments backwards.**

Every change starts on its own branch cut from `master`, and you raise one PR: into `dev`.
From there, automation carries the change to `sit` and, if it's selected for the release, to that
release's pre-prod branch (e.g. `2.16.0-pre-prod`). Production is released only from the
pre-prod branch → `master`.

This page is the short version for developers. The source of truth is the
[Feature-Based Branching Strategy: Quick Guide](https://aitl.atlassian.net/wiki/spaces/TECH/pages/2955968521)
and its detail pages on Confluence. For commits, PR titles and descriptions, see the
[Pull Request Guidelines](./pr_guideline.md).

**Scope:** all backend (`be-*`) repositories. Frontend repositories cut their branches from the
latest `dev`, not `master`. Data team repositories keep their `DAT-<story_id>` branch names.

## Branches

| Branch | What it is | Rule |
| --- | --- | --- |
| `master` | Production | Protected. Changes arrive only from the release's pre-prod branch or `hotfix/*` (P1–P3 incidents) |
| `dev` | Development validation | `master` + features in development |
| `sit` | QA and SIT validation | `master` + features in SIT |
| `<x.y.z>-pre-prod` | Release candidate, e.g. `2.16.0-pre-prod` | Cut from `master` for each release. `master` + the features selected for it |
| `story/<TICKET>`, `bug/<TICKET>` | One Jira story or bug | Always cut from the latest `master`. Short-lived |
| `hotfix/<TICKET>` | Emergency production fix | Cut from `master`, incident process only |

`master`, `dev`, `sit` and the pre-prod branches are protected: no direct commits, no force
pushes, changes only through PRs and the automation.

## Naming

- `story/LM-12345` for a story, `bug/AI-2648` for a bug, `hotfix/LM-12345` for an emergency fix.
- Lowercase prefix, the Jira key exactly as in Jira, nothing else. **No environment segment**
  (`story/dev/...`, `bug/sit/...`) and no `feature/` prefix: one branch per ticket, whatever the
  environment.
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
  ├──► dev               you raise the PR · repository owner approves · CI passes · Squash and merge
  │                      test on Dev against every acceptance criterion → Jira: Ready for SIT
  │                      add the squash commit to Deployment Instructions → Jira: SIT-Release
  ├──► sit               automation picks up SIT-Release tickets and cherry-picks their
  │                      Deployment Instructions commit(s) (temporary, see below)
  │                      SIT QA + Product QA
  └──► 2.16.0-pre-prod   if selected for the release: the same commit(s) are cherry-picked (temporary)
          ▼
       master ──► Production
```

**You raise one PR: into `dev`.** Automation takes the change to `sit` and the pre-prod
branch, and the release owner merges the pre-prod branch into `master`. Keep your branch until the
ticket is released, in case of rework. The only PRs you raise elsewhere are:

- a `-sit-promotion` branch into `sit`, when automation reports a conflict on `sit` (see [Conflicts](#conflicts));
- a P1–P3 `hotfix/*` into `master`, then into `dev`, `sit` and the pre-prod branch to back-propagate it (see [Incidents](#incidents)).

```bash
git fetch origin
git switch master
git pull --ff-only origin master
git switch -c story/LM-12345
git push -u origin story/LM-12345
```

## Deployment Instructions (for now)

> ⚠️ **Temporary workaround.** Cherry-picking commits into `sit`, and into pre-prod for
> production, is a temporary workaround. The release automation pipeline is being changed so that
> it picks up the PRs and moves them forward on its own. Once that's live you'll only raise the
> single PR into `dev`, with no commit to add. **All developers will be notified when it goes
> live**, and this page will be updated.

Until then, the automation takes your change to `sit` and pre-prod by **cherry-picking the
commit(s) listed in the Jira ticket's Deployment Instructions**, not by promoting your branch:

1. Merge your PR into `dev` with **Squash and merge**, so the PR lands as a single commit.
2. Test on Dev against every acceptance criterion, then move the ticket to **Ready for SIT**.
   This means Dev is done, but Deployment Instructions are still to be added.
3. Copy the squash commit's URL from the merged PR into the ticket's **Deployment Instructions**.
4. Move the ticket to **SIT-Release**. The automation picks up SIT-Release tickets and
   cherry-picks their commits onto `sit`.

Example, from [LM-12259](https://aitl.atlassian.net/browse/LM-12259):

```yaml
env: [dev, sit, prod]
client: [laep, ukpn]
commit:
  - url: "https://github.com/Advanced-Infrastructure/be-dataset-service/commit/6148d6c993d6060f83640dde1afe8228aca1bb13"
```

- `env`: the environments the change goes to.
- `client`: the clients it applies to.
- `commit`: the squash commit on `dev`. Use the full commit URL, not the PR URL.
- **Rework:** a later PR into `dev` for the same ticket adds its squash commit as another `- url:` entry.

## Never do this

- Cut a branch from `dev`, `sit`, a pre-prod branch, `legacy-dev` or `legacy-sit`.
- Merge `dev`, `sit` or a pre-prod branch into your feature branch.
- Merge `dev` → `sit`, `sit` → `master`, `dev` → `master`, or any environment into another.
- Rebase or force-push a branch once it has been merged into `dev`.

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
| Not released yet | Same ticket, same branch. Push the fix, raise a new PR into `dev`, squash-merge it and add its commit to the Deployment Instructions |
| Already released | New bug ticket and a new `bug/<TICKET>` branch from the latest `master`, linked to the original |

## Incidents

- **P1–P3:** `hotfix/<TICKET>` from `master` → `master` → Production, then back-propagate the fix to
  `dev`, `sit` and the current pre-prod branch through PRs. The developer on the incident owns the
  back-propagation.
- **Everything else:** the normal flow through `dev` and `sit`. After SIT QA, automation
  updates the pre-prod branch and `master`.

## Reviews and approvals

- Every PR into `dev` needs the **repository owner's** approval (Primary, else Secondary, else
  Tertiary) and passing CI. A change across several repos needs each repo's owner.
- You own the quality of your change. Approval is a checkpoint, not a hand-over.
