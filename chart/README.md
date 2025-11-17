# Flux KCL Controller Helm Chart

This Helm chart deploys the Flux KCL Controller to your Kubernetes cluster.

## Prerequisites

- Kubernetes 1.21+
- Helm 3.0+
- **Flux Source Controller** must be installed first:
  ```bash
  flux install --components=source-controller
  # OR for full Flux:
  flux install
  ```

## Installing the Chart

To install the chart with the release name `flux-kcl-controller`:

```bash
helm install flux-kcl-controller ./chart -n flux-system --create-namespace
```

## Uninstalling the Chart

To uninstall the `flux-kcl-controller` deployment:

```bash
helm uninstall flux-kcl-controller -n flux-system
```

## Configuration

The following table lists the configurable parameters of the Flux KCL Controller chart and their default values.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of controller replicas | `1` |
| `image.repository` | Controller image repository | `ghcr.io/kcl-lang/flux-kcl-controller` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `image.tag` | Image tag (overrides chart appVersion) | `""` |
| `imagePullSecrets` | Image pull secrets | `[]` |
| `nameOverride` | Override the chart name | `""` |
| `fullnameOverride` | Override the full name | `""` |
| `serviceAccount.create` | Create service account | `true` |
| `serviceAccount.annotations` | Service account annotations | `{}` |
| `serviceAccount.name` | Service account name | `""` |
| `podAnnotations` | Pod annotations | `{"prometheus.io/scrape": "true", "prometheus.io/port": "8083"}` |
| `podSecurityContext` | Pod security context | `{}` |
| `securityContext.allowPrivilegeEscalation` | Allow privilege escalation | `false` |
| `securityContext.readOnlyRootFilesystem` | Read-only root filesystem | `true` |
| `service.type` | Service type | `ClusterIP` |
| `service.port` | Service port | `8083` |
| `service.annotations` | Service annotations | `{}` |
| `serviceMonitor.enabled` | Enable Prometheus ServiceMonitor | `false` |
| `serviceMonitor.interval` | Scrape interval | `30s` |
| `serviceMonitor.scrapeTimeout` | Scrape timeout | `10s` |
| `resources.limits.cpu` | CPU limit | `1000m` |
| `resources.limits.memory` | Memory limit | `1Gi` |
| `resources.requests.cpu` | CPU request | `50m` |
| `resources.requests.memory` | Memory request | `64Mi` |
| `controller.logLevel` | Log level (debug, info, error) | `info` |
| `controller.leaderElection` | Enable leader election | `true` |
| `rbac.create` | Create RBAC resources | `true` |
| `rbac.clusterAdmin` | Grant cluster-admin privileges | `true` |
| `crds.install` | Install KCLRun CRD | `true` |
| `crds.installFluxSourceCRDs` | Install Flux Source CRDs | `false` |
| `priorityClassName` | Priority class name | `""` |
| `podDisruptionBudget.enabled` | Enable PodDisruptionBudget | `false` |
| `podDisruptionBudget.minAvailable` | Minimum available pods | `1` |
| `nodeSelector` | Node selector | `{}` |
| `tolerations` | Tolerations | `[]` |
| `affinity` | Affinity rules | `{}` |

## Examples

### Install with custom image

```bash
helm install flux-kcl-controller ./chart \
  --set image.repository=my-registry/flux-kcl-controller \
  --set image.tag=v0.8.2 \
  -n flux-system
```

### Install with Flux Source CRDs

If you don't have Flux installed and want to install the source CRDs:

```bash
helm install flux-kcl-controller ./chart \
  --set crds.installFluxSourceCRDs=true \
  -n flux-system
```

### Install with custom values file

Create a custom values file for your environment:

```yaml
# custom-values.yaml
replicaCount: 2

resources:
  limits:
    cpu: 2000m
    memory: 2Gi
  requests:
    cpu: 200m
    memory: 256Mi

serviceMonitor:
  enabled: true

podDisruptionBudget:
  enabled: true
  minAvailable: 1
```

Then install with:

```bash
helm install flux-kcl-controller ./chart \
  --values custom-values.yaml \
  -n flux-system
```

### Install with Flux Source CRDs

If you don't have Flux installed and want to install the source CRDs:

```bash
helm install flux-kcl-controller ./chart \
  --set crds.installFluxSourceCRDs=true \
  -n flux-system
```

### Install for production with high availability

```bash
helm install flux-kcl-controller ./chart \
  --set replicaCount=2 \
  --set podDisruptionBudget.enabled=true \
  --set serviceMonitor.enabled=true \
  -n flux-system
```

### Install without cluster-admin privileges

If you want more restrictive RBAC (note: this may limit functionality):

```bash
helm install flux-kcl-controller ./chart \
  --set rbac.clusterAdmin=false \
  -n flux-system
```

## Usage

After installing the chart, create a Flux source and KCLRun resource:

1. Create a GitRepository source:

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: my-kcl-source
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/your-org/your-kcl-repo
  ref:
    branch: main
```

2. Create a KCLRun resource:

```yaml
apiVersion: krm.kcl.dev.fluxcd/v1alpha1
kind: KCLRun
metadata:
  name: my-kclrun
  namespace: flux-system
spec:
  interval: 1m
  sourceRef:
    kind: GitRepository
    name: my-kcl-source
  path: ./path/to/kcl
```

## Troubleshooting

Check the controller logs:

```bash
kubectl logs -n flux-system -l app=kcl-controller
```

Check KCLRun status:

```bash
kubectl get kclrun -A
kubectl describe kclrun <name> -n <namespace>
```

## More Information

- [Flux KCL Controller](https://github.com/kcl-lang/flux-kcl-controller)
- [KCL Language](https://kcl-lang.io)
- [Flux CD](https://fluxcd.io)
