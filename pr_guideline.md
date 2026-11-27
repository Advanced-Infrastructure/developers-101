# Pull requests

How we write commits and pull requests in the backend (`be-*`) repositories. Where branches
come from and where PRs go are covered in [Branching and Promotion](./branch_protection.md).

## The rules at a glance

Follow these on every PR into `dev`, `sit`, `pre-prod` or `master`. They're being rolled out as
an automatic check (see [How the rules are checked](#how-the-rules-are-checked)).

| What | Rule | Example |
| --- | --- | --- |
| Branch | `story/<TICKET>`, `bug/<TICKET>` or `hotfix/<TICKET>`, cut from the latest `master` | `story/LM-12345` |
| PR target | Your PR goes into `dev`. Automation raises the PRs into `sit` and beyond | `story/LM-12345` → `dev` |
| PR title | `<TICKET>: <summary>`, same ticket as the branch, no environment prefix | `LM-12345: Add export filter for datasets` |
| PR description | Jira link, what changed and why, how it was tested, impact | See the [template](#pr-description) |
| Commit message | `<TICKET>: <imperative summary>` | `LM-12345: Fix null region in cost query` |
| Branch history | Only this ticket's commits on top of `master`. Nothing from `dev`, `sit` or another ticket | |

- [Write commits as `<TICKET>: <imperative summary>`](#commit-messages)
- [Do only one thing with each commit](#do-only-one-thing-with-each-commit)
- [Don't mix refactoring with functional changes in the same commit](#dont-mix-refactoring-with-functional-changes)
- [Make each commit atomic](#atomic-commits)
- [Title and describe the PR properly](#pr-name-rule)
- [Keep pull requests small](#keep-prs-small)
- [Leave pull requests in draft until they are potentially ready to merge](#leave-in-draft)
- [Don't request a review until tests have passed](#dont-request-review-until-tests-passed)
- [Try not to surprise reviewers with urgent, complex pull requests](#dont-surprise-with-urgent-prs)
- [Acknowledge every review comment](#acknowledge-comments)
- [When re-requesting reviews, make it clear what has changed](#make-clear-what-has-changed-since-last-review)
- [Ensure you can be identified from your GitHub account](#ensure-you-can-be-identified)
- [How the rules are checked](#how-the-rules-are-checked)

## Commits

### <a name="commit-messages"></a><a name="imperative-commit-subjects"></a>Write commits as `<TICKET>: <imperative summary>`

Start every commit subject with the Jira key of the branch's ticket, a colon and a space, then
a short summary in the imperative mood:

```text
LM-12345: Add export filter for datasets        ✓
LM-12345: Added export filter                   ✗ past tense
Fixes export filter                             ✗ no ticket
[LM-12345] add export filter / LM-12345 || ...  ✗ old formats
```

Imperative means "Fix bug", not "Fixed bug" or "Fixes bug". A good subject completes the
sentence "If merged, this commit will…". This also matches the messages Git writes itself for
`git merge` and `git revert`. Merge commits (e.g. merging `origin/master` into your branch)
don't need the ticket prefix.

### <a name="do-only-one-thing-with-each-commit"></a>Do only one thing with each commit

Express a single thought with each commit. That can be a very small commit (a typo fix) or a
large one (moving many modules to another package). A reviewer should be able to hold the commit
in their head. If you're writing "change this and that", split the commit.

### <a name="dont-mix-refactoring-with-functional-changes"></a>Don't mix refactoring with functional changes in the same commit

Commits that both restructure code and change behaviour are hard to review. Put the refactoring
in its own commit.

### <a name="atomic-commits"></a>Make each commit atomic

After each commit the tests should pass and the code should be safe to deploy. Don't
half-implement something in one commit and fix it in the next. To land a failing test before its
fix, mark it `pytest.mark.xfail`.

## Pull requests

### <a name="pr-name-rule"></a><a name="pr-description"></a>Title and describe the PR properly

**Title:** `<TICKET>: <summary>`, using the same ticket as the branch. Don't add `DEV:`, `SIT:`
or `PROD:`: one branch goes to every environment, and the target branch already says which one.

**Description:** use this template. Keep it short, but don't leave anything out.

```markdown
## Jira
https://aitl.atlassian.net/browse/LM-12345

## What and why
What this PR changes, and the reason for it.

## How it was tested
Local run (LocalStack, local DBs), unit and integration tests, SQL checked against the
master/dev/SIT dumps where the database is touched.

## Impact
Database changes or migrations, config or environment variables, deployment steps,
other services affected. "None" is a valid answer.
```

### <a name="keep-prs-small"></a>Keep pull requests small

Big PRs are hard to review, attract endless comments, conflict more and are riskier to deploy.
If a PR runs past a few hundred lines, split it, for example into separate tickets, or into
sub-branches of the ticket branch (see [Branching and Promotion](./branch_protection.md)).
It's never too late to break up a PR that has become hard to review.

### <a name="leave-in-draft"></a>Leave pull requests in draft until they are potentially ready to merge

Open PRs as [drafts](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests#draft-pull-requests)
so the repository owner isn't tagged too early. Mark them "ready for review" once you believe
they could go to production. Asking for early feedback on a draft is fine.

### <a name="dont-request-review-until-tests-passed"></a>Don't request a review until tests have passed

Before asking for a review:

- Run the unit and integration tests locally.
- Run `ruff check` and the pre-commit hooks. Setup is in [Setting up Ruff](./enforcing_ruff.md).
- Make sure CI and the PR guardrails check pass.

Exceptions: failures that already exist on `master` and are unrelated to your change, or when you
need the reviewer's help with the failure. In both cases keep the PR in draft if it isn't
production-ready, and say why the tests fail in the description.

### <a name="dont-surprise-with-urgent-prs"></a>Try not to surprise reviewers with urgent, complex pull requests

If a time-sensitive PR is coming, tell the repository owner ahead of time. The more complex the
change, the more notice they need.

## Reviews

### <a name="acknowledge-comments"></a>Acknowledge every review comment

Respond to every comment so the reviewer knows you've seen it. Often a 👍 is enough.

### <a name="make-clear-what-has-changed-since-last-review"></a>When re-requesting reviews, make it clear what has changed

Say what changed since the last review, so the reviewer doesn't reread everything:

- **Before the branch has been merged into `dev`:** you may amend or fixup commits, but leave a
  PR comment listing what changed.
- **Once any of its commits are in `dev`:** don't rewrite history (no rebase, amend or
  force-push), because that causes duplicate commits and false conflicts on the next promotion.
  Add new commits, and tell the reviewer which commit to review from.

## <a name="ensure-you-can-be-identified"></a>Ensure you can be identified from your GitHub account

Show your full name on the GitHub account you use for work, so commits and PRs can be traced to
a person. If you'd rather not show it on a personal profile, create a separate work account.

## <a name="how-the-rules-are-checked"></a>How the rules are checked

These checks are being rolled out. Follow the rules now; the checks will catch anything missed.

- **On every PR:** a `pr-guardrails` check will validate the branch name, target, title,
  description, commit messages and branch history, and list anything to fix in the check's
  summary. At first it only warns. Later, rule errors will block the merge.
- **Regular audits:** recent PRs are reviewed against these rules. A PR that breaks one may get
  a comment saying what to change. Fix the title and description in place. If the branch itself
  is wrong, recreate it from `master` and raise a new PR.

Questions about a rule or a failing check: ask the repository owner or DevOps.
