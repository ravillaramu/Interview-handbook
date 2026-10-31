# Git Scenario-Based Questions and Answers

## Git and GitHub Scenario-Based Questions and Answers

### 1) A developer accidentally commits sensitive credentials to the repository. How do you handle it?

```text
Answer:
- Immediately rotate the exposed secrets, API keys, passwords, and tokens.
- Remove the sensitive content from the Git history using `git filter-repo` or `git filter-branch`.
- Force-push the cleaned branch only after a coordinated team decision because it rewrites history.
- Add `.gitignore` and pre-commit hooks to prevent secrets from entering the repo again.
- Use GitHub secret scanning and enable branch protection rules.
- Review access logs and revoke any compromised credentials.

Example:
- If an AWS access key was committed, rotate the key immediately and update services to use IAM roles or GitHub secrets.
```

### 2) A merge conflict occurs during a pull request. How do you resolve it safely?

```text
Answer:
- Pull the latest branch updates and understand the conflict scope.
- Identify whether the conflict is in application code, configs, or dependency files.
- Use `git status` and `git diff` to inspect the exact changes.
- Merge carefully by keeping the correct business logic and preserving required changes from both branches.
- Test the code after resolving conflicts and verify the application builds successfully.
- Commit the resolution with a clear message and push the updated branch.

Example:
- If both branches changed the same function, compare both versions and decide which logic is correct or merge both behavior intentionally.
```

### 3) A team member rebases a branch and causes history issues for others. How do you manage the situation?

```text
Answer:
- Communicate with the team before rewriting shared history.
- Check whether the branch is already shared with others.
- If a rebase caused issues, reset the branch to the original remote state and restore the proper commit history.
- Use `git reflog` to recover lost commits.
- Educate the team to avoid rebasing shared branches and prefer merge commits for collaborative work.
- For feature branches, rebase locally before pushing, but do not rebase once others have pulled it.

Example:
- If the main branch was already used by another developer, a rebase would rewrite commit hashes and confuse the team.
```

### 4) A pull request is failing CI checks. What do you do?

```text
Answer:
- Read the failing CI job logs to identify whether the issue is build, test, lint, or security scanning.
- Reproduce the issue locally or in a similar environment using the same command as CI.
- Fix the root cause, not just the symptom.
- Push the fix and optionally rerun the workflow if supported.
- Add validations to the pipeline to catch errors earlier.

Example:
- If a lint rule fails, fix the formatting or code quality issue and rerun the GitHub Actions workflow.
```

### 5) You need to revert a bad production deployment that was already merged. How do you handle it?

```text
Answer:
- Identify the exact commit or PR that introduced the bad change.
- Use `git revert <commit_sha>` for a safe, non-destructive rollback.
- If the change is already merged and released, create a new commit that reverses the impact.
- Validate the rollback in staging or a lower environment before production.
- Communicate the rollback clearly to stakeholders and document root cause.

Example:
- A bad config change is reverted by a new commit instead of a hard reset, preserving history and avoiding destructive behavior.
```

### 6) A feature branch is stale and behind the main branch. How do you update it safely?

```text
Answer:
- Fetch the latest changes from the remote using `git fetch origin`.
- Rebase the feature branch onto `main` using `git rebase origin/main` or merge `main` into the branch if the team prefers merge strategy.
- Resolve conflicts carefully and run validation tests.
- Force-push only if the branch was rebased and the team is aligned on the workflow.
- Ensure the branch is clean and CI passes before merging.

Example:
- A developer rebases before opening a PR to keep linear history and reduce merge conflicts.
```

### 7) A developer accidentally deletes a branch that still has needed work. How do you recover it?

```text
Answer:
- Check whether the branch still exists remotely or in the reflog.
- Use `git reflog` to locate the lost commit hash.
- Recover the branch using `git checkout -b <branch-name> <commit-sha>` or `git branch <branch-name> <commit-sha>`.
- If the branch was deleted on GitHub, restore it from the remote if the server supports restore or from a local reflog backup.
- Add a policy to protect important branches and reduce accidental deletions.

Example:
- A developer deletes a feature branch after merging, then discovers a needed hotfix was not included. Git reflog can restore the lost commit.
```

### 8) Developers are creating too many merge commits in a busy repository. How do you improve Git hygiene?

```text
Answer:
- Standardize a branching strategy like GitFlow, trunk-based development, or feature branching.
- Encourage rebasing feature branches before merging when appropriate.
- Use PRs to review and squash commits when required.
- Configure merge commit policy based on team needs.
- Use squash merges to keep the main branch history clean and readable.

Example:
- A team prefers squash and merge for PRs to keep main branch history simple and easy to track.
```

### 9) The team needs to audit who changed what in the repository. How do you ensure traceability?

```text
Answer:
- Use clear commit messages with issue or ticket IDs.
- Require PR descriptions and review comments for context.
- Enforce branch protection and code review policies.
- Use GitHub Insights, commit history, and tags for audit trails.
- Keep release notes aligned with GitHub releases and tags.
- Use signed commits where security policy requires it.

Example:
- Every commit includes the Jira ticket ID such as `DEVOPS-184` to trace the change to an issue.
```

### 10) Your team follows a strict commit message convention. How do you ensure all contributors follow it automatically?

### 11) A developer made a force push that overwrote shared history. How do you mitigate and recover from the situation?

### 12) A tag was mistakenly created on the wrong commit and pushed to the remote. How do you fix and retag properly?

### 13) You accidentally pushed to the `main` branch instead of your feature branch. What do you do to fix it?

### 14) Youre mentoring a junior developer who accidentally committed sensitive files. How do you walk them through cleaning it up using Git?

### 15) You accidentally deleted a local branch that hadnt been pushed yet. How would you recover it?

### 16) How do you sign commits in Git? What is the use of that?

### 17) What is the difference between `git pull` and `git fetch`?

### 18) What is the difference between `git reset` and `git revert`?

### 19) How do you stash changes in Git?

### 20) What is the purpose of the `.gitignore` file?

### 21) What is the difference between `git diff` and `git difftool`?
