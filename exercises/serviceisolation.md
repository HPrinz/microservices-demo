#### Abweisung aller Anfragen an einen Service (Service Isolation)

- Für Testzwecke

```shell
kubectl apply -f istio-explore/routing/deny-currencyservice.yaml
```

**denier** - Adapter, der alle Anfragen mit den google.rpc.Code 7 (PERMISSION_DENIED) abweist

```yaml
apiVersion: config.istio.io/v1alpha2
kind: denier
metadata:
  name: deny
  namespace: istio-system
spec:
  status:
    code: 7
    message: Not allowed
```

**checknothing** - Die (hier leeren) Daten, die dem denier-Adapter übergeben werden

```yaml
apiVersion: config.istio.io/v1alpha2
kind: checknothing
metadata:
  name: denyrequest
  namespace: istio-system
spec:
```

**rule** - greift, wenn der requests an `emailservice` gestellt werden und weist sie durch Anwendung des Handlers ab. Dem Handler werden keine Daten übergeben (checknothing).

```yaml
apiVersion: config.istio.io/v1alpha2
kind: rule
metadata:
  name: deny-emailservice-service
  namespace: istio-system
spec:
  match: destination.service=="emailservice.default.svc.cluster.local"
  actions: #then
  - handler: deny.denier
    instances:
    - denyrequest.checknothing
```

#### Abweisung aller curl-Requests

**rule** - greift, wenn der user-agent des requests mit `curl` beginnt und wendet den Handler mit leeren Daten (checknothing) an

```yaml
apiVersion: config.istio.io/v1alpha2
kind: rule
metadata:
  name: mixer-deny-curl
  namespace: istio-system
spec:
  match: match(request.headers["user-agent"], "curl*")
  actions:
  - handler: deny.denier
    instances:
    - denyrequest.checknothing
```


