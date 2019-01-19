### Delay Injection

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



