### Timeout & Retry

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

