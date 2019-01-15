### Metriken

[TOC]

#### 1. Prometheus-Adapter

**prometheus** - Konsumiert Metriken und aggregiert sie als konfigurierbare Distributions/Histogramme oder Counter.

```yaml
apiVersion: config.istio.io/v1alpha2
kind: prometheus
metadata:
  name: handler
  namespace: istio-system
spec:
  metrics:
  - name: request_count
    instance_name: requestcount.metric.istio-system
    kind: COUNTER
    label_names:
    - destination_service
    - destination_version
    - response_code
  - name: request_duration
    instance_name: requestduration.metric.istio-system
    kind: DISTRIBUTION
    label_names:
    - destination_service
    - destination_version
    - response_code
    buckets:
      explicit_buckets:
        bounds: [0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10]
```

#### 2. Modell für die Requestduration-Metrik überschreiben

> TODO ergänzen?
>
> TODO: wiederherstellen?

**metric** - Überschreibung der default requestduration-Metrik

```yaml
apiVersion: config.istio.io/v1alpha2
kind: metric
metadata:
  name: requestduration
  namespace: istio-system
spec:
  value: response.duration | "0ms"
  dimensions:
    destination_service: destination.service | "unknown"
    destination_version: destination.labels["version"] | "unknown"
    response_code: response.code | 000
  monitored_resource_type: '"UNSPECIFIED"'
```

**metric** - überschreibt die default requestcount-Metrik

```yaml
apiVersion: config.istio.io/v1alpha2
kind: metric
metadata:
  name: requestcount
  namespace: istio-system
spec:
  value: "1"
  dimensions:
    destination_service: destination.service | "unknown"
    destination_version: destination.labels["version"] | "whatever"
    response_code: response.code | 000
  monitored_resource_type: '"UNSPECIFIED"'
```

#### 3. Metriken auf den Prometheus-Adapter mappen, wenn productcatalogservice angefragt wird

**rule** - Liefert die `requestduration`-Metrik an den `prometheus`-Handler wenn der Ziel-Service `productcatalogservice` ist.

```yaml
apiVersion: config.istio.io/v1alpha2
kind: rule
metadata:
  name: promhttp
  namespace: istio-system
spec:
  match: destination.service == "productcatalogservice.default.svc.cluster.local" 
  actions:
  - handler: handler.prometheus
    instances:
    - requestduration.metric.istio-system
    - requestcount.metric.istio-system
```

> TODO:  ` && request.headers["x-user"] == "user1"`



> TODO: raus?

**rule** -sendet default Metriken zu empfangenen und gesendeten Bytes in TCP-Anfragen an Prometheus

```yaml
apiVersion: config.istio.io/v1alpha2
kind: rule
metadata:
  name: promtcp
  namespace: istio-system
spec:
  match: context.protocol == "tcp"
  actions:
  - handler: handler.prometheus
    instances:
    - tcpbytesreceived.metric
    - tcpbytessent.metric
```

> sum(rate(istio_request_count[10m])) by (destination_service)
>
> sum(istio_request_count) by (destination_service)