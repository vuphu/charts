# charts

This repository contains multiple Helm charts used internally for deploying services.

Charts are published as OCI artifacts to GitHub Container Registry at `oci://ghcr.io/vuphu/charts`.

## Charts

| Chart | Description | Path |
| --- | --- | --- |
| `application` | Generic `Deployment` + `Service` for a single containerized app | [`charts/application`](charts/application) |

## Usage

Log in to the registry (only needed if the package is private):

```sh
echo "$GITHUB_TOKEN" | helm registry login ghcr.io --username <github-user> --password-stdin
```

Install or upgrade a chart:

```sh
helm upgrade --install my-app oci://ghcr.io/vuphu/charts/application \
  --version 1.1.0 \
  -f values.yaml
```

## `application` chart

Renders a `Deployment` and a `Service`, both named after `name` and selected by the label `app: <name>`.

### Values

| Key | Description |
| --- | --- |
| `name` | Name used for the Deployment, Service, container and `app` label |
| `deployment.replicas` | Number of pod replicas |
| `deployment.ips` | Name of the image pull secret; omitted when empty |
| `deployment.container.package` | Container image repository |
| `deployment.container.tag` | Container image tag |
| `deployment.env` | List of environment variables (see below) |
| `service.port` | Port exposed by the Service |
| `service.targetPort` | Container port the Service forwards to |

Each `deployment.env` entry sets either a literal `value` or a `secret` reference:

```yaml
deployment:
  env:
    - name: LOG_LEVEL
      value: info
    - name: DATABASE_URL
      secret:
        store: my-app-secrets # Secret name
        key: database-url     # key within the Secret
```

### Example `values.yaml`

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

## Publishing

Charts are published by GitHub Actions on every push to `main`:

- [`publish-chart.yaml`](.github/workflows/publish-chart.yaml) — reusable workflow that packages a chart and pushes it to the OCI registry. If the chart version already exists in the registry, the push is skipped.
- [`publish-application.yaml`](.github/workflows/publish-application.yaml) — calls the reusable workflow for the `application` chart.

To release a new chart version, bump `version` in the chart's `Chart.yaml` and merge to `main`.

### Adding a new chart

1. Create the chart under `charts/<chart-name>/` with a `Chart.yaml` and `templates/`.
2. Add a workflow `.github/workflows/publish-<chart-name>.yaml`:

   ```yaml
   name: Publish <chart-name>

   on:
     push:
       branches:
         - main

   jobs:
     publish:
       uses: ./.github/workflows/publish-chart.yaml
       with:
         chart_name: <chart-name>
         chart_path: charts/<chart-name>
       permissions:
         contents: read
         packages: write
   ```

3. Add the chart to the table above.

### Workflow inputs

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `chart_name` | yes | — | Helm chart name |
| `chart_path` | yes | — | Directory containing `Chart.yaml` |
| `registry` | no | `ghcr.io` | OCI registry hostname |
| `registry_image` | no | `github.repository` | OCI repository path, without the hostname |

Optional secrets `registry_username` and `registry_token` default to `github.actor` and `github.token`.

## Local development

```sh
helm lint charts/application -f values.yaml
helm template my-app charts/application -f values.yaml
```
