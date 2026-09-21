# Parsops Helm Chart

A Helm chart for deploying applications on Kubernetes.

It supports Deployments and StatefulSets, multiple containers, Services, Ingress, and Horizontal Pod Autoscaling. You can also run more than one deployment or service from a single release.

## Prerequisites

- Kubernetes 1.19+
- Helm 3+

## Install from the Helm repository

Add the ParsOps Helm repository:

```bash
helm repo add parsops https://chart.parsops.com/
helm repo update
```

Install the chart:

```bash
helm install my-release parsops/parsops-chart
```

To install a specific chart version:

```bash
helm install my-release parsops/parsops-chart --version 0.1.30
```

Use your own values file:

```bash
helm install my-release parsops/parsops-chart -f my-values.yaml
```

Search available charts and versions:

```bash
helm search repo parsops --versions
```

## Install from the local chart source

For local development or testing:

```bash
helm install my-release ./parsops-chart
```

Use your own values file:

```bash
helm install my-release ./parsops-chart -f my-values.yaml
```

## Upgrade

From the published repository:

```bash
helm repo update
helm upgrade my-release parsops/parsops-chart -f my-values.yaml
```

From the local chart source:

```bash
helm upgrade my-release ./parsops-chart -f my-values.yaml
```

## Uninstall

```bash
helm uninstall my-release
```

## Published repository

The chart repository is hosted at:

<https://chart.parsops.com/>

The Helm repository must expose `index.yaml` and packaged chart archives such as `parsops-chart-0.1.30.tgz` at this URL.

## What you can configure

| Feature | Values key | Notes |
| --- | --- | --- |
| Containers | `containers` | Image, ports, env, probes, resources |
| Replicas | `replicaCount` | Ignored when autoscaling is on |
| Deployment or StatefulSet | `statefulset.enabled` | `false` uses a Deployment |
| Multi-deployment | `deployments` | Optional; one release, many apps |
| Service | `service` or `services` | Single service or a list of services |
| Ingress | `ingress` | Disabled by default |
| Autoscaling | `autoscaling` | HPA based on CPU |
| Volumes | `volumes` / `volumeMounts` | Extra volumes for pods |
| Init containers | `initContainers` | Run before main containers |
| Service account | `serviceAccount` | Create or reuse one |
| Global defaults | `global` | Shared image, labels, resources |

Full options live in [`parsops-chart/values.yaml`](parsops-chart/values.yaml).

## Quick examples

### Basic app

```yaml
replicaCount: 2

containers:
  - name: app
    image:
      repository: my-registry/my-app
      tag: "1.0.0"
    ports:
      - name: http
        containerPort: 8080
        protocol: TCP

service:
  enabled: true
  type: ClusterIP
  ports:
    - name: http
      port: 80
      targetPort: 8080
      protocol: TCP
```

### Multiple services

Prefer the `services` list when you need more than one Service:

```yaml
services:
  - name: my-gateway
    enabled: true
    type: ClusterIP
    selector:
      component: gateway
    ports:
      - name: http
        port: 8080
        targetPort: 8080
        protocol: TCP
```

### Multiple deployments

Use `deployments` when one release should create several Deployments. Comment out or remove the top-level `replicaCount` and `containers` blocks when you use this mode.

```yaml
deployments:
  api:
    replicaCount: 2
    container:
      name: api
      image:
        repository: my-registry/api
        tag: "1.0.0"
      ports:
        - name: http
          containerPort: 8080

  worker:
    replicaCount: 1
    container:
      name: worker
      image:
        repository: my-registry/worker
        tag: "1.0.0"
      command: ["./run-worker"]
```

### StatefulSet

```yaml
statefulset:
  enabled: true
  serviceName: my-headless
```

### Autoscaling

```yaml
autoscaling:
  enabled: true
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
```

## Lint

This repo uses [yamllint](https://github.com/adrienverge/yamllint) via pre-commit:

```bash
pre-commit run yamllint --all-files
```
