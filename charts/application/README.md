# application

This chart deploys an application as a Kubernetes `Deployment`, exposes it with a `Service`, and optionally provisions `PersistentVolumeClaim`s for state.

Both resources use `name` as their name and the `app: <name>` label connects the Service to the Deployment's pods.

## Values

| Key | Description |
| --- | --- |
| `name` | Name used for the Deployment, Service, container, and `app` label |
| `replicas` | Number of pod replicas |
| `image.pullSecret` | Image pull secret name; omit or leave empty when it is not needed |
| `image.package` | Required container image repository |
| `image.tag` | Required container image tag |
| `env` | List of environment variables |
| `volumes` | List of persistent volumes to mount; omit when the application is stateless |
| `service.port` | Port exposed by the Service |
| `service.targetPort` | Container port to which the Service forwards traffic |

Each `env` entry can contain either a literal `value` or a `secret` reference:

```yaml
env:
  - name: LOG_LEVEL
    value: info
  - name: DATABASE_URL
    secret:
      store: my-app-secrets # Secret name
      key: database-url # Key within the Secret
```

Each `volumes` entry creates a `ReadWriteOnce` PersistentVolumeClaim named `<name>-<volume name>` and mounts it into the container:

```yaml
volumes:
  - name: data
    mountPath: /data
    size: 1Gi
    storageClass: local-path # Optional; the cluster default when omitted
```

When volumes are set, the Deployment uses the `Recreate` strategy so the old pod releases the volume before the new one starts. Claims carry `helm.sh/resource-policy: keep`, so uninstalling the release does not delete the data.

## Example values

See [values.yaml](values.yaml) for defaults and commented examples. To deploy your own image, create an override file:

```yaml
name: my-app

replicas: 2

image:
  package: ghcr.io/vuphu/my-app
  tag: "1.2.3"
  pullSecret: ghcr-pull-secret

env:
  - name: PORT
    value: "8080"

service:
  port: 80
  targetPort: 8080
```

## Migrating from 1.x

Version 2.0.0 removes the `deployment` wrapper. Move `replicas`, `env`, and `volumes` to the root, rename `deployment.ips` to `image.pullSecret`, and replace `deployment.container.package` / `deployment.container.tag` with `image.package` / `image.tag`. The `service` values keep their existing structure.
