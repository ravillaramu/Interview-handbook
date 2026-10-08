# Shell Script Examples

## HTTP health-check script

This Bash script checks a service endpoint with a bounded number of retries. It exits non-zero if the endpoint remains unhealthy, making it suitable for a simple CI/CD verification step.

Save as `check-health.sh`:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

: "${URL:?Set URL, for example URL=https://service.example.com/health}"

max_attempts="${MAX_ATTEMPTS:-10}"
delay_seconds="${DELAY_SECONDS:-3}"

for ((attempt = 1; attempt <= max_attempts; attempt++)); do
  if curl --fail --silent --show-error --max-time 5 "$URL" >/dev/null; then
    printf 'Health check passed on attempt %s\n' "$attempt"
    exit 0
  fi

  if ((attempt < max_attempts)); then
    printf 'Health check attempt %s/%s failed; retrying in %s seconds\n' \
      "$attempt" "$max_attempts" "$delay_seconds" >&2
    sleep "$delay_seconds"
  fi
done

printf 'Health check failed after %s attempts: %s\n' "$max_attempts" "$URL" >&2
exit 1
```

Run it:

```sh
chmod +x check-health.sh
URL=https://service.example.com/health MAX_ATTEMPTS=12 DELAY_SECONDS=5 ./check-health.sh
```

Use a health endpoint intended for readiness checks, avoid printing credentials or sensitive response bodies, and set retry limits appropriate to your deployment timeout.