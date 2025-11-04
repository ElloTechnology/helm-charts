# Ello App Helm Chart

A flexible and production-ready Helm chart for deploying applications on Kubernetes. This chart provides comprehensive configuration options for deploying containerized applications with best practices built-in.

## Features

- ✅ **Production-ready defaults** with security best practices
- ✅ **Flexible probe configuration** (HTTP, TCP, exec, gRPC) for liveness, readiness, and startup
- ✅ **Multi-sidecar support** with full feature parity
- ✅ **Autoscaling** with Horizontal Pod Autoscaler (HPA)
- ✅ **Pod Disruption Budgets** for high availability
- ✅ **Network Policies** for network segmentation
- ✅ **Service mesh ready** (Istio, Linkerd compatible)
- ✅ **Prometheus monitoring** via ServiceMonitor
- ✅ **External Secrets** integration
- ✅ **Gateway API** support (HTTPRoute)
- ✅ **Init containers** and lifecycle hooks
- ✅ **Topology spread constraints** for multi-zone deployments
- ✅ **ConfigMap and PVC** management

## Prerequisites

- Kubernetes 1.22+
- Helm 3.0+

### Optional Dependencies

- **Prometheus Operator** - For ServiceMonitor support
- **External Secrets Operator** - For external secrets integration
- **Gateway API** - For HTTPRoute support
- **Metrics Server** - For HPA based on CPU/memory

## Installation

### Add the Helm repository

```bash
helm repo add ello https://ellotechnology.github.io/helm-charts
helm repo update
```

### Install the chart

```bash
# Basic installation
helm install my-app ello/app

# With custom values
helm install my-app ello/app -f values.yaml

# With inline values
helm install my-app ello/app \
  --set image.repository=myapp \
  --set image.tag=v1.0.0 \
  --set service.port=8080
```

### Upgrade

> **⚠️ Breaking Changes in v2.0.0**
>
> If upgrading from v1.x to v2.0.0, please review the [CHANGELOG.md](CHANGELOG.md) for breaking changes and migration guide. Key changes include:
> - Default probes removed (must be explicitly defined)
> - Deprecated fields removed (`port`, `pvc.created`, `pvc.create`)
> - Minimum Kubernetes version increased to 1.22+
> - Only `networking.k8s.io/v1` Ingress API supported

```bash
helm upgrade my-app ello/app -f values.yaml
```

### Uninstall

```bash
helm uninstall my-app
```

## Configuration

### Basic Configuration

#### Image Configuration

```yaml
image:
  repository: nginx
  tag: "1.25"
  pullPolicy: IfNotPresent

imagePullSecrets:
  - name: registry-credentials
```

#### Container Ports

```yaml
containerPort:
  name: http
  port: 8080
  protocol: TCP

extraPorts:
  - name: metrics
    port: 9090
    protocol: TCP
  - name: grpc
    port: 50051
    protocol: TCP
```

#### Service Configuration

```yaml
service:
  type: ClusterIP
  port: 80
  targetPort: http
  protocol: TCP
  portName: http
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
```

### Advanced Configuration

#### Security Context

```yaml
# Pod-level security context
podSecurityContext:
  runAsNonRoot: true
  runAsUser: 65534
  fsGroup: 65534
  seccompProfile:
    type: RuntimeDefault

# Container-level security context
securityContext:
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  runAsNonRoot: true
  runAsUser: 65534
  capabilities:
    drop:
      - ALL
```

#### Environment Variables

```yaml
# Simple key-value pairs
env:
  LOG_LEVEL: "info"
  NODE_ENV: "production"

# From secrets
secretEnv:
  DATABASE_PASSWORD:
    name: db-secret
    key: password
  API_KEY:
    name: api-credentials
    key: api-key

# From ConfigMaps or Secrets
envFrom:
  - configMapRef:
      name: app-config
  - secretRef:
      name: app-secrets
```

#### Health Probes

**HTTP Probe:**

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: http
    scheme: HTTP
    httpHeaders:
      - name: Custom-Header
        value: Awesome
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  successThreshold: 1
  failureThreshold: 3
```

**TCP Probe:**

```yaml
livenessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 20
```

**Exec Probe:**

```yaml
livenessProbe:
  exec:
    command:
      - cat
      - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```

**gRPC Probe:**

```yaml
livenessProbe:
  grpc:
    port: 50051
    service: health.v1.Health
  initialDelaySeconds: 10
```

**Startup Probe** (for slow-starting applications):

```yaml
startupProbe:
  httpGet:
    path: /healthz
    port: http
  initialDelaySeconds: 0
  periodSeconds: 10
  timeoutSeconds: 3
  failureThreshold: 30
```

#### Resource Limits

```yaml
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

#### Autoscaling

```yaml
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
  targetMemoryUtilizationPercentage: 80
```

#### Pod Disruption Budget

```yaml
pdb:
  enabled: true
  minAvailable: 1
  # OR use maxUnavailable
  # maxUnavailable: 1
  unhealthyPodEvictionPolicy: IfHealthyBudget
```

#### Lifecycle Hooks

```yaml
lifecycle:
  preStop:
    exec:
      command: ["/bin/sh", "-c", "sleep 15"]
  postStart:
    exec:
      command: ["/bin/sh", "-c", "echo Container started"]
```

#### Persistent Storage

```yaml
pvc:
  enabled: true
  name: app-data
  accessModes:
    - ReadWriteOnce
  size: 10Gi
  storageClassName: fast-ssd

volumes:
  - name: data
    persistentVolumeClaim:
      claimName: app-data

volumeMounts:
  - name: data
    mountPath: /data
```

### Sidecar Containers

Deploy sidecar containers alongside your main application:

```yaml
sidecars:
  - name: logging-agent
    image:
      repository: fluent/fluent-bit
      tag: "2.0"
      pullPolicy: IfNotPresent
    env:
      - name: LOG_LEVEL
        value: "info"
    ports:
      - name: metrics
        containerPort: 2020
        protocol: TCP
    volumeMounts:
      - name: logs
        mountPath: /var/log
        readOnly: true
    resources:
      limits:
        cpu: 100m
        memory: 128Mi
      requests:
        cpu: 50m
        memory: 64Mi
    livenessProbe:
      httpGet:
        path: /api/v1/health
        port: 2020
      initialDelaySeconds: 30
      periodSeconds: 10
```

See the [repository README](../../readme.md#sidecar-containers) for more sidecar examples.

### Networking

#### Ingress

```yaml
ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: myapp.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: myapp-tls
      hosts:
        - myapp.example.com
```

#### Gateway API (HTTPRoute)

```yaml
httpRoute:
  enabled: true
  annotations:
    route.example.com/timeout: "30s"
  parentRefs:
    - name: gateway
      sectionName: https
      namespace: gateway-system
  hostnames:
    - myapp.example.com
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /api
      filters:
        - type: RequestHeaderModifier
          requestHeaderModifier:
            set:
              - name: X-Custom-Header
                value: custom-value
```

#### Network Policy

```yaml
networkPolicy:
  enabled: true
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: allowed-namespace
      ports:
        - protocol: TCP
          port: 8080
  egress:
    - to:
        - namespaceSelector: {}
      ports:
        - protocol: TCP
          port: 443
        - protocol: TCP
          port: 53
          protocol: UDP
```

### Monitoring

#### Prometheus ServiceMonitor

```yaml
monitoring:
  enabled: true
  serviceMonitor:
    enabled: true
    interval: 30s
    scrapeTimeout: 10s
    path: /metrics
    labels:
      release: prometheus
```

### Advanced Scheduling

#### Topology Spread Constraints

```yaml
topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        app.kubernetes.io/name: app
  - maxSkew: 1
    topologyKey: kubernetes.io/hostname
    whenUnsatisfiable: ScheduleAnyway
    labelSelector:
      matchLabels:
        app.kubernetes.io/name: app
```

#### Node Affinity

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: node-type
              operator: In
              values:
                - high-memory
```

#### Pod Priority

```yaml
priorityClassName: high-priority
```

#### Runtime Class

```yaml
runtimeClassName: gvisor
```

## Common Patterns

### Production Deployment

```yaml
replicaCount: 3

image:
  repository: myapp
  tag: "v1.2.3"
  pullPolicy: IfNotPresent

resources:
  limits:
    cpu: 1000m
    memory: 1Gi
  requests:
    cpu: 250m
    memory: 256Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70

pdb:
  enabled: true
  minAvailable: 2

podSecurityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 1000

securityContext:
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  runAsNonRoot: true
  capabilities:
    drop:
      - ALL

topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        app.kubernetes.io/name: myapp
```

### Development Environment

```yaml
replicaCount: 1

image:
  repository: myapp
  tag: "latest"
  pullPolicy: Always

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 100m
    memory: 128Mi

autoscaling:
  enabled: false

livenessProbe:
  httpGet:
    path: /healthz
    port: http
  initialDelaySeconds: 10
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: http
  initialDelaySeconds: 5
  periodSeconds: 5
```

## Sidecar Containers

The app chart supports running sidecar containers alongside the main application container. Sidecars share the same pod, network namespace, and storage volumes with the main container, making them ideal for:

- **Logging agents** (Fluent Bit, Fluentd, Filebeat)
- **Service mesh proxies** (Envoy, Linkerd)
- **Monitoring agents** (Prometheus exporters, Datadog agent)
- **Security tools** (Vault agent, authentication proxies)
- **Helper services** (Configuration reloaders, data syncers)

### Basic Usage

Add sidecars to your values file:

```yaml
sidecars:
  - name: logging-agent
    image:
      repository: fluent/fluent-bit
      tag: "2.0"
      pullPolicy: IfNotPresent
    resources:
      limits:
        cpu: 100m
        memory: 128Mi
      requests:
        cpu: 50m
        memory: 64Mi
```

### Multiple Sidecars

You can run multiple sidecars in a single pod:

```yaml
sidecars:
  - name: logging-agent
    image:
      repository: fluent/fluent-bit
      tag: "2.0"
    volumeMounts:
      - name: logs
        mountPath: /var/log
        readOnly: true

  - name: metrics-exporter
    image:
      repository: prom/statsd-exporter
      tag: "v0.24.0"
    ports:
      - name: metrics
        containerPort: 9102
        protocol: TCP
```

### Advanced Configuration

Sidecars support full feature parity with the main container:

#### Commands and Arguments

```yaml
sidecars:
  - name: envoy-proxy
    image:
      repository: envoyproxy/envoy
      tag: "v1.28"
    command:
      - envoy
    args:
      - -c
      - /etc/envoy/envoy.yaml
      - --service-cluster
      - my-cluster
```

#### Environment Variables

```yaml
sidecars:
  - name: datadog-agent
    image:
      repository: datadog/agent
      tag: "7.48"
    env:
      - name: DD_SITE
        value: "datadoghq.com"
      - name: DD_ENV
        value: "production"
    secretEnv:
      - name: DD_API_KEY
        valueFrom:
          secretKeyRef:
            name: datadog-secret
            key: api-key
```

#### Security Context

```yaml
sidecars:
  - name: vault-agent
    image:
      repository: vault
      tag: "1.15"
    securityContext:
      runAsUser: 100
      runAsNonRoot: true
      readOnlyRootFilesystem: true
      capabilities:
        drop:
          - ALL
```

#### Health Probes

Sidecars support all probe types: httpGet, tcpSocket, exec, and grpc.

**HTTP Probe:**

```yaml
sidecars:
  - name: nginx-sidecar
    image:
      repository: nginx
      tag: "1.25"
    ports:
      - name: http
        containerPort: 8080
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10
      timeoutSeconds: 5
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
```

**TCP Probe:**

```yaml
sidecars:
  - name: redis-sidecar
    image:
      repository: redis
      tag: "7.0"
    livenessProbe:
      tcpSocket:
        port: 6379
      initialDelaySeconds: 15
      periodSeconds: 20
```

**Exec Probe:**

```yaml
sidecars:
  - name: app-helper
    image:
      repository: busybox
      tag: "1.36"
    livenessProbe:
      exec:
        command:
          - cat
          - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

**gRPC Probe:**

```yaml
sidecars:
  - name: grpc-service
    image:
      repository: my-grpc-service
      tag: "v1.0"
    livenessProbe:
      grpc:
        port: 9090
        service: health.v1.Health
      initialDelaySeconds: 10
```

### Shared Volumes

Sidecars can access volumes defined in the main values:

```yaml
volumes:
  - name: shared-data
    emptyDir: {}
  - name: config
    configMap:
      name: app-config

sidecars:
  - name: data-processor
    image:
      repository: my-processor
      tag: "1.0"
    volumeMounts:
      - name: shared-data
        mountPath: /data
      - name: config
        mountPath: /etc/config
        readOnly: true
```

### Complete Example

Here's a full example with an Envoy proxy sidecar for a service mesh:

```yaml
sidecars:
  - name: envoy-proxy
    image:
      repository: envoyproxy/envoy
      tag: "v1.28"
      pullPolicy: IfNotPresent
    command:
      - envoy
    args:
      - -c
      - /etc/envoy/envoy.yaml
      - --service-cluster
      - my-app
    env:
      - name: ENVOY_LOG_LEVEL
        value: "info"
    ports:
      - name: proxy
        containerPort: 8000
        protocol: TCP
      - name: admin
        containerPort: 9901
        protocol: TCP
    volumeMounts:
      - name: envoy-config
        mountPath: /etc/envoy
        readOnly: true
    securityContext:
      runAsUser: 1337
      runAsNonRoot: true
    resources:
      limits:
        cpu: 200m
        memory: 256Mi
      requests:
        cpu: 100m
        memory: 128Mi
    readinessProbe:
      httpGet:
        path: /ready
        port: 9901
      initialDelaySeconds: 3
      periodSeconds: 3
    livenessProbe:
      httpGet:
        path: /server_info
        port: 9901
      initialDelaySeconds: 10
      periodSeconds: 10

volumes:
  - name: envoy-config
    configMap:
      name: envoy-config
```

### Best Practices

1. **Resource Limits**: Always set resource limits and requests for sidecars to prevent them from consuming excessive resources
2. **Health Probes**: Configure appropriate health probes to ensure sidecars are healthy before receiving traffic
3. **Security Context**: Apply restrictive security contexts (runAsNonRoot, readOnlyRootFilesystem) when possible
4. **Shared Volumes**: Use emptyDir volumes for sharing data between main container and sidecars
5. **Graceful Shutdown**: Ensure sidecars handle SIGTERM properly for clean pod termination
6. **Logging**: Configure sidecars to log to stdout/stderr for proper log aggregation

## Troubleshooting

### Pod not starting

1. Check pod events:

   ```bash
   kubectl describe pod <pod-name>
   ```

2. Check logs:

   ```bash
   kubectl logs <pod-name>
   ```

3. Check init container logs (if using init containers):

   ```bash
   kubectl logs <pod-name> -c init
   ```

### Probes failing

- Ensure probe paths return appropriate status codes (200-399 for HTTP)
- Adjust `initialDelaySeconds` if application takes time to start
- Check `timeoutSeconds` is sufficient
- Use `startupProbe` for slow-starting applications

### Service not accessible

1. Check service endpoints:

   ```bash
   kubectl get endpoints <service-name>
   ```

2. Verify port configuration matches container ports
3. Check network policies aren't blocking traffic

### HPA not scaling

1. Ensure Metrics Server is installed:

   ```bash
   kubectl get deployment metrics-server -n kube-system
   ```

2. Check HPA status:

   ```bash
   kubectl get hpa
   kubectl describe hpa <hpa-name>
   ```

3. Verify resource requests are set

## Parameters

For a complete list of configuration parameters, see [values.yaml](values.yaml).

### Key Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `1` |
| `image.repository` | Container image repository | `nginx` |
| `image.tag` | Container image tag | `"1.25"` |
| `service.type` | Kubernetes service type | `ClusterIP` |
| `service.port` | Service port | `80` |
| `ingress.enabled` | Enable ingress | `false` |
| `autoscaling.enabled` | Enable HPA | `false` |
| `pdb.enabled` | Enable Pod Disruption Budget | `false` |
| `resources` | Resource limits and requests | `{}` |

## Contributing

Contributions are welcome! Please submit issues and pull requests to the [GitHub repository](https://github.com/ElloTechnology/helm-charts).

## License

This chart is provided as-is under the MIT License.

## Support

For support and questions:

- GitHub Issues: <https://github.com/ElloTechnology/helm-charts/issues>
- Documentation: <https://github.com/ElloTechnology/helm-charts>

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for release history and changes.
