# application

This chart deploys a single containerized application as a Kubernetes `Deployment` and exposes it with a `Service`.

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
