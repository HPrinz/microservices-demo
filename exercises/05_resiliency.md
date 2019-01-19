## Resilienz

[TOC]

### Teil 1: Circuit Breaker

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

> TODO Screenshot



### Teil 2: Fault Injection

**Inhalt:**

- Fault Injection sorgt dafür, dass n% der Anfragen mit einem Fehler beantwortet werden
- Kein Werkzeug um Resilienz herzustellen, sondern um sie zu testen - z.B. Testen von Circuit Breaker 

#### 1. Zum Test des Circuit Breakers: Fault Injection für Recommendation-Service

```shell
kubectl apply -f istio-explore/resilience/fault_injection.yaml
```

 **VirtualService** - gibt bei 50% der Anfragen eine 503 zurück

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: recommendationservice
spec:
  hosts:
  - "recommendationservice.default.svc.cluster.local"
  http:
  - route:
    - destination:
        host: "recommendationservice.default.svc.cluster.local"
    fault:
      abort:
        percent: 50
        httpStatus: 503
```

[Darstellung im Graph in Kiali](http://localhost:20001/console/graph/namespaces?namespaces=default)

#### 2. Cleanup

```shell
kubectl delete -f istio-explore/fault_injection.yaml
```

#### 3. Zum Test von Retry und Timeout: Delay Injection von Cartservice

```
kubectl delete -f istio-explore/delay_injection.yaml
```

**VirtualService** - verzögert alle Anfragen an cartservice um 5 Sekunden

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: cartservice
spec:
  hosts:
  - "*"
  gateways:
  - frontend-gateway
  http:
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        host: cartservice
    fault:
      delay:
        percent: 100
        fixedDelay: 5s

```



### Teil 3: Timeout & Retry

**Inhalt:**

- Timeout: Zeit nach der eine Anfrage abgebrochen wird
- Retry: Anzahl der Neuversuche bis die Anfrage als gescheitert gilt

#### 1. Timeout & Retry festlegen

```shell
kubectl apply -f istio-explore/timeout_retry.yaml
```

 **VirtualService** - wenn der recommendationservice nicht innerhalb von 2 Sekunden antwortet, wird maximal 3 mal neu versucht (?)

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: cartservice
spec:
  hosts:
  - cartservice
  http:
  - route:
    - destination:
        host: cartservice
    retries:
      attempts: 3
      perTryTimeout: 2s
```



### Teil 4: Delay Injection

- Verzögert n% der Anfragen um m Sekunden
- Testen von Retry und Timeout

#### 2. Zum Test von Retry und Timeout: Delay Injection von Cartservice

```
kubectl delete -f istio-explore/delay_injection.yaml
```

**VirtualService** - verzögert alle Anfragen an cartservice um 5 Sekunden

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: cartservice
spec:
  hosts:
  - "*"
  gateways:
  - frontend-gateway
  http:
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        host: cartservice
    fault:
      delay:
        percent: 100
        fixedDelay: 5s

```



