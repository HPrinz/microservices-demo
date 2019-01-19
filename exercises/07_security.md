### Security | mTLS

#### 1. mTLS für alle Services anschalten

```
kubectl apply -f istio-explore/security/mtls.yaml
```

**Policy** - Adservice kann nur mit mTLS-Verbindungen angefragt werden (?)

```yaml
apiVersion: "authentication.istio.io/v1alpha1"
kind: Policy
metadata:
  name: adservice-policy
spec:
  targets:
  - name: adservice
  peers:
  - mtls:
      mode: STRICT
```

**DestinationRule** - Automatisches mTLS bei Anfrage von Adservice (?)

```yaml
apiVersion: "networking.istio.io/v1alpha3"
kind: DestinationRule
metadata:
  name: adservice-istio-client-mtls
spec:
  host: adservice.default.svc.cluster.local
  trafficPolicy:
    tls:
      mode: ISTIO_MUTUAL
```

Fehler!

#### 2. mTLS im Frontend optional machen

```
kubectl apply -f istio-explore/security/mtls-frontend.yaml
```

**Policy** - mTLS für Frontend-Service optional (PERMISSIVE)

```yaml
apiVersion: "authentication.istio.io/v1alpha1"
kind: Policy
metadata:
  name: frontend-policy
spec:
  targets:
  - name: frontend
  peers:
  - mtls:
      mode: PERMISSIVE
```

**Policy** - mTLS für Frontend-Gateway optional (PERMISSIVE)

```yaml
apiVersion: "authentication.istio.io/v1alpha1"
kind: Policy
metadata:
  name: frontend-external-policy
spec:
  targets:
  - name: frontend-external
  peers:
  - mtls:
      mode: PERMISSIVE # ISTIO_MUTUAL
```
