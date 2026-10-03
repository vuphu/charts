# external-secret

This chart creates a namespaced External Secrets Operator `SecretStore` and an `ExternalSecret` that uses it to synchronize remote values into a Kubernetes `Secret`.

The External Secrets Operator and its `external-secrets.io/v1` custom resource definitions must already be installed in the cluster.

## Values

| Key | Description | Default |
| --- | --- | --- |
| `namespace` | Namespace for both resources; uses the Helm release namespace when empty | `""` |
| `name` | Name of the `ExternalSecret` | `external-secrets` |
| `secretStoreName` | Name of the `SecretStore` and the store reference | `external-secrets-store` |
| `targetName` | Name of the Kubernetes `Secret` created by the operator; required | `""` |
| `refreshPolicy` | External Secret refresh policy | `OnChange` |
| `refreshInterval` | Interval at which the secret is refreshed | `"0"` |
| `provider` | External Secrets provider configuration rendered under `SecretStore.spec.provider`; required | `{}` |
| `secrets` | List of remote secret keys to synchronize; required | `[]` |

Each remote key becomes a key in the target Kubernetes Secret. The chart removes a leading slash and replaces the remaining slashes with underscores. For example, `/application/database/url` becomes `application_database_url`.

## Example values

The provider object is passed directly to the External Secrets Operator. This example uses AWS Secrets Manager with workload identity:

```yaml
name: my-app-secrets
secretStoreName: aws-secrets-manager
targetName: my-app

provider:
  aws:
    service: SecretsManager
    region: ap-southeast-1
    auth:
      jwt:
        serviceAccountRef:
          name: external-secrets

secrets:
  - /my-app/database/url
  - /my-app/api/key
```
