# DevOps Engineer Interview Questions and Answers (5 Years Experience)

## Linux Scenario-Based Questions and Answers

### 1) Scenario: A Linux server is responding slowly. How do you identify the bottleneck?

Answer:
- Check load and CPU with `uptime`, `top` or `htop`, and `mpstat` if available.
- Inspect memory and swap with `free -h` and `vmstat 1`; check whether processes are being killed by the OOM killer.
- Check disk capacity and I/O with `df -h`, `df -i`, and tools such as `iostat` if installed.
- Inspect process-level usage and system logs, then correlate the issue with recent deployments or traffic changes.
- Avoid killing processes before understanding their role; mitigate the specific bottleneck and monitor recovery.

### 2) Scenario: A filesystem is full, but `du` does not show enough usage to explain it. What could be happening?

Answer:
- Check both block and inode usage with `df -h` and `df -i`.
- Look for deleted files still held open by processes using `lsof +L1` where available.
- Check mounted filesystems, container log growth, temporary directories, and reserved filesystem space.
- Rotate or truncate logs safely using the service's supported procedure; restart a process only when appropriate.
- Add disk and inode alerts, log rotation, and retention limits to prevent recurrence.

### 3) Scenario: A systemd service will not start after a release. How do you debug it?

Answer:
- Check `systemctl status <service>` and `journalctl -u <service> --since "15 minutes ago"` for the failure reason.
- Validate the unit file, executable path, environment files, working directory, and user/group settings.
- Check file permissions, dependencies, port conflicts, and required mounts or secrets.
- Run configuration validation or a safe foreground test using the service's normal runtime user.
- Restore the prior release if impact is ongoing, then correct and test the unit or application configuration.

### 4) Scenario: An application cannot bind to its configured port. What do you check?

Answer:
- Identify the listening process with `ss -ltnp` or an equivalent tool.
- Check whether the service is binding to the correct address and port and whether another process already owns it.
- Verify firewall rules, host security controls, and whether the service is expected to listen on localhost or a routable interface.
- Review service logs and privileges; ports below 1024 may require specific capabilities or elevated privileges.
- Change only the intended service configuration and confirm the port and application health afterward.

### 5) Scenario: A deployment fails with “permission denied.” How do you troubleshoot without over-permissioning?

Answer:
- Identify the exact user, operation, path, and file mode involved using logs, `id`, `namei -l`, `stat`, and `ls -l`.
- Check ownership, parent-directory execute permissions, ACLs, mount options, and SELinux/AppArmor denials if enabled.
- Grant the service account only the required access to the relevant path.
- Avoid broad `chmod 777` or running the application as root as a workaround.
- Retest under the actual service identity and document the required ownership and permissions in deployment automation.

### 6) Scenario: SSH access to a server suddenly stops working. What do you investigate?

Answer:
- Confirm network reachability, DNS, routing, firewall/security-group rules, and the SSH port.
- Check `sshd` service status and server authentication logs through console or another approved access method.
- Verify the key, username, `authorized_keys` permissions, account status, disk space, and SSH daemon configuration.
- Check whether a recent security policy, configuration change, or automated ban blocked the connection.
- Restore access through an audited break-glass path and tighten or fix the exact failing control.

### 7) Scenario: A process is consuming excessive memory or is being killed. What do you do?

Answer:
- Identify the process and its memory trend using `ps`, `top`/`htop`, and service or application metrics.
- Check kernel messages with `dmesg` or `journalctl -k` for OOM events and inspect cgroup or container memory limits.
- Determine whether the cause is a leak, workload spike, misconfigured limit, or host-wide pressure.
- Mitigate safely by reducing load, restarting only with an approved recovery plan, or adjusting capacity/limits based on evidence.
- Add memory trend monitoring and a regression test or alert for the underlying cause.

### 8) Scenario: A scheduled backup or maintenance task did not run. How do you diagnose it?

Answer:
- Check whether the task is configured in cron, a systemd timer, or another scheduler, and confirm its last and next run times.
- Review scheduler and job logs, environment variables, working directory, permissions, and command paths.
- Run the command manually as the same user with a controlled environment.
- Check lock files, time zone/time synchronization, network or storage dependencies, and exit status handling.
- Add success/failure monitoring and test restore procedures, not only backup creation.

### 9) Scenario: DNS resolution works on one server but fails on another. What do you inspect?

Answer:
- Compare resolver configuration and test the same hostname with `getent hosts` and `dig` or `nslookup`.
- Check search domains, resolver reachability, firewall rules, local caches, and `/etc/hosts`.
- Determine whether the issue is limited to one DNS server, record, network, or application runtime.
- Review recent DNS or network changes and use logs/metrics from the resolver.
- Fix the relevant resolver or record and validate both name resolution and the dependent application connection.

### 10) Scenario: A server's clock differs from the rest of the environment and authentication is failing. How do you respond?

Answer:
- Check time and synchronization status using `timedatectl` and the configured NTP/chrony service.
- Review network access to approved time sources and logs for synchronization errors.
- Correct the time through the system's configured time service rather than manually changing clocks without coordination.
- Consider effects on certificates, tokens, scheduled jobs, and distributed logs.
- Alert on clock skew and validate synchronization across the fleet.
