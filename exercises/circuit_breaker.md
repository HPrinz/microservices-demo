### Circuit Breaker

#### 1. Circuit Breaker für Recommendation-Service

```shell
kubectl apply -f istio-explore/circuitbreaker.yaml
```

**DestinationRule** - definiert den Fehlerzustand und die Dauer der "Pause" für den Service

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: recommendationservice
spec:
  host: recommendationservice
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100 # maximal 100 TCP-Verbindungen
      http:
        maxRequestsPerConnection: 10 # maximal 10 HTTP-Requests pro Verbindung
        http1MaxPendingRequests: 1024 # maximal 1024 pending HTTP-Requests
    outlierDetection:
      consecutiveErrors: 7 # maximal 7 aufeinander folgende Fehler ...
      interval: 5m # ... innerhalb von 5 Minuten
      baseEjectionTime: 15m # ansonsten pausiert der Service 15 Minuten
```

[Darstellung im Graph in Kiali](http://localhost:20001/console/graph/namespaces?namespaces=default)

