# Kubernetes Deployment and Service Example

This manifest deploys two replicas of a sample HTTP application and exposes them internally through a `ClusterIP` Service. Replace the example image and health-check paths with values supported by your application before applying.

Save as `app.yaml`:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: example
  labels:
    app.kubernetes.io/part-of: sample-platform
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: sample-app-config
  namespace: example
data:
  APP_ENV: "production"
  LOG_LEVEL: "info"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app
  namespace: example
  labels:
    app.kubernetes.io/name: sample-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app.kubernetes.io/name: sample-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  template:
    metadata:
      labels:
        app.kubernetes.io/name: sample-app
    spec:
      automountServiceAccountToken: false
      terminationGracePeriodSeconds: 30
      securityContext:
        runAsNonRoot: true
        runAsUser: 10001
        runAsGroup: 10001
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: app
          image: ghcr.io/your-org/sample-app:1.0.0
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 8080
              protocol: TCP
          envFrom:
            - configMapRef:
                name: sample-app-config
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 256Mi
          startupProbe:
            httpGet:
              path: /healthz
              port: http
            failureThreshold: 30
            periodSeconds: 5
          readinessProbe:
            httpGet:
              path: /ready
              port: http
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /healthz
              port: http
            periodSeconds: 10
          volumeMounts:
            - name: tmp
              mountPath: /tmp
      volumes:
        - name: tmp
          emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: sample-app
  namespace: example
spec:
  type: ClusterIP
  selector:
    app.kubernetes.io/name: sample-app
  ports:
    - name: http
      port: 80
      targetPort: http
      protocol: TCP
```

Apply and verify:

```sh
kubectl apply -f app.yaml
kubectl rollout status deployment/sample-app -n example
kubectl get pods,service,endpointslices -n example
kubectl port-forward service/sample-app 8080:80 -n example
```

The sample image, UID, port, and probe paths must match the actual application. `readOnlyRootFilesystem: true` may require additional writable `emptyDir` mounts for application-specific temporary or cache paths. Add an Ingress or Gateway only after configuring its controller, TLS, DNS, and access policy. Use Kubernetes Secrets or an external secret manager for sensitive values; do not put credentials in a ConfigMap or source-controlled manifest.