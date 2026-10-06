# DevOps Engineer Interview Questions and Answers (5 Years Experience)

## CI/CD and Jenkins Scenario-Based Questions and Answers

### CI/CD

### 1) Scenario: CI builds and deploys a container image, but Kubernetes keeps running the old version. What do you check?

Answer:
- Compare the image reference in the deployment with the image digest produced by the build.
- Check whether the pipeline pushed to the expected registry, repository, and tag, and whether the deployment references that same artifact.
- Avoid mutable tags such as `latest`; use unique version tags and, for stronger immutability, deploy by digest.
- Inspect rollout status, pod events, and the image actually running in each pod. A successful pipeline step does not prove a rollout completed.
- Confirm pull credentials and registry access, then verify the deployed commit and digest in release metadata.

### 2) Scenario: A vulnerable dependency is detected in a deployed container image. What is your response?

Answer:
- Assess the vulnerability's severity, exploitability, affected package, and whether the vulnerable code is reachable.
- Follow the organization's incident and risk process; isolate or mitigate an actively exploitable service if needed.
- Upgrade or replace the dependency, rebuild the image from source, and rerun tests and image scans.
- Promote the newly built immutable artifact through environments; do not patch a running container manually.
- Track affected image digests and deployments so the vulnerable version is replaced everywhere, then add a suitable CI policy gate.

### 3) Scenario: A pipeline passes in CI but fails after deployment. How do you improve confidence?

Answer:
- Compare the build and deployment environments, configuration, permissions, and dependency versions.
- Add integration tests and deployment smoke tests against a representative environment.
- Use health checks and progressive rollout gates, with automatic pause or rollback when service-level signals degrade.
- Promote the exact tested artifact rather than rebuilding it for each environment.
- Capture deployment metadata and logs so a failed release can be traced to its commit and artifact.

### 4) Scenario: A pipeline is slow and developers wait too long for feedback. How would you optimize it?

Answer:
- Measure duration by stage and identify the actual bottleneck before changing the workflow.
- Run independent lint, test, and security checks in parallel where resource usage permits.
- Cache dependencies safely using lockfiles or checksums, and avoid rebuilding unchanged components.
- Use path filters and incremental tests where confidence remains adequate.
- Keep a fast pull-request validation path, while retaining deeper scheduled or pre-release checks.

### 5) Scenario: The same artifact must be promoted from test to production without rebuilding. How do you design this?

Answer:
- Build once from a reviewed commit and assign an immutable version or content digest.
- Store the artifact in a controlled registry or artifact repository with provenance and scan results.
- Deploy that exact artifact to each environment, changing only environment-specific configuration.
- Require approvals and automated checks at promotion boundaries, especially for production.
- Record the commit, artifact digest, approver, and deployment result for audit and rollback.

### 6) Scenario: A production deployment introduces errors. What rollback approach do you use?

Answer:
- Pause further rollout and assess impact using service health, error rate, and latency signals.
- Roll back to the previous known-good immutable artifact or revert the source change, following the platform's safe procedure.
- Verify compatibility of database schema and other stateful changes before rollback.
- Confirm recovery from the customer perspective and communicate status.
- Perform a blameless review and add tests, progressive delivery, or compatibility checks to prevent recurrence.

### 7) Scenario: A workflow needs cloud credentials to deploy. How do you avoid long-lived secrets?

Answer:
- Prefer workload identity federation, such as GitHub Actions OIDC to an AWS IAM role, when supported.
- Restrict trust to the expected repository, branch, tag, environment, or workflow identity.
- Grant the role only the deployment permissions needed and use environment approval rules for production.
- Avoid printing credentials or sensitive environment variables in logs.
- Audit role assumptions and rotate or revoke any credentials that may have been exposed.

### 8) Scenario: Multiple commits trigger overlapping deployments to the same environment. How do you prevent race conditions?

Answer:
- Serialize deployments per environment or service using pipeline concurrency controls or a deployment lock.
- Cancel obsolete non-production runs when safe, but do not cancel an in-progress production operation without understanding its state.
- Deploy immutable artifacts and make the pipeline verify the intended revision before promotion.
- Use environment approvals and rollout health gates to prevent a later run from masking an incomplete earlier release.
- Test concurrency behavior with overlapping pipeline runs.

### Jenkins

### 9) Scenario: Jenkins controller is overloaded and builds are queued. How do you improve capacity safely?

Answer:
- Inspect queue length, executor utilization, controller CPU/memory, and build duration to find the bottleneck.
- Keep executors off the controller for production workloads and run builds on appropriately sized agents.
- Scale or provision agents dynamically where possible, and assign labels and resource limits to workload types.
- Avoid excessive parallelism that overloads shared dependencies or agents.
- Monitor controller and agent health, and test capacity changes during representative load.

### 10) Scenario: A Jenkins pipeline works on one agent but fails on another. How do you make builds reproducible?

Answer:
- Compare agent labels, operating system, tool versions, environment variables, workspace contents, and permissions.
- Use controlled agent images or configuration management and pin required tool and dependency versions.
- Run the build in a container when appropriate, while still managing host-level requirements explicitly.
- Ensure clean workspaces or use isolated workspaces to avoid stale artifacts.
- Record the agent image and toolchain versions with each build.

### 11) Scenario: A Jenkins job needs a deployment secret. How do you manage it?

Answer:
- Store the credential in Jenkins Credentials or an approved external secret manager, not in a Jenkinsfile or repository.
- Scope credentials as narrowly as possible and bind them only for the stage that needs them.
- Avoid echoing secret values; restrict access to job configuration, logs, and credential administration.
- Prefer short-lived federated credentials for cloud deployments where available.
- Rotate exposed credentials and audit usage if a secret appears in logs or source control.

### 12) Scenario: A Jenkins build is stuck waiting for an agent. How do you debug it?

Answer:
- Check the queue item's requested label and confirm a matching agent is online and has free executors.
- Review agent connection logs, cloud provisioning limits, container startup failures, and any node usage restrictions.
- Verify the pipeline's label expression and resource requirements match the available fleet.
- Check whether maintenance mode, authorization, or a disconnected node prevents scheduling.
- Add queue and agent-capacity monitoring so the issue is visible before it blocks releases.

### 13) Scenario: A Jenkins controller was compromised or is suspected to be insecure. What immediate actions do you take?

Answer:
- Follow the incident response process: restrict access, preserve evidence, and assess whether credentials or artifacts were exposed.
- Revoke or rotate credentials accessible to the controller and review recent job configuration and plugin changes.
- Inspect audit and system logs, controller access, agent activity, and unexpected builds.
- Restore from a known-good backup or rebuild from configuration as appropriate; do not assume restarting removes persistence.
- Harden access with least privilege, supported plugin versions, network restrictions, backups, and regular security updates.

### 14) Scenario: A plugin update breaks many Jenkins jobs. How do you reduce the risk of future updates?

Answer:
- Identify the plugin and dependency changes from update logs and reproduce in a non-production Jenkins environment.
- Restore service using a tested rollback or recovery plan; preserve job and controller data.
- Pin and inventory plugins, remove unused ones, and maintain compatibility documentation.
- Test controller and representative pipelines before production upgrades, with backups and a maintenance window.
- Prefer Jenkins Configuration as Code and version-controlled pipeline libraries to make recovery repeatable.

### 15) Scenario: Several teams duplicate the same Jenkins pipeline logic. How do you standardize it?

Answer:
- Extract stable common steps into a versioned Shared Library or maintained pipeline template.
- Keep application-specific stages configurable without allowing arbitrary unsafe behavior.
- Test library changes against representative pipelines and release versions deliberately.
- Document ownership, upgrade guidance, and a deprecation path.
- Migrate incrementally and preserve a way to roll back a library change.

### 16) Scenario: A Jenkins pipeline reports success, but the deployment is only partially complete. How do you prevent false success?

Answer:
- Make each deployment step fail explicitly on errors and validate the exit status of invoked tools.
- Add post-deployment checks for rollout completion, service readiness, and application-level smoke tests.
- Set timeouts and clear failure conditions for stalled operations.
- Publish deployment status only after the target system confirms the intended version is healthy.
- Test failure paths so the pipeline cannot return a success status after a partial deployment.