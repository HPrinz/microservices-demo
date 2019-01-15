### Routing

[TOC]

#### A/B Deployment

TODO

#### Load Balancing?

TODO

#### Canary Deployment

TODO

Prozentuale Verteilung auf verschiedene Label/Versionen

**DestinationRole**

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: guestbook-ui
spec:
  host: guestbook-ui
  subsets:
  - name: v1
    labels:
      version: "1.0"
  - name: v2
    labels:
      version: "2.0"
```



**VirtualService** - routet 80% auf v1 und 20% auf v2

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: guestbook-ui
spec:
  hosts:
  - "*"
  gateways:
  - guestbook-gateway
  http:
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        host: guestbook-ui
        subset: v1
      weight: 80
    - destination:
        host: guestbook-ui
        subset: v2
      weight: 20
```



#### Routing anhand des User Agents

TODO

**VirtualService** - Routet alle Chrome User-Agents auf v2

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: guestbook-ui
spec:
  hosts:
  - "*"
  gateways:
  - guestbook-gateway
  http:
  - match:
    - uri:
        prefix: /
      headers:
        user-agent:
          regex: ".*Chrome.*"
    route:
    - destination:
        host: guestbook-ui
        subset: v2
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        host: guestbook-ui
        subset: v1
```
