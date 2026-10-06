# DevOps Engineer Interview Questions and Answers (5 Years Experience)

## Terraform Scenario-Based Questions and Answers

### 1) Scenario: An automated `terraform apply` times out partway through and may have created some resources. How do you recover safely?

Answer:
- Stop other applies against the same state and inspect the CI logs and provider error to determine which operations completed. Terraform normally records successful operations as it proceeds, so do not assume the whole apply was rolled back.
- Run `terraform plan` with the same configuration, workspace, backend, and variables to compare configuration, state, and real infrastructure. Do not immediately run `destroy` or blindly repeat a broad apply.
- If state and infrastructure are consistent, correct the underlying API/network issue and apply the reviewed plan.
- If a resource exists but is absent from state, import it using an `import` block or `terraform import` after confirming its correct address and configuration. If state tracks a resource that no longer exists, refresh/plan and reconcile intentionally.
- Use `-target` only as a documented, temporary recovery measure when a specific dependency must be repaired; then run a full plan.
- Check for partially configured cloud resources and validate application health after recovery.

### 2) Scenario: A remote state file is corrupted or accidentally deleted. How do you recover?

Answer:
- Freeze Terraform runs for that state and preserve the damaged object and logs for investigation.
- For an S3 backend, check bucket versioning and restore the correct prior object version using the approved recovery procedure. Confirm the backend key, workspace, lineage, and serial are appropriate before resuming.
- For other backends, use their supported state history, snapshots, or backups. State locking prevents concurrent writers; it does not replace backups.
- Run `terraform state pull` where possible to save and inspect the current state, then validate the restored state with `terraform plan` before any apply.
- If no usable backup exists, inventory real resources and import them into the correct configuration/state incrementally. Use `terraform plan -refresh-only` to review observed drift; it does not reconstruct missing resource bindings by itself.
- Never replace a shared state file while an apply is active, and protect state and plan files as sensitive data.

### 3) Scenario: How would you organize Terraform code and state for dev, staging, and production in a large organization?

Answer:
- Keep reusable modules separate from environment-specific root configurations, inputs, and backend keys.
- Isolate production state and access from non-production state to reduce blast radius and contention; use separate state boundaries aligned to ownership and lifecycle.
- Workspaces can be useful for similar instances of one configuration, but separate root configurations/backends are often clearer when environments need different access controls, settings, or release schedules.
- Use a consistent promotion and review process, with environment-specific role permissions and a clear path for module version upgrades.
- Pass only stable, necessary outputs between states, or use a service registry/parameter store for shared interfaces. Avoid tightly coupling many stacks through broad `terraform_remote_state` access.

### 4) Scenario: A monolithic Terraform configuration manages over 1,000 resources and takes 20 minutes to plan. How do you refactor it?

Answer:
- First measure provider/API bottlenecks and identify resources with distinct lifecycle or ownership; do not split solely by arbitrary resource counts.
- Break the configuration into cohesive stacks such as networking, shared data services, and application platforms, each with its own state and deployment ownership.
- Define explicit, stable interfaces between stacks. Prefer published outputs or a controlled parameter store; use `terraform_remote_state` only when consumers are authorized to read the entire state object.
- Move resources without recreating them: use Terraform `moved` blocks (Terraform 1.1+) for address/module refactors, or carefully reviewed `terraform state mv` where needed.
- Refactor incrementally: back up state, review a plan with no unintended create/destroy actions, and apply one boundary at a time.
- Measure plan time and operational complexity after each split; more states add orchestration and dependency management costs.

### 5) Scenario: How do you design a reliable Terraform CI/CD pipeline and handle concurrent runs?

Answer:
- On pull requests, run formatting, initialization, validation, security/policy checks, and a saved plan using the proposed code and controlled credentials.
- Require review of the plan; apply only from a trusted, protected branch or approved release process, using the reviewed plan where the workflow supports it.
- Use remote state with locking and least-privilege credentials. Locking serializes writers to the same state; it does not prevent conflicting changes across different states.
- Add concurrency controls per environment/state, short-lived identity federation where available, and approval gates for production.
- Store plan artifacts securely and briefly because plans can contain sensitive values; never expose them to untrusted pull-request code.
- Record commit, plan, approver, and apply result, and test the pipeline's failure and recovery paths.

### 6) Scenario: How do you manage database passwords and API keys used by Terraform?

Answer:
- Do not hardcode secrets or commit secret-bearing `.tfvars`, state, or plan files.
- Prefer workload identity and provider roles so Terraform does not need static cloud access keys.
- Retrieve application secrets from an approved secret manager and pass them to the workload through its runtime secret mechanism when possible, rather than embedding secret values in Terraform-managed resources unnecessarily.
- `sensitive = true` redacts values in many CLI displays; it does not encrypt or prevent values from being stored in state or plan files.
- Restrict access and encryption for backend state, CI logs, plan artifacts, and secret-manager policies. Use ephemeral variables/resources only where supported by the required Terraform and provider versions.
- Rotate exposed credentials and audit their usage.

### 7) Scenario: How do you prevent developers from provisioning insecure or excessively expensive resources?

Answer:
- Apply preventive controls with policy as code, such as OPA/Conftest or Sentinel, against the Terraform plan before apply.
- Enforce organization standards such as approved regions and instance types, encryption, mandatory tags, and restricted network exposure.
- Add static security checks and cost estimation (for example, Infracost) to pull requests; treat estimates as decision support, not exact bills.
- Use cloud IAM/SCPs and quotas as defense in depth, since CI policy alone can be bypassed by another provisioning path.
- Provide clear policy feedback and an exception process with an owner and expiration, then monitor actual spend and deployed configuration.

### 8) Scenario: Someone changes a security group directly in the AWS Console. How does Terraform detect and handle the drift?

Answer:
- A normal `terraform plan` refreshes observed provider state and compares it with configuration; `terraform plan -refresh-only` focuses on proposing state updates for observed changes.
- If the manual change is unauthorized, review the plan and apply the configuration that restores the approved rule.
- If the change is intentional and approved, update the HCL first, review the resulting plan, and apply so code remains the source of truth.
- Check for `ignore_changes` or provider behavior that may hide or limit detection.
- Investigate why out-of-band changes were possible and improve access controls and drift monitoring.

### 9) Scenario: Production drift exists, and Terraform proposes replacing a resource. How do you reconcile it without downtime?

Answer:
- Do not apply until you understand why replacement is proposed. Read the plan's replacement reason and confirm the resource address, provider, state, and configuration.
- Compare the live resource with state and configuration; inspect provider documentation to determine whether the changed attribute is immutable or whether a configuration mismatch is causing replacement.
- If the manual change is desired, update configuration and import/state only when the binding is actually wrong. If it is not desired, plan a safe correction.
- For unavoidable replacement, design a parallel replacement and traffic/data migration where supported, with health checks and a rollback path.
- Back up critical data, verify dependencies, and review the final plan with service owners before production execution.

### 10) Scenario: How do you deploy across multiple AWS accounts and regions from one CI/CD pipeline?

Answer:
- Use provider aliases for multiple regions and configure account access through role assumption rather than long-lived user keys.
- Prefer CI workload identity federation (such as OIDC) to assume narrowly scoped roles in target accounts.
- Configure explicit provider aliases and pass the intended provider to child modules; avoid relying on an implicit default region/account.
- Isolate state per account/region or ownership boundary and use backend access controls appropriate to each environment.
- Validate the caller identity and target region/account before applying, and use approvals for production.

### 11) Scenario: An EC2 instance type change is in-place, but an AMI change or database subnet-group change requires replacement. How do you handle production safely?

Answer:
- Inspect the plan and provider documentation to understand whether the operation is in-place or replace-required; behavior depends on the resource and provider version.
- For stateless compute, use a rolling or blue-green replacement behind a load balancer, with health checks and enough spare capacity.
- `create_before_destroy` can help only when the provider permits both resources to coexist and naming, quota, and dependency constraints allow it.
- For databases and other stateful services, plan data migration, replicas/failover, backups, and compatibility explicitly; lifecycle flags alone do not make a replacement zero-downtime.
- Apply in a controlled window or progressive rollout, validate service health, and retain a tested recovery path.

### 12) Scenario: How do you build a reusable VPC module that works across AWS regions and handles subnet CIDRs?

Answer:
- Discover or accept an explicit set of Availability Zones, and make selection deterministic so a change in AZ ordering does not unexpectedly remap subnets.
- Accept a VPC CIDR and calculate subnet blocks with functions such as `cidrsubnet`, documenting the prefix and subnet allocation plan.
- Validate inputs, including CIDR overlap, subnet counts, address capacity, and region/AZ compatibility.
- Consider AWS VPC IP Address Manager (IPAM) for centrally governed allocation at organizational scale.
- Expose stable outputs (VPC ID, subnet IDs, route table IDs) and test module behavior across supported regions and input combinations.

### 13) Scenario: You need to remove an S3 bucket from Terraform management without deleting it or its data. What do you do?

Answer:
- On a suitable Terraform version, prefer a `removed` block with `lifecycle { destroy = false }` so the handoff is explicit and reviewable; otherwise, after a reviewed backup and state check, use `terraform state rm <address>`.
- A `state rm` removes Terraform's binding only; it does not call AWS to delete the bucket. The bucket remains but becomes unmanaged.
- Remove or update the resource configuration in the same change, or the next plan may propose creating a bucket again.
- `terraform destroy` removes managed resources by calling the provider to delete them; it is not equivalent to removing a state binding.
- Confirm ownership and future management before unmanaging the bucket.

### 14) Scenario: Terraform modules in separate repositories depend on each other, and deployments fail due to version conflicts. How do you improve reliability?

Answer:
- Publish modules as versioned releases and pin consumers to known-good versions rather than floating branches or latest tags.
- Define and test provider and Terraform version constraints, and upgrade dependencies through reviewed changes.
- Model inter-stack dependencies explicitly and deploy in a controlled order; avoid hidden coupling through undocumented remote-state outputs.
- Use contract tests or example root configurations to validate module inputs and outputs before release.
- Maintain a compatibility matrix and a rollback path for module releases.

### 15) Scenario: The cloud provider API rate-limits a Terraform apply. How do you respond?

Answer:
- Identify the throttled API and provider error in logs; check whether multiple pipelines or stacks are making concurrent requests.
- Use provider-supported retries/backoff and, if appropriate, reduce Terraform concurrency with `-parallelism=<n>`. Retry behavior and configuration are provider-specific, not a universal Terraform setting.
- Avoid automatic retries of the entire apply while another run may be active; first confirm state lock and operation status.
- Split very large or highly parallel workloads along sound ownership/lifecycle boundaries when it reduces contention.
- Use delays only for a documented eventual-consistency dependency; blanket `time_sleep` resources usually slow runs without fixing throttling.

### 16) Scenario: State is corrupted and there is no known-good backup. What is the recovery plan?

Answer:
- Freeze applies and preserve the current state and diagnostic evidence; confirm no other process is writing to the backend.
- Reconstruct an inventory of the actual resources using provider APIs and compare it with the configuration and any surviving state/plan artifacts.
- Import resources incrementally with `import` blocks or `terraform import`, ensuring each resource maps to the correct address and configuration.
- Use `terraform plan -refresh-only` to review provider-observed differences for resources already tracked; it cannot discover and bind every untracked object automatically.
- Review normal plans in small, logical batches and do not apply until unexpected creates, deletes, or replacements are understood.
- After recovery, enable backend versioning/snapshots, locking, restricted access, and tested backups. Avoid relying on deprecated `terraform refresh` as a recovery shortcut.

### 17) Scenario: How do you reduce the risk of accidentally deleting critical infrastructure?

Answer:
- Review the full plan, especially replacement and destroy actions, and require peer approval for production changes.
- Use separate state boundaries, least-privilege cloud roles, protected branches, and policy checks for critical resources.
- Apply `lifecycle { prevent_destroy = true }` to resources that must not be destroyed through Terraform while that lifecycle rule remains in configuration.
- Understand the limitation: removing the resource block also removes its lifecycle rule from the desired configuration, so `prevent_destroy` is not a complete deletion safeguard.
- Use provider-native protections, backups, deletion protection, and recovery tests where supported; do not bypass a deliberate protection without approval.

### 18) Scenario: How do you implement a multi-account, multi-region Terraform architecture?

Answer:
- Use reusable modules with separate root configurations and state boundaries for account, region, and ownership as appropriate.
- Authenticate from CI with short-lived role assumption and provider aliases, with least privilege in each target account.
- Pin module/provider versions, keep backend keys isolated, and make cross-stack interfaces explicit and minimal.
- Use centralized policy, tagging, logging, and release controls while allowing environment-specific inputs.
- Validate the target account/region and review changes independently for each state before apply.

### 19) Scenario: How do you achieve zero-downtime infrastructure updates with Terraform?

Answer:
- Design the service architecture and deployment process for overlap: rolling replacement, blue-green environments, load balancer target groups, or weighted traffic shifting.
- Make application and schema changes backward-compatible so old and new versions can coexist during rollout.
- Use health checks and external deployment validation to decide when to shift traffic; avoid relying on `local-exec` provisioners as the primary health or rollout mechanism.
- Use `create_before_destroy` only when resource naming, provider behavior, dependencies, and quotas allow concurrent old/new resources.
- Plan stateful changes separately with replication, migration, backups, and tested recovery. Terraform lifecycle settings alone cannot guarantee zero downtime.

### 20) Scenario: What happens if the Terraform state file is accidentally deleted?

Answer:
- Terraform loses the bindings between configuration addresses and real resources. The cloud resources do not get deleted, but Terraform may propose creating duplicates or fail because names/identifiers already exist.
- Do not run `apply`. Freeze runs, then restore a verified backend version or backup.
- If no backup exists, inventory and import existing resources into the correct state, then review plans carefully.
- State loss is distinct from deleting the resource configuration; protect state with versioning, locking, encryption, access controls, and backups.

### 21) Scenario: What happens if you delete a resource definition from Terraform configuration?

Answer:
- If the resource remains in state and is simply removed from configuration, the next plan normally proposes destroying it.
- Review the plan before applying. If the intent is to stop managing the resource but retain it, use a `removed` block with `destroy = false` where supported, or carefully remove its state binding with `terraform state rm`.
- Removing the state binding does not delete the real resource; leaving its configuration in place may make Terraform plan to create it again.
- `prevent_destroy` blocks a planned destroy only while its lifecycle rule remains in configuration; it is not a substitute for an explicit unmanagement or protection workflow.
