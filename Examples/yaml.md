# YAML Syntax Examples

YAML is indentation-sensitive. Use spaces (not tabs), keep indentation consistent, and quote values that could be interpreted as booleans, numbers, or special strings when they are intended to be strings.

## Mapping, list, and scalar values

```yaml
application:
  name: sample-api
  environment: "production"
  replicas: 3
  enabled: true
  maintenanceWindow: null
  ports:
    - name: http
      containerPort: 8080
    - name: metrics
      containerPort: 9090
```

## Strings and multiline values

```yaml
values:
  # Quote strings containing special characters or ambiguous values.
  release: "v1.2.0"
  accountId: "001234"
  enabledText: "false"

  # Literal block preserves line breaks.
  script: |
    echo "Starting checks"
    ./run-tests.sh

  # Folded block joins lines into a wrapped string.
  description: >
    A longer description can be split across source lines
    and is folded into a single paragraph.
```

## Multiple documents in one file

Use `---` to separate Kubernetes resources or other YAML documents:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-settings
data:
  LOG_LEVEL: "info"
---
apiVersion: v1
kind: Service
metadata:
  name: app
spec:
  selector:
    app: sample
  ports:
    - port: 80
      targetPort: 8080
```

## Common mistakes

- **Wrong indentation:** a child key must be indented beneath its parent by spaces.
- **Tabs:** YAML indentation must use spaces.
- **Unquoted special values:** quote values such as `"false"`, `"00123"`, or strings containing `:` when the value should remain a string.
- **Incorrect list placement:** each list item starts with `-` at the same indentation level.
- **Treating YAML as a templating language:** expressions like `${NAME}` or `{{ value }}` are interpreted only by the application or tool that processes them, not by YAML itself.
- **Secrets in plain text:** YAML syntax does not encrypt data. Use the platform's secret manager and avoid committing secret values.

GitHub Actions, Azure Pipelines, Docker Compose, and Kubernetes each add their own schema and expression rules on top of YAML. Valid YAML can still be invalid for a particular tool; validate with that tool's linter or CLI.