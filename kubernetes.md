# DevOps Engineer Interview Questions and Answers (5 Years Experience)

## Kubernetes Cluster Overview

- **Cluster:** A set of control-plane and worker nodes that run containerized workloads. Managed services such as EKS operate some or all control-plane components for you.
- **Control plane:** The API server exposes the Kubernetes API; etcd stores cluster state; the scheduler assigns unscheduled Pods to nodes; and the controller manager runs controllers that reconcile actual state toward desired state.
- **Node:** A machine that runs Pods. The kubelet communicates with the API server and makes sure the Pod's containers are running as specified. A container runtime, such as containerd, runs containers.
- **Pod:** The smallest deployable Kubernetes unit, containing one or more co-located containers that share network identity and can share volumes.
- **kube-proxy:** Implements Service networking on many clusters. Some CNIs replace or supplement it with eBPF-based data planes.
- **CNI:** Provides Pod networking and may enforce network policies. Namespace separation alone does not block Pod-to-Pod traffic.
- **Cloud controller manager and cloud integrations:** Integrate Kubernetes with provider resources such as load balancers and routes; they are not required in every cluster.

## Kubernetes Scenario-Based Questions and Answers

### 1) Scenario: Pods in one namespace can communicate, but Pods in another namespace cannot reach them. How do you troubleshoot?

Answer:
- Kubernetes namespaces do not isolate network traffic by themselves. Check whether a `NetworkPolicy` selects either the source or destination Pods and whether the installed CNI enforces policies.
- Confirm the destination Service exists and its selector matches ready Pods:
  `kubectl get svc,endpointslices -n <namespace>`
- Check DNS and use the fully qualified Service name, for example `api.team-a.svc.cluster.local`.
- From a temporary or approved diagnostic Pod, test DNS and connectivity to the Service and Pod IP; inspect source and destination policies, CNI status, and relevant network logs.
- Verify ports and protocol on the Service and application, and check that the application listens on the Pod interface rather than only loopback.
- Make the smallest policy or configuration change needed, then test both allowed and intentionally denied traffic.

### 2) Scenario: HPA is not scaling Pods even though CPU appears above the configured threshold. How do you debug it?

Answer:
- Inspect `kubectl describe hpa <name> -n <namespace>` for current metrics, conditions, events, and the `ScalingActive`/`AbleToScale` status.
- Check that Metrics Server is healthy and that resource metrics are available with `kubectl top pods`.
- For CPU utilization targets, verify containers have CPU requests; utilization is calculated relative to requests. Check the target reference, metric type, and HPA `minReplicas`/`maxReplicas`.
- Confirm CPU is actually the metric HPA observes; dashboards may show node CPU or a different time window. For custom/external metrics, validate the adapter and metric query.
- Check scaling behavior policies, stabilization windows, Pod readiness/startup behavior, pending Pods, and whether the cluster has capacity to schedule additional replicas.
- Validate the scale-up with HPA events, Deployment replica count, and Pod scheduling events.

### 3) Scenario: New worker nodes join an EKS cluster, but Pods remain `Pending`. What could be wrong?

Answer:
- Start with `kubectl describe pod <pod> -n <namespace>` and its Events; a Pod in `Pending` is commonly not schedulable yet.
- Check node readiness, allocatable CPU/memory, taints, labels, node selectors/affinity, topology spread, and any namespace quota or limit range.
- Check EKS node-group capacity, Auto Scaling limits, and whether the scheduler can place the workload on the joined nodes.
- For PVC-backed Pods, inspect PVC/PV binding, StorageClass, volume topology, and EBS CSI driver health.
- Check VPC CNI health and subnet/IP capacity when nodes or Pods cannot obtain network addresses.
- Image pull failures usually appear after a Pod is assigned to a node as `ErrImagePull` or `ImagePullBackOff`; inspect the Pod phase and events rather than treating every failure as a scheduling issue.

### 4) Scenario: You need to deploy a critical microservice update without downtime. What strategy do you use?

Answer:
- Use a rolling Deployment with readiness probes and a suitable strategy, often `maxUnavailable: 0` and `maxSurge: 1` or higher if cluster capacity allows.
- Ensure the application can run old and new versions concurrently and that database/API changes are backward-compatible during the rollout.
- Handle graceful shutdown with an adequate termination grace period and application signal handling; use a `preStop` hook only when its behavior is understood.
- Monitor rollout status and service-level error/latency signals; pause or roll back if the new ReplicaSet is unhealthy.
- For higher-risk changes, use canary or blue-green delivery with explicit traffic shifting and validation.
- A PodDisruptionBudget helps limit voluntary disruptions, but should not be treated as a substitute for safe Deployment rollout settings and capacity.

### 5) Scenario: Users report intermittent failures between microservices, but individual service logs look normal. How do you find the root cause?

Answer:
- Establish the affected requests, time range, and service path. Compare request rate, error rate, latency percentiles, and saturation for each dependency.
- Use distributed tracing and a shared correlation/trace ID to locate the failing hop; logs from one service alone may not show downstream delays or retries.
- Check Service endpoints, DNS, connection pools, timeouts, retry behavior, and network policy/CNI health.
- Inspect node and Pod restarts, throttling, memory pressure, readiness changes, and rollout events.
- Correlate symptoms with deployments, configuration, certificates, and external dependency status.
- Mitigate the failing dependency or roll back the change, then confirm end-to-end recovery and add missing telemetry or alerts.

### 6) Scenario: A load balancer URL returns HTTP 503 for an application in Kubernetes. How do you troubleshoot?

Answer:
- Identify which layer returns the 503: cloud load balancer, ingress controller, service mesh, or application. Check the response headers and the corresponding access/controller logs.
- Check the load balancer target health and listener/routing rules, then inspect the Ingress/Gateway and its backend Service.
- Verify the Service selector, `port`/`targetPort`, and ready EndpointSlices. A Service with no ready endpoints is a common cause.
- Check Pod readiness probes, application logs, rollout status, and whether the app is listening on the configured port/interface.
- For EKS, confirm target type (instance or IP), target-group health-check path/port, subnet and security-group rules, and AWS Load Balancer Controller status.
- Test in-cluster Service connectivity, correct the failing layer, and verify the external URL and target health after recovery.

### 7) Scenario: A Pod is in `CrashLoopBackOff`. What are likely causes and how do you investigate?

Answer:
- `CrashLoopBackOff` means the container repeatedly exits or is restarted; it is a restart backoff state, not the underlying cause.
- Inspect `kubectl describe pod` for exit codes, restart count, events, and probe failures. Read current and previous container logs with `kubectl logs <pod> -c <container> --previous`.
- Check command/arguments, environment variables, ConfigMaps/Secrets, mounted files and permissions, and application startup dependencies.
- Check `Last State` for `OOMKilled`, then compare memory use with requests/limits and inspect node pressure.
- Validate startup, readiness, and liveness probes; an overly aggressive liveness probe can restart a slow-starting application. Use a startup probe where appropriate.
- Fix the cause and confirm the container remains healthy over time. Image pull errors typically show as `ImagePullBackOff`, not `CrashLoopBackOff`.

### 8) Scenario: A Pod stays in `Pending` after deployment. What are the likely causes?

Answer:
- Use `kubectl describe pod` and inspect Events first; they usually identify the scheduling or volume-binding reason.
- Check insufficient node CPU/memory, unbound PVCs, quota/limit-range constraints, and node selectors, affinity, topology spread, taints, and tolerations.
- Confirm nodes are `Ready`, schedulable, and have enough allocatable capacity; inspect scheduler health if events indicate scheduler errors.
- For storage, inspect PVC, StorageClass, provisioner, and zone/topology compatibility.
- Distinguish scheduling `Pending` from container startup failures such as `ImagePullBackOff`, which generally happen after scheduling.
- Resolve the specific constraint rather than removing placement or resource controls indiscriminately.

### 9) Scenario: How do you safely update a Kubernetes cluster?

Answer:
- Follow the provider's supported upgrade path and read version-specific release notes, API deprecations, and version-skew requirements.
- Inventory deprecated APIs, validate manifests and add-ons in a test environment, and confirm backups and recovery procedures for application data.
- For EKS, upgrade the control plane through the supported process, then compatible add-ons (such as VPC CNI, CoreDNS, and kube-proxy) and node groups according to the EKS upgrade guidance.
- Upgrade nodes gradually: add or update capacity, cordon and drain in controlled batches while respecting PodDisruptionBudgets, then verify workloads before proceeding.
- Monitor cluster and application health after each stage; keep capacity available during replacement and avoid assuming that a managed control-plane version can simply be rolled back.
- etcd backup is relevant when operating a self-managed control plane; AWS manages the EKS control-plane etcd. Back up and test recovery for stateful application data separately.

### 10) Scenario: A Service exists, but it has no endpoints. What do you check?

Answer:
- Compare the Service selector with the labels on Pods in the same namespace.
- Check whether matching Pods are Ready; unready Pods are typically excluded from ready endpoints.
- Verify Service ports and `targetPort`, and confirm the application listens on that port.
- Inspect EndpointSlices, Pod events, readiness probes, and any `publishNotReadyAddresses` setting that may be relevant.
- Correct the selector or readiness issue and verify that endpoints populate and the Service is reachable from a client Pod.

### 11) Scenario: A PersistentVolumeClaim is stuck in `Pending`. How do you investigate?

Answer:
- Describe the PVC and inspect events for provisioning or binding errors.
- Verify the StorageClass, provisioner/CSI driver, access mode, requested capacity, and any `volumeBindingMode`.
- Check cloud quotas, permissions, subnet/zone constraints, and CSI controller/node component health.
- Confirm a statically provisioned PV, if used, matches the claim's capacity, access modes, and storage class.
- Resolve the specific storage issue and validate the claim binds and the Pod mounts it; do not delete a claim containing important data without a recovery plan.
