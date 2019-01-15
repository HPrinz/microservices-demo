### Policies

#### Whitelist Checking

adapter | **listchecker** - Prüft einen Eingabewert gegen eine white/blacklist

```yaml
apiVersion: config.istio.io/v1alpha2
kind: listchecker
metadata:
  name: staticversion
  namespace: istio-system
spec:
  providerUrl: http://white_list_registry/
  blacklist: false
```

#### 