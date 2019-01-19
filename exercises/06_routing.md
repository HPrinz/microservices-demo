

## Routing

[TOC]

### Teil 1: Routing

#### 1. Zweite Version des Frontends deployen

```shell
kubectl apply -f istio-explore/routing/frontend_v2.yaml
```

**Deployment** - für alternative Version des Frontend mit Label "version: 2"

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: frontend2
spec:
  template:
    metadata:
      labels:
        app: frontend
        version: "2"
    spec:
      containers:
        - name: server
          image: gcr.io/hanna-prinz/frontend:c0ade69 # anderer Tag
          ports:
          - containerPort: 8080
          env:
            [...]
```

#### 2. DestinationRole

```
kubectl apply -f istio-explore/routing/frontend-destination-role.yaml
```

**DestinationRule** - Mappt Labels auf "Subsets"

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: frontend
spec:
  host: frontend
  subsets:
  - name: v1
    labels:
      version: "1"
  - name: v2
    labels:
      version: "2"
```

#### 3. Canary Release

```shell
kubectl apply -f istio-explore/routing/frontend-canary.yaml
```

**VirtualService** - routet 80% auf v1 und 20% auf v2

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: frontend
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
        host: frontend
        subset: v1
      weight: 20
    - destination:
        host: frontend
        subset: v2
      weight: 80
```



#### 4. Canary Release für Chrome Browser

```
kubectl apply -f istio-explore/routing/frontend-canary-chrome.yaml
```



**VirtualService** - Routet alle Chrome User-Agents auf v2

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: frontend
spec:
  hosts:
  - "*"
  gateways:
  - frontend-gateway
  http:
  - match:
    - uri:
        prefix: /
      headers:
        user-agent:
          regex: ".*Chrome.*"
    route:
    - destination:
        host: frontend
        subset: v2
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        host: frontend
        subset: v1
```

#### 5. A/B Deployment

TODO

#### 6. Load Balancing?

TODO



### Teil 2: Service Isolation

Abweisung aller Anfragen an einen Service für Testzwecke

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

