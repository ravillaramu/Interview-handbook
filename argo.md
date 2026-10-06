# DevOps Engineer Interview Questions and Answers (5 Years Experience)

## Argo CD Scenario-Based Questions and Answers

### 1) Scenario: An Argo CD application is `OutOfSync` immediately after a successful sync. What do you investigate?

Answer:
- Open the application's diff and identify the exact fields that differ between Git and the live resource.
- Check whether a mutating admission webhook, operator, defaulting behavior, or another controller changes those fields.
- Confirm the desired manifest comes from the expected repository, revision, path, and Helm/Kustomize values.
- Prefer correcting the source configuration. Use a narrowly scoped `ignoreDifferences` rule only for fields intentionally owned by another controller, and document why.
- Sync again and verify the diff is resolved without hiding unrelated drift.

### 2) Scenario: A sync reports success, but the application is unhealthy and users see errors. What do you do?

Answer:
- Treat Argo CD sync status and application health as separate signals; inspect both.
- Review the affected Kubernetes resources, pod events, logs, readiness probes, and service endpoints.
- Check whether the deployment's image, configuration, secrets, or dependencies differ from the last healthy revision.
- Stop promotion or roll back to a known-good Git revision using the team's release process.
- Verify user-facing health and document the cause; add an appropriate health or rollout gate if one was missing.

### 3) Scenario: Automatic sync is enabled, but changes in Git are not reaching the cluster. How do you troubleshoot?

Answer:
- Confirm the application points to the expected revision and that the commit is present in that branch or tag.
- Check repository connectivity and credentials, manifest generation errors, and the Argo CD application conditions.
- Verify auto-sync is enabled and inspect sync policy options such as prune and self-heal.
- Check whether sync windows, resource hooks, an operation already in progress, or invalid manifests are blocking the sync.
- Resolve the specific error, then observe a controlled sync and confirm the live resources match the intended commit.

### 4) Scenario: A resource was deleted from Git, but Argo CD did not remove it from the cluster. Is this expected?

Answer:
- Check whether automated pruning is enabled; automated sync does not necessarily imply deletion of resources removed from Git.
- Inspect the sync policy and application status before enabling prune.
- Confirm the resource is managed by this application and is not protected by a resource-specific policy or finalizer.
- Enable pruning only after understanding the impact and protecting stateful or critical resources as appropriate.
- Consider using a reviewed sync or deletion workflow for high-risk resources.

### 5) Scenario: A production deployment must not happen automatically during a maintenance window. How would you control it?

Answer:
- Use Argo CD sync windows to allow or deny synchronization for selected applications and times.
- Keep production promotion controlled through protected Git branches, reviewed pull requests, and environment-specific approvals.
- Verify the effective window, timezone, application selectors, and manual override permissions.
- Test the policy in a non-production application and ensure emergency procedures are documented and audited.

### 6) Scenario: Argo CD cannot generate manifests from a Helm or Kustomize application. What do you check?

Answer:
- Read the application condition and repo-server logs for the exact generation error.
- Verify the source path, chart or overlay, tool version, values files, and referenced files exist at the selected revision.
- Check repository credentials and any required Helm repository or OCI registry access.
- Reproduce manifest rendering in CI with the same tool versions and inputs.
- Fix the source error and validate the rendered resources before syncing to a cluster.

### 7) Scenario: A team wants to deploy the same service to dev, staging, and production. How do you structure it?

Answer:
- Keep application code and deployment configuration version-controlled and make environment differences explicit.
- Use a clear promotion model, such as environment-specific overlays or controlled values, while avoiding duplicated manifests that drift.
- Promote the same immutable image digest through environments rather than rebuilding different artifacts for each stage.
- Separate application definitions and cluster access by environment, and protect production configuration with approvals.
- Track the deployed Git revision and image digest for traceability and rollback.

### 8) Scenario: Argo CD has excessive cluster permissions. How would you reduce its blast radius?

Answer:
- Inventory which clusters, namespaces, and resource types Argo CD actually needs to manage.
- Use dedicated service accounts and least-privilege Kubernetes RBAC; scope application destinations and projects.
- Restrict who can create or modify `AppProject` and `Application` resources, because those can affect destinations and sources.
- Separate high-trust production access where needed and protect credentials and the Argo CD API.
- Test permission changes against representative syncs and alert on unauthorized or unexpected changes.

### 9) Scenario: Two teams deploy conflicting resources and one team's sync keeps reverting the other's changes. How do you resolve it?

Answer:
- Identify ownership by checking which Argo CD application tracks each resource and where the manifests are maintained.
- Establish a single source of truth for each resource; avoid multiple applications managing the same object.
- Split shared platform resources from application-owned resources and define clear team boundaries.
- Review automated self-heal and prune settings before changing them; disabling reconciliation can conceal drift.
- Reconcile the manifests in Git, then verify both applications converge without fighting over the resource.

### 10) Scenario: A bad Git change has already been synchronized to production. How do you roll back?

Answer:
- Select the last known-good, immutable Git revision or release and follow the team's protected rollback procedure.
- Revert the offending change in Git so the desired state remains corrected and auditable.
- Sync the rollback and check hooks, health status, and dependencies; a sync alone does not guarantee application-level recovery.
- Verify customer-facing behavior and data integrity, especially for database or irreversible migrations.
- Record the incident and improve promotion checks or progressive delivery safeguards.
