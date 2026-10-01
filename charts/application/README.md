# application

This chart deploys a single containerized application as a Kubernetes `Deployment` and exposes it with a `Service`, with optional `PersistentVolumeClaim`s for state.

Both resources use `name` as their name and the `app: <name>` label connects the Service to the Deployment's pods.

## Values

| Key | Description |
| --- | --- |
| `name` | Name used for the Deployment, Service, container, and `app` label |
| `deployment.replicas` | Number of pod replicas |
| `deployment.ips` | Image pull secret name; omit or leave empty when it is not needed |
| `deployment.container.package` | Container image repository |
| `deployment.container.tag` | Container image tag |
| `deployment.env` | List of environment variables |
| `deployment.volumes` | List of persistent volumes to mount; omit when the application is stateless |
| `service.port` | Port exposed by the Service |
| `service.targetPort` | Container port to which the Service forwards traffic |

Each `deployment.env` entry can contain either a literal `value` or a `secret` reference:

```yaml
deployment:
  env:
    - name: LOG_LEVEL
      value: info
    - name: DATABASE_URL
      secret:
        store: my-app-secrets # Secret name
        key: database-url # Key within the Secret
```

Each `deployment.volumes` entry creates a `ReadWriteOnce` PersistentVolumeClaim named `<name>-<volume name>` and mounts it into the container:

```yaml
deployment:
  volumes:
    - name: data
      mountPath: /data
      size: 1Gi
      storageClass: local-path # Optional; the cluster default when omitted
```

When volumes are set, the Deployment uses the `Recreate` strategy so the old pod releases the volume before the new one starts. Claims carry `helm.sh/resource-policy: keep`, so uninstalling the release does not delete the data.

## Example values

Create a `values.yaml` file:

```yaml
name: my-app

deployment:
  replicas: 2
  ips: ghcr-pull-secret
  container:
    package: ghcr.io/vuphu/my-app
    tag: "1.2.3"
  env:
    - name: PORT
      value: "8080"

service:
  port: 80
  targetPort: 8080
```
