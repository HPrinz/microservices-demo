### Quota Checking

> TODO nur frontend-calls oder alle?
>
> TODO: angewandter machen?

handler | **memquota** - maximal 5 Requests/Second

```yaml
apiVersion: config.istio.io/v1alpha2
kind: memquota
metadata:
  name: handler
  namespace: istio-system
spec:
  quotas:
  - name: requestcount.quota.istio-system
    maxAmount: 5
    validDuration: 1s
```
template | **quota** | Bemessunggrundlage

```yaml
apiVersion: "config.istio.io/v1alpha2"
kind: quota
metadata:
  name: requestcount
  namespace: istio-system
spec:
  dimensions:
    source: request.headers["x-forwarded-for"] | "unknown"
    destination: destination.labels["app"] | destination.workload.name | "unknown"
    destinationVersion: destination.labels["version"] | "unknown"
```

**rule** - Mapping von memquota-Handler und quota 

```yaml
apiVersion: config.istio.io/v1alpha2
kind: rule
metadata:
  name: quota
  namespace: istio-system
spec:
  actions:
  - handler: handler.memquota
    instances:
    - requestcount.quota
```

**QuotaSpec** - ?

```yaml
apiVersion: config.istio.io/v1alpha2
kind: QuotaSpec
metadata:
  name: request-count
  namespace: istio-system
spec:
  rules:
  - quotas:
    - charge: 1
      quota: requestcount
```

**QuotaSpecBinding** - ?

```yaml
apiVersion: config.istio.io/v1alpha2
kind: QuotaSpecBinding
metadata:
  name: request-count
  namespace: istio-system
spec:
  quotaSpecs:
  - name: request-count
    namespace: istio-system
  services:
  - name: frontend
    namespace: default
```
