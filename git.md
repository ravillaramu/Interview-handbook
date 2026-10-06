# DevOps Engineer Interview Questions and Answers (5 Years Experience)

## Git, GitHub, and GitHub Actions Scenario-Based Questions and Answers

### 1) Scenario: A developer accidentally commits sensitive credentials to the repository. How do you handle it?

Answer:
- Immediately rotate the exposed secrets, API keys, passwords, and tokens.
- Remove the sensitive content from the Git history using `git filter-repo` or `git filter-branch`.
- Force-push the cleaned branch only after a coordinated team decision because it rewrites history.
- Add `.gitignore` and pre-commit hooks to prevent secrets from entering the repo again.
- Use GitHub secret scanning and enable branch protection rules.
- Review access logs and revoke any compromised credentials.

Example:
- If an AWS access key was committed, rotate the key immediately and update services to use IAM roles or GitHub secrets.

### 2) Scenario: A merge conflict occurs during a pull request. How do you resolve it safely?

Answer:
- Pull the latest branch updates and understand the conflict scope.
- Identify whether the conflict is in application code, configs, or dependency files.
- Use `git status` and `git diff` to inspect the exact changes.
- Merge carefully by keeping the correct business logic and preserving required changes from both branches.
- Test the code after resolving conflicts and verify the application builds successfully.
- Commit the resolution with a clear message and push the updated branch.

Example:
- If both branches changed the same function, compare both versions and decide which logic is correct or merge both behavior intentionally.

### 3) Scenario: A team member rebases a branch and causes history issues for others. How do you manage the situation?

Answer:
- Communicate with the team before rewriting shared history.
- Check whether the branch is already shared with others.
- If a rebase caused issues, reset the branch to the original remote state and restore the proper commit history.
- Use `git reflog` to recover lost commits.
- Educate the team to avoid rebasing shared branches and prefer merge commits for collaborative work.
- For feature branches, rebase locally before pushing, but do not rebase once others have pulled it.

Example:
- If the main branch was already used by another developer, a rebase would rewrite commit hashes and confuse the team.

### 4) Scenario: A pull request is failing CI checks. What do you do?

Answer:
- Read the failing CI job logs to identify whether the issue is build, test, lint, or security scanning.
- Reproduce the issue locally or in a similar environment using the same command as CI.
- Fix the root cause, not just the symptom.
- Push the fix and optionally rerun the workflow if supported.
- Add validations to the pipeline to catch errors earlier.

Example:
- If a lint rule fails, fix the formatting or code quality issue and rerun the GitHub Actions workflow.

### 5) Scenario: You need to revert a bad production deployment that was already merged. How do you handle it?

Answer:
- Identify the exact commit or PR that introduced the bad change.
- Use `git revert <commit_sha>` for a safe, non-destructive rollback.
- If the change is already merged and released, create a new commit that reverses the impact.
- Validate the rollback in staging or a lower environment before production.
- Communicate the rollback clearly to stakeholders and document root cause.

Example:
- A bad config change is reverted by a new commit instead of a hard reset, preserving history and avoiding destructive behavior.

### 6) Scenario: A feature branch is stale and behind the main branch. How do you update it safely?

Answer:
- Fetch the latest changes from the remote using `git fetch origin`.
- Rebase the feature branch onto `main` using `git rebase origin/main` or merge `main` into the branch if the team prefers merge strategy.
- Resolve conflicts carefully and run validation tests.
- Force-push only if the branch was rebased and the team is aligned on the workflow.
- Ensure the branch is clean and CI passes before merging.

Example:
- A developer rebases before opening a PR to keep linear history and reduce merge conflicts.

### 7) Scenario: A developer accidentally deletes a branch that still has needed work. How do you recover it?

Answer:
- Check whether the branch still exists remotely or in the reflog.
- Use `git reflog` to locate the lost commit hash.
- Recover the branch using `git checkout -b <branch-name> <commit-sha>` or `git branch <branch-name> <commit-sha>`.
- If the branch was deleted on GitHub, restore it from the remote if the server supports restore or from a local reflog backup.
- Add a policy to protect important branches and reduce accidental deletions.

Example:
- A developer deletes a feature branch after merging, then discovers a needed hotfix was not included. Git reflog can restore the lost commit.

### 8) Scenario: A team wants to enforce quality gates before merging to main. How do you design it?

Answer:
- Enable branch protection rules in GitHub.
- Require pull request reviews before merging.
- Require status checks like lint, test, security scan, and build to pass.
- Block direct pushes to the main branch.
- Require signed commits for higher security if needed.
- Use GitHub Actions to enforce CI and security policies automatically.

Example:
- Main branch requires at least 2 approvals and a green build before merge.

### 9) Scenario: A GitHub Actions pipeline is flaky and sometimes fails randomly. What do you check?

Answer:
- Review logs for timeouts, resource limits, or rate-limits.
- Check whether the workflow depends on external services or network access.
- Use caching for dependencies to reduce runtime and instability.
- Validate environment variables, secrets, and matrix strategies.
- Confirm runner images are updated and consistent across jobs.
- Add retries for non-deterministic external steps when appropriate.

Example:
- A workflow fails because a package registry rate-limits requests. Using a cached dependency layer or a private registry reduces instability.

### 10) Scenario: A release pipeline is deploying the wrong artifact version. How do you fix it?

Answer:
- Verify the Git tag, GitHub release version, and CI artifact version align.
- Check workflow variables and environment configuration.
- Ensure the build is triggered by the correct branch or tag.
- Validate artifact provenance and checksum before deployment.
- Use immutable build artifacts and versioning tags to avoid confusion.
- Add manual approval gates for production rollout.

Example:
- A pipeline accidentally deploys the latest branch build instead of the intended release tag. The fix is to pin artifacts and enforce tag-based triggers.

### 11) Scenario: Developers are creating too many merge commits in a busy repository. How do you improve Git hygiene?

Answer:
- Standardize a branching strategy like GitFlow, trunk-based development, or feature branching.
- Encourage rebasing feature branches before merging when appropriate.
- Use PRs to review and squash commits when required.
- Configure merge commit policy based on team needs.
- Use squash merges to keep the main branch history clean and readable.

Example:
- A team prefers squash and merge for PRs to keep main branch history simple and easy to track.

### 12) Scenario: The team needs to audit who changed what in the repository. How do you ensure traceability?

Answer:
- Use clear commit messages with issue or ticket IDs.
- Require PR descriptions and review comments for context.
- Enforce branch protection and code review policies.
- Use GitHub Insights, commit history, and tags for audit trails.
- Keep release notes aligned with GitHub releases and tags.
- Use signed commits where security policy requires it.

Example:
- Every commit includes the Jira ticket ID such as `DEVOPS-184` to trace the change to an issue.

### 13) Scenario: A new GitHub Action is failing because a secret is not available in a workflow job. How do you diagnose it?

Answer:
- Confirm the secret is defined in the repository, organization, or environment settings.
- Check whether the workflow is running on the correct branch or environment.
- Ensure the secret is referenced correctly in YAML and scope matches the job context.
- Verify that `secrets` are not exposed to pull requests when not intended.
- Check environment protection rules and access permissions.

Example:
- A production deployment job fails because the secret is defined only for `main`, but the workflow is running from a feature branch.

### 14) Scenario: You need to manage many repositories with consistent CI/CD standards. How do you do it?

Answer:
- Create reusable GitHub Actions workflows with organization-level templates.
- Standardize linting, testing, and security scanning across repositories.
- Use templates or reusable workflows to maintain consistency.
- Add central governance policies for branch protection and required checks.
- Document repository standards and onboarding steps for new teams.

Example:
- An organization uses a shared workflow called `ci.yml` for all Node.js services, ensuring consistent build and test behavior.

### 15) Scenario: The team wants to keep GitHub Actions cost-efficient. What strategy would you recommend?

Answer:
- Use self-hosted runners for predictable workloads or large builds.
- Cache dependencies and build artifacts to reduce repeated work.
- Run only necessary jobs on pull requests and avoid duplicate pipelines.
- Use matrix strategies carefully and restrict triggers to relevant branches.
- Delete unused artifacts and set workflow concurrency to prevent redundant jobs.
- Use path filters to trigger workflows only when relevant files change.

Example:
- A workflow only runs for application directories, not for docs-only changes.

