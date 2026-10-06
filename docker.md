# DevOps Engineer Interview Questions and Answers (5 Years Experience)

## Docker Scenario-Based Questions and Answers

### 1) Scenario: A Java application container exits immediately after startup. How do you debug it?

Answer:
- First, check the container logs using `docker logs <container_id>` to identify the actual failure.
- Check the application entrypoint and command in the Dockerfile: `ENTRYPOINT` and `CMD`.
- Verify environment variables such as DB host, port, secrets, and config files.
- Confirm the container is not failing due to missing dependencies or permission issues.
- Use `docker inspect` to review container config, health status, restart policy, and mounted volumes.
- If it is a Spring Boot app, confirm the app is listening on the correct port and that the `EXPOSE` and health checks match runtime expectations.

Example:
- If the app expects `DB_HOST`, but it is missing, startup fails.
- Fix by supplying config via environment variables or Kubernetes ConfigMaps/Secrets.

### 2) Scenario: Your Docker image is very large and slows down deployments. How do you reduce its size?

Answer:
- Use multi-stage builds to separate build and runtime dependencies.
- Use a minimal base image such as Alpine, distroless, or Debian slim.
- Remove package managers, apt caches, and temporary files before finalizing the image.
- Combine RUN commands to reduce layer count and avoid repeated cache invalidation.
- Use `.dockerignore` to exclude unnecessary files and directories.
- Avoid installing build tools in the final image when only runtime artifacts are needed.
- Use `--no-cache` only when necessary; otherwise rely on layered caching to optimize builds.

Example:
- Node.js build stage installs dependencies and compiles the app.
- Final runtime stage copies only the built `dist` folder and production `node_modules`, reducing image size significantly.

### 3) Scenario: A Docker build is failing due to cache invalidation and taking too long. How do you optimize the Dockerfile?

Answer:
- Order instructions from least frequently changing to most frequently changing.
- Install OS dependencies before copying source code so dependency layers can be cached.
- Copy `package.json` or `requirements.txt` first and run installation before copying full source code.
- Group related commands into fewer `RUN` instructions to reduce layers.
- Use `.dockerignore` to avoid invalidating cache due to irrelevant file changes.
- Pin versions to avoid unexpected dependency updates.

Example:
- Put `COPY package*.json ./` before `COPY . .` so dependency installation is cached when only app code changes.

### 4) Scenario: A container starts successfully but is not reachable externally. What would you check?

Answer:
- Verify the application is listening on the correct port inside the container.
- Check Docker port mapping with `docker run -p hostPort:containerPort` or `docker-compose` configuration.
- Confirm the service is bound to `0.0.0.0` instead of `localhost` when needed.
- Inspect `docker ps`, `docker inspect`, and firewall/network rules.
- If using Docker Compose, validate container names, service definitions, and published ports.
- Check if the application is running in a different network namespace or a reverse proxy is blocking traffic.

Example:
- A Node.js app listens on `127.0.0.1` by default. It still works inside the container but not from outside. Fix with `HOST=0.0.0.0`.

### 5) Scenario: Your application is running in Docker, but it cannot connect to Redis/MySQL/PostgreSQL. What would you do?

Answer:
- Confirm the database service is running and healthy.
- Ensure both containers are on the same Docker network or are connected through Compose service names.
- Use service names as hostnames instead of localhost or the host machine IP.
- Check environment variables for credentials, database names, and ports.
- Verify container readiness using health checks and wait-for logic.
- Validate firewall restrictions and security group rules if running in a cloud environment.

Example:
- In Compose, the app should connect to `postgres:5432`, not `localhost:5432`.

### 6) Scenario: Your production Docker container runs as root, and the security team flags it. How do you fix it?

Answer:
- Create a non-root user and group in the Dockerfile.
- Set file ownership and permissions appropriately.
- Run the application as a non-root user with `USER <uid>`.
- Avoid world-writable directories and ensure the app has proper access.
- Use read-only filesystems where possible for better security.
- Scan the image with Trivy, Anchore, or Snyk and fix critical vulnerabilities.

Example:
- Add:
  `RUN groupadd -r app && useradd -r -g app app`
  `USER app`

### 7) Scenario: A container restarts repeatedly due to crashes. How do you troubleshoot it?

Answer:
- Check `docker logs` to find the root error.
- Review the restart policy with `docker inspect`.
- Check resource exhaustion: CPU, memory, disk space, and file descriptors.
- Validate health checks and dependency availability.
- Confirm whether the main process exits because of a failed startup check or an exception.
- Investigate exit codes and use `docker events` for timestamps around the restart.

Example:
- If memory is exhausted, the application may crash with `Killed` or OOMKilled. Check memory limits and optimize memory usage.

### 8) Scenario: A Docker Compose application fails when deployed on a new machine. How do you ensure environment consistency?

Answer:
- Use a `docker-compose.yml` with versioned dependencies and environment variables.
- Commit the Compose file and configuration to source control.
- Store secrets in `.env` files or secret managers and avoid committing sensitive values.
- Ensure all required volumes and networks are declared explicitly.
- Document startup steps, health checks, and port mappings.
- Use container health checks and service dependencies to avoid startup races.

Example:
- Add `depends_on` with health checks to ensure DB is ready before the app starts.

### 9) Scenario: Your team wants to reduce deployment time using Docker, but builds are slow due to large dependency installs. What do you suggest?

Answer:
- Use BuildKit for better caching and parallelization.
- Build the image in CI with cached layers from a registry or build cache.
- Keep base images small and stable.
- Leverage artifact repositories for dependencies instead of building everything in the container.
- Split build into separate steps so changes in source code do not re-trigger package installation layers.
- Use remote caches and shared build agents for consistency.

### 10) Scenario: A production service is experiencing high CPU and memory usage after containerization. How do you diagnose and fix it?

Answer:
- Measure container resource usage with Docker stats and monitoring tools like Prometheus, Grafana, or cAdvisor.
- Check for memory leaks or thread proliferation in the app itself.
- Set proper CPU and memory limits in the orchestrator or Docker runtime.
- Reduce the number of worker processes and optimize application concurrency.
- Enable autoscaling and resource requests/limits in Kubernetes if applicable.
- Reconfigure JVM or Node.js runtime parameters for the container environment.

Example:
- A Java app may require tuned heap size flags to avoid frequent GC pauses.

### 11) Scenario: You need to persist application data across container restarts. How do you design it?

Answer:
- Use Docker volumes for persistent data instead of storing data in the container filesystem.
- Use bind mounts only for development or configuration that needs to be synced with the host.
- Keep data volumes separate from application layers to avoid destroying data during rebuilds.
- Back up volume data regularly and ensure snapshots are taken in production.
- Configure the correct ownership and permissions so the app can write to the volume.

Example:
- Use `docker volume create postgres-data` and mount it to `/var/lib/postgresql/data`.

### 12) Scenario: A deploy failed because the container was trying to write to a read-only filesystem. What is your approach?

Answer:
- Validate mounted volumes and whether the container was launched with a read-only root filesystem.
- Check for write operations in `/tmp`, logs, or application cache directories.
- Add a writable volume for logs or cache if required.
- Adjust application configuration to use a dedicated writable path.
- Review security hardening settings and update them for compatibility.

### 13) Scenario: The team wants zero-downtime deployments with Docker. How do you achieve that?

Answer:
- Run multiple replicas behind a load balancer.
- Use rolling updates and health checks.
- Keep application state externalized in a database or shared service.
- Use Kubernetes or an orchestration tool to gradually replace old containers.
- Ensure readiness probes are configured correctly before traffic is routed to a new instance.
- Keep the container image immutable and versioned for traceability.

### 14) Scenario: A Docker registry is slow or unavailable during deployments. How do you handle this?

Answer:
- Use a private registry with high availability and replication.
- Cache images in CI/CD to avoid repeated pulls.
- Use image signing and vulnerability scanning before deployment.
- Set up a fallback registry or mirror for critical environments.
- Implement retries and proper timeout settings in the CI pipeline.
- Monitor registry performance and storage capacity.

### 15) Scenario: A Dockerized app works in development but fails in production. What could be the reason?

Answer:
- Differences in environment variables, secrets, or config files.
- The app may be using development mode rather than production mode.
- Resource limits are too strict in production.
- Database or service endpoints differ between environments.
- Logging and debugging settings are different.
- The application may rely on local file paths or ephemeral storage that do not exist in production.

Best practice:
- Use the same base image and configuration management strategy across environments.
- Validate with staging before production rollout.
- Use configuration templating and environment-specific secrets.
