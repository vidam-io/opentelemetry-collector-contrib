# Azure authenticator extension
<!-- status autogenerated section -->
| Status        |           |
| ------------- |-----------|
| Stability     | [alpha]  |
| Distributions | [contrib] |
| Issues        | [![Open issues](https://img.shields.io/github/issues-search/open-telemetry/opentelemetry-collector-contrib?query=is%3Aissue%20is%3Aopen%20label%3Aextension%2Fazureauth%20&label=open&color=orange&logo=opentelemetry)](https://github.com/open-telemetry/opentelemetry-collector-contrib/issues?q=is%3Aopen+is%3Aissue+label%3Aextension%2Fazureauth) [![Closed issues](https://img.shields.io/github/issues-search/open-telemetry/opentelemetry-collector-contrib?query=is%3Aissue%20is%3Aclosed%20label%3Aextension%2Fazureauth%20&label=closed&color=blue&logo=opentelemetry)](https://github.com/open-telemetry/opentelemetry-collector-contrib/issues?q=is%3Aclosed+is%3Aissue+label%3Aextension%2Fazureauth) |
| [Code Owners](https://github.com/open-telemetry/opentelemetry-collector-contrib/blob/main/CONTRIBUTING.md#becoming-a-code-owner)    | [@constanca-m](https://www.github.com/constanca-m) |

[alpha]: https://github.com/open-telemetry/opentelemetry-collector/blob/main/docs/component-stability.md#alpha
[contrib]: https://github.com/open-telemetry/opentelemetry-collector-releases/tree/main/distributions/otelcol-contrib
<!-- end autogenerated section -->

This extension implements both `extensionauth.HTTPClient` and `extensionauth.Server`, so it can be used in both exporters and receivers.

Additionally, the extension also implements `azcore.TokenCredential` so that Azure components can get the token by running the function `GetToken`. If the component supports HTTP client, then this should not be necessary, as the token will be placed in the authorization header.

> **WARNING**: Currently, the token is not cached, so if you use the extension as an implementation of `extensionauth.HTTPClient` and `extensionauth.Server`, instead of `azcore.TokenCredential`, use it with caution. Each of these requests to get a token will cause a new request being sent to Azure Identity. Under high load, be careful to not hit the service [request limits](https://learn.microsoft.com/en-us/entra/identity/users/directory-service-limits-restrictions). The requests that go through the extension for this purpose need to have the `Host` header. Read more about [request headers](https://learn.microsoft.com/en-us/azure/azure-app-configuration/rest-api-headers#request-headers).

It supports 4 different types of authentication:
- Managed identity for Azure resources
- Workload identity for Kubernetes
- Service principal with either a client secret or client certificate path for non Azure.
- And the default credentials. This is not recommended for production.

## Examples

### Managed identity

User based:
```yaml
extensions:
  azureauth:
    managed_identity:
      client_id: ${CLIENT_ID}
```

System based (leave `client_id` field empty):
```yaml
extensions:
  azureauth:
    managed_identity:
```

### Workload identity

```yaml
extensions:
  azureauth:
    workload_identity:
      client_id: ${CLIENT_ID}
      federated_token_file: ${FILE}
      tenant_id: ${TENANT_ID}
```

### Service principal

With client secret:
```yaml
extensions:
  azureauth:
    service_principal:
      client_id: ${CLIENT_ID}
      tenant_id: ${TENANT_ID}
      client_secret: ${CLIENT_SECRET}
```

With client certificate path:
```yaml
extensions:
  azureauth:
    service_principal:
      client_id: ${CLIENT_ID}
      tenant_id: ${TENANT_ID}
      client_certificate_path: ${CLIENT_CERTIFICATE_PATH}
```

### Default authentication

Not recommended for production.
```yaml
extensions:
  azureauth:
    use_default: true
```