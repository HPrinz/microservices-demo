### Fault Injection



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

####  2. Cleanup

```shell
kubectl delete -f istio-explore/fault_injection.yaml
```

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



