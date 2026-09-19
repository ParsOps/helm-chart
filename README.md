# Parsops Helm Chart

A Helm chart for deploying applications on Kubernetes.

It supports Deployments and StatefulSets, multiple containers, Services, Ingress, and Horizontal Pod Autoscaling. You can also run more than one deployment or service from a single release.

## Prerequisites

- Kubernetes 1.19+
- Helm 3+

## Install

```bash
helm install my-release ./parsops-chart
```

Use your own values file:

```bash
helm install my-release ./parsops-chart -f my-values.yaml
```

## Upgrade

```bash
helm upgrade my-release ./parsops-chart -f my-values.yaml
```

## Uninstall

```bash
helm uninstall my-release
```

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
