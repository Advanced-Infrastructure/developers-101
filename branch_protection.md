# GitHub Branching and PR Guidelines

## 1. Branching Rules

### 1.1 General Branching Convention
- The `master` branch is protected. No direct pushes are allowed.
- All changes must be made through feature or bug branches and merged via Pull Requests (PRs).
- The `dev` branch is the main base branch for development work.

### 1.2 Team-Specific Branch Naming Conventions
- **Platform Team**: Use `/story/<story_id>` or `/bug/<bug_id>`
- **Data Team**: Use `/DAT-<story_id>`
- `<story_id>` and `<bug_id>` correspond to Jira tickets.
- For branches which are from SIT or PROD (as quick fix) use `/story/sit/<story_id>` or `bug/prod/<story_id>` i.e. include the environment name in the branch name.

## 2. Pull Request (PR) Guidelines

### 2.1 PR Requirements
- All PRs must have a clear description explaining the changes.
- Reference the Jira ticket ID in the PR title or description.
- Follow the coding standards of the project.
- PRs should be small and focused. Read more about writing effective PRs [here] ('./pr_guideline.md')
- PRs should have appropriate labels for tracking purposes.

### 2.2 Code Review Process
- Developers cannot push directly to `master`. A review is required.
- The reviewer is assigned based on the team:
  - **Platform Team:** A senior developer from the Platform team reviews the PR.
  - **Data Team:** A senior developer from the Data team reviews the PR.
- All code must be reviewed and approved before merging.

## 3. Additional GitHub Etiquette

- **Commit Messages**: Use meaningful commit messages following the format: `[Jira_ID] Brief description of change`
- **Rebasing**: Keep your branch updated with `dev` by regularly rebasing.
- **Merging**: Use squash and merge to keep a clean commit history.
- **Conflict Resolution**: Ensure conflicts are resolved before requesting a review.

By following these guidelines, we ensure a smooth and structured development workflow, reducing merge conflicts and improving code quality.
