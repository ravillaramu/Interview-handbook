# DevOps Engineer Interview Questions and Answers (5 Years Experience)

## Monitoring and Observability Scenario-Based Questions and Answers

### 1) Scenario: Users report that a service is slow, but CPU and memory dashboards look normal. How do you investigate?

Answer:
- Confirm the affected user journey, time window, scope, and whether the issue is latency, errors, or both.
- Check request latency percentiles, error rate, throughput, saturation, and dependency metrics rather than relying only on host CPU and memory.
- Use distributed traces to locate time spent in application code, databases, queues, or external services.
- Correlate traces with structured logs and recent deployments or configuration changes using consistent request and release identifiers.
- Validate recovery with the same user-facing signals and add instrumentation for any blind spot found.

### 2) Scenario: An alert fires repeatedly but engineers find no customer impact. How do you improve it?

Answer:
- Review the alert expression, threshold, evaluation window, labels, and the signal's relationship to user impact.
- Check whether the alert is caused by normal traffic variation, noisy metrics, missing data, or an overly sensitive threshold.
- Prefer actionable alerts tied to symptoms or service-level objectives, with clear runbooks and ownership.
- Use appropriate aggregation and a sustained evaluation window to reduce transient noise without masking real incidents.
- Test the updated alert against historical incidents and normal traffic before deploying it.

### 3) Scenario: A critical service is down, but no alert fired. What do you do?

Answer:
- Treat the outage as an incident and use logs, metrics, traces, and infrastructure health checks to restore service.
- Determine whether telemetry was missing, delayed, incorrectly labeled, or excluded by the alert query.
- Check both the monitored service and the monitoring pipeline: exporter, agent, scrape target, ingestion, and alert evaluator.
- Add independent black-box checks for important user-facing endpoints where appropriate.
- Test the alert end-to-end, including notification delivery and on-call routing.

### 4) Scenario: A Prometheus deployment is running out of storage. How do you address it?

Answer:
- Inspect retention settings, time-series growth, scrape frequency, target count, and disk usage trends.
- Identify high-cardinality labels such as user IDs, request IDs, or unbounded paths and remove or normalize them.
- Set retention by time and/or size according to operational and compliance requirements.
- Increase storage capacity or use an approved long-term metrics backend if required, with a tested migration plan.
- Monitor ingestion rate and storage forecasts so the issue is detected before capacity is exhausted.

### 5) Scenario: Metrics are present, but a dashboard shows gaps for one service. What do you check?

Answer:
- Verify the service's exporter or instrumentation is running and the endpoint is reachable.
- Check scrape target health, scrape configuration, labels, relabeling rules, and authentication or TLS errors.
- Confirm that the dashboard query uses the labels and metric names actually emitted by the service.
- Check for timestamp issues, ingestion delays, rate limits, or backend outages.
- Add a target-health alert and validate the metric path from source to dashboard.

### 6) Scenario: Your team needs an alert for high error rate, but traffic varies significantly. How do you design it?

Answer:
- Use a ratio of failed requests to total requests rather than a fixed error count alone.
- Consider a minimum request-volume condition so a small number of errors at very low traffic does not create misleading noise.
- Use an appropriate rolling window and separate warning from urgent thresholds based on SLO impact.
- Pair the alert with latency and availability signals, and include service, environment, and owner labels.
- Evaluate it against historical traffic, including low-traffic periods and known incidents.

### 7) Scenario: A deployment is followed by a latency regression. How do you connect monitoring to release decisions?

Answer:
- Annotate dashboards and telemetry with release version, commit, and deployment time.
- Compare pre- and post-deployment latency percentiles, error rates, saturation, and dependency behavior.
- Use a canary or progressive rollout and halt promotion if agreed health thresholds are exceeded.
- Roll back to the prior artifact when the evidence points to the release and recovery is safer than live debugging.
- Keep the same dashboards and service-level checks available during rollout and post-deployment observation.

### 8) Scenario: On-call engineers receive too many alerts from one failing dependency. How do you reduce alert fatigue without hiding an outage?

Answer:
- Group related alerts by service, dependency, and incident so one failure does not create many independent pages.
- Page on actionable customer impact; route lower-severity diagnostic alerts to tickets or dashboards.
- Tune thresholds and evaluation windows using incident history, and use inhibition or deduplication for known causal relationships.
- Keep independent critical checks, and make the primary alert point to a runbook with clear ownership.
- Review alert volume and missed incidents regularly with the on-call team.

### 9) Scenario: Logs are too large and difficult to search during incidents. What changes do you make?

Answer:
- Standardize structured logs with timestamp, severity, service, environment, release, and trace or correlation ID.
- Avoid logging secrets and unnecessary high-volume payloads; define sampling and retention based on diagnostic and compliance needs.
- Use consistent parsing and indexing, and ensure the on-call team can search across services and time ranges.
- Set log rotation or ingestion limits and monitor pipeline backpressure or dropped events.
- Test that critical failure details remain available after applying filtering or sampling.

### 10) Scenario: Your team wants to define meaningful reliability targets for a service. How do you establish SLOs?

Answer:
- Select user-centered service indicators such as successful request ratio and latency within a defined threshold.
- Define the measurement window and target based on user expectations and historical performance, not solely on infrastructure capacity.
- Calculate the error budget and agree how it influences release risk and reliability work.
- Instrument and dashboard the indicators, and alert on meaningful error-budget burn rates.
- Review SLO usefulness with stakeholders and adjust carefully as service behavior and expectations evolve.
