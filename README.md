# Hipster Shop - Istio Demo

## Setup

### Tools installieren

1. **gcloud installieren**

   ```shell
   curl https://sdk.cloud.google.com | bash
   exec -l $SHELL
   ```

2. **kubectl installieren**

   ```shell
   gcloud components install kubectl
   ```

### Cluster auf GKE aufsetzen

1. Google Cloud Benutzerkonto auf https://cloud.google.com/ erstellen

2. Den folgenden Befehl in Konsole ausführen, um sich zu authentifizieren

   ```shell
   gcloud init
   ```

3. Die Region & Zone  für das Rechenzentrum wählen

   ```shell
   gcloud config set compute/region europe-west3 # Frankfurt
   gcloud config set compute/zone europe-west3-c
   ```

4. Cluster mit dem Namen `istio-demo` erstellen

   ```shell
   gcloud container clusters create istio-demo \
      --num-nodes=3 --cluster-version latest \
      --machine-type n1-standard-2 --enable-autoupgrade \
      --scopes cloud-platform
   ```

5. Dem aktuellen Nutzer Admin-Rechte für das Cluster geben (notwendig für Istio)

   ```shell
   kubectl create clusterrolebinding cluster-admin-binding \
        --clusterrole=cluster-admin \
        --user=$(gcloud config get-value core/account)
   ```

### Istio installieren

1. Istio herunterladen


```sh
export ISTIO_VERSION=1.1.3
curl -L https://git.io/getLatestIstio | sh -
export ISTIO=$PWD/istio-$ISTIO_VERSION
export PATH=$PATH:$ISTIO/bin
```

2. Kubernetes Namespace für Istio erstellen

```sh
kubectl create namespace istio-system
```

3. CRDs installieren

```sh
for i in $ISTIO/install/kubernetes/helm/istio-init/files/crd*yaml; do kubectl apply -f $i; done
```

4. Kiali Passwort und Benutzername festlegen und in Kubernetes Secrets ablegen

```sh
KIALI_USERNAME=$(read '?Kiali Username: ' uval && echo -n $uval | base64)
KIALI_PASSPHRASE=$(read -s "?Kiali Passphrase: " pval && echo -n $pval | base64)

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: kiali
  namespace: istio-system
  labels:
    app: kiali
type: Opaque
data:
  username: $KIALI_USERNAME
  passphrase: $KIALI_PASSPHRASE
EOF
```

5. Eine Istio-Installationskonfiguration für Kubernets mit helm auf Basis von `values.yaml` erstellen

```sh
helm template $ISTIO/install/kubernetes/helm/istio --values istio-values.yaml \
 --namespace istio-system --name istio > custom-istio.yaml
```

6. Die Konfiguration auf dem Cluster anwenden

```sh
 kubectl apply -f custom-istio.yaml
```

7. Dem Standard-Namespace `default` für automatische Injektion von Sidecars markieren

```sh
 kubectl label namespace default istio-injection=enabled
```

8. Die Kubernetes Konfiguration für das Deployment des Hipster Shops anwenden

```sh
kubectl apply -f kubernetes-manifests
```

9. Die Eintritts-IP-Adresse des `istio-ingressgateways` herausfinden und ausgeben

```sh
INGRESS_HOST="$(kubectl -n istio-system get service istio-ingressgateway \
-o jsonpath='{.status.loadBalancer.ingress[0].ip}')"
echo "$INGRESS_HOST"
```

10. Alle eingehenden Verbindungen und die ausgehende Verbindung zur EZB mit Istio-Konfiguration erlauben 

```sh
kubectl apply -f istio-explore/01_setup/frontend-gateway.yaml
kubectl apply -f istio-explore/01_setup/whitelist-egress-currencyprovider.yaml
```

Weitere Istio-Konfigurationen im Ordner `istio-explore`

