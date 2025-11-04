# Changelog

All notable changes to this Helm chart will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2025-01-04

### Added

- Initial release of the Ello App Helm chart
- Support for basic Kubernetes deployments with configurable replicas
- Service configuration with ClusterIP, NodePort, and LoadBalancer support
- Ingress support with multiple hosts and TLS
- Gateway API support via HTTPRoute
- Horizontal Pod Autoscaler (HPA) with CPU and memory metrics
- Pod Disruption Budget (PDB) for high availability
- ServiceAccount with configurable automount settings
- ConfigMap management with external configuration
- PersistentVolumeClaim (PVC) support with flexible access modes and storage classes
- External Secrets integration for secret management
- Init container support for pre-start tasks
- **Comprehensive sidecar container support** with full feature parity
  - Multiple sidecars per deployment
  - All probe types (HTTP, TCP, exec, gRPC)
  - Environment variables and secrets
  - Security context per sidecar
  - Resource limits and volume mounts
- **Advanced health probe configuration**
  - Support for httpGet, tcpSocket, exec, and grpc probes
  - Liveness, readiness, and startup probes
  - Configurable timeouts, periods, and thresholds
- **Security enhancements**
  - Pod and container security contexts with best practice defaults
  - Network Policy support for network segmentation
  - Security-hardened examples
- **Monitoring and observability**
  - Prometheus ServiceMonitor for metrics collection
  - Configurable scrape intervals and paths
- **Advanced scheduling**
  - Topology spread constraints for multi-zone deployments
  - Node affinity and anti-affinity rules
  - Pod priority classes
  - Runtime class selection (gVisor, Kata, etc.)
  - Configurable termination grace period
- **Lifecycle management**
  - preStop and postStart hooks
  - Graceful shutdown support
- **Environment configuration**
  - Simple environment variables (env)
  - Secret-backed environment variables (secretEnv)
  - ConfigMap and Secret mounting (envFrom)
- **Helm hooks** for pre/post install/upgrade tasks
- Comprehensive values.yaml documentation
- Production, development, and security-hardened example configurations

### Changed

- Enhanced service port configuration with backward compatibility
- Improved probe configuration to support all Kubernetes probe types
- Extended PVC configuration with storage class and volume mode options
- Enhanced PDB with maxUnavailable support and unhealthyPodEvictionPolicy

### Fixed

- Service port configuration now properly handles targetPort and protocol
- PVC template now accepts enabled, create, or created flags for backward compatibility
- Container port configuration with proper fallback logic

### Security

- Added secure default security context examples
- Documented security best practices
- Added network policy templates
- Included pod security standards examples

## [Unreleased]

### Planned

- Support for StatefulSet workloads
- CronJob and Job templates
- Vertical Pod Autoscaler (VPA) support
- Custom metrics for HPA
- Keda ScaledObject support
- Certificate management via cert-manager
- Vault integration examples
- Istio and Linkerd service mesh configurations
- ArgoCD ApplicationSet examples

---

## Upgrade Notes

### From 0.0.x to 0.1.0

This release introduces many new features while maintaining backward compatibility with existing deployments.

**Breaking Changes:**

- None

**New Required Fields:**

- None (all new features are optional)

**Deprecations:**

- `port` field is deprecated in favor of `containerPort`. The `port` field will continue to work but may be removed in future versions.
- `pvc.created` is deprecated in favor of `pvc.enabled`. Both will continue to work.

**Recommended Actions:**

1. Review and update security contexts to follow best practices
2. Configure resource limits if not already set
3. Enable PodDisruptionBudget for production workloads
4. Consider adding network policies for enhanced security
5. Enable Prometheus ServiceMonitor if using Prometheus Operator
6. Review probe configurations and add startup probes for slow-starting apps

**Migration Example:**

Old configuration:

```yaml
port:
  port: 8080
  name: http

pvc:
  created: true
  size: 10Gi
```

New configuration (recommended):

```yaml
containerPort:
  port: 8080
  name: http
  protocol: TCP

pvc:
  enabled: true
  size: 10Gi
  storageClassName: fast-ssd
  accessModes:
    - ReadWriteOnce
```

The old configuration will continue to work, but we recommend migrating to the new format for better clarity and additional features.

## Support

For questions, issues, or contributions:

- GitHub Issues: <https://github.com/ElloTechnology/helm-charts/issues>
- GitHub Discussions: <https://github.com/ElloTechnology/helm-charts/discussions>
