## Logging

**Inhalt:**

- Automatische Erzeugung von definierten Logeinträgen
- Auf die Standardausgabe (stdio) und nach Fluentd

#### 1. Logeintrag definieren 

```bash
kubectl apply -f istio-explore/logging/logentry.yaml
```

**logentry instance** - mapping von Attributen aus Request-Metadaten, Kontext etc. auf benutzerdefinierte Variablen

```yaml
apiVersion: "config.istio.io/v1alpha2"
kind: logentry
metadata:
  name: newlog
  namespace: istio-system
spec:
  severity: '"info"'
  timestamp: request.time | context.time | timestamp("2000-01-01T00:00:00Z") # default time
  variables:
    source: source.labels["app"] | source.workload.name | "unknown"
    user: source.user | "unknown"
    destination: destination.labels["app"] | destination.workload.name | "unknown"
    responseCode: response.code | 0
    responseSize: response.size | 0
    latency: response.duration | "0ms"
    protocol: request.scheme | context.protocol | "unknown" # results in http, https or tcp
  monitored_resource_type: '"UNSPECIFIED"'
```

[Liste der möglichen Attribute](https://istio.io/docs/reference/config/policy-and-telemetry/attribute-vocabulary/) (`request.`, `response.`, `source.`, ...)

#### 2. Logs auf der Standardausgabe generieren

```bash
kubectl apply -f istio-explore/logging/stdio.yaml
```

**stdio handler** - generiert die Logeinträge und schreibt sie in stdio

```yaml
apiVersion: "config.istio.io/v1alpha2"
kind: stdio
metadata:
  name: stdio-handler
  namespace: istio-system
spec:
 severity_levels:
   info: 0 # Params.Level.INFO
   warning: 1 # Params.Level.WARNING
 outputAsJson: true
```

**rule** - mappt stdio-handler <--> logentry-instance 

```yaml
apiVersion: "config.istio.io/v1alpha2"
kind: rule
metadata:
  name: newlogstdio
  namespace: istio-system
spec:
  match: destination.labels["app"]!="telemetry" # match only for non-telemetry-calls
  actions:
   - handler: stdio-handler.stdio
     instances:
     - newlog.logentry
```

### 3. Logs anzeigen

```shell
kubectl -n istio-system logs $(kubectl -n istio-system get pods -l istio-mixer-type=telemetry -o jsonpath='{.items[0].metadata.name}') -c mixer | grep \"instance\":\"newlog.logentry.istio-system\"
```



Beispiel:

```json
{
  "level": "info",
  "time": "2019-01-15T09:18:00.994485Z",
  "instance": "newlog.logentry.istio-system",
  "destination": "frontend",
  "latency": "202.081643ms",
  "protocol": "http",
  "responseCode": 200,
  "responseSize": 15699,
  "source": "loadgenerator",
  "user": "unknown"
  },
{
  "level": "info",
  "time": "2019-01-15T09:18:01.147674Z",
  "instance": "newlog.logentry.istio-system",
  "destination": "cartservice",
  "latency": "6.094983ms",
  "protocol": "http",
  "responseCode": 200,
  "responseSize": 37,
  "source": "frontend",
  "user": "unknown"
  },
{
  "level": "info",
  "time": "2019-01-15T09:18:01.205250Z",
  "instance": "newlog.logentry.istio-system",
  "destination": "cartservice",
  "latency": "8.506792ms",
  "protocol": "http",
  "responseCode": 200,
  "responseSize": 53,
  "source": "frontend",
  "user": "unknown"
  }
```



### 4. Logging mit Fluentd, Kibana, Elasticsearch

https://istio.io/docs/tasks/telemetry/fluentd/

1. **Installiere Fluentd, Kibana, Elasticsearch**

  ```
k apply -f kubernetes-manifests/logging.yaml
  ```

2. **Installiere fluentd-istio**

  ```
kubectl apply -f istio-explore/logging/logentry.yaml
kubectl apply -f istio-explore/logging/fluentd.yaml
  ```

**stdio handler** - generiert die Logeinträge und schreibt sie in fluentd

```yaml
apiVersion: "config.istio.io/v1alpha2"
kind: fluentd
metadata:
  name: handler
  namespace: istio-system
spec:
  address: "fluentd-es.logging:24224"
```

**rule** - mappt fluentd-Handler auf logentry-instance 

```yaml
apiVersion: "config.istio.io/v1alpha2"
kind: rule
metadata:
  name: new-log-fluentd
  namespace: istio-system
spec:
  # match: "true"
  match: destination.labels["app"]!="telemetry" # match only for non-telemetry and non-tcp calls
  actions:
   - handler: handler.fluentd
     instances:
     - newlog.logentry
```

3. **Logs in Kibana ansehen**

port-forward zu Kibana

  ```
kubectl -n logging port-forward deployment/kibana 5601:5601 &
  ```

- Klicke ein bisschen im Hipstershop herum

- Öffne http://localhost:5601 im Browser

- Klicke auf “Set up index patterns” und trage  `*` ein. 

- Wähle im nächsten Schritt `@timestamp` als Time Filter und klicke “Create index pattern.”

- Klicke links auf “Discover”. Es sollten Logs zu sehen sein.

  

### 5. Stackdriver?

https://istio.io/blog/2018/export-logs-through-stackdriver/

https://github.com/istio/istio/blob/master/mixer/adapter/stackdriver/operatorconfig/stackdriver.yaml

https://istio.io/docs/reference/config/policy-and-telemetry/adapters/stackdriver/

