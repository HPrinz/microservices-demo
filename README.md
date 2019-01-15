# Hipster Shop: Cloud-Native Microservices Demo Application

This project contains a 10-tier microservices application. The application is a
web-based e-commerce app called **“Hipster Shop”** where users can browse items,
add them to the cart, and purchase them.

**Google uses this application to demonstrate Kubernetes, GKE, Istio,
Stackdriver, gRPC, OpenCensus** and similar cloud-native technologies.

> **Note to Googlers:** Please fill out the form at
[go/microservices-demo](http://go/microservices-demo) if you are using this
application.


## Screenshots

| Home Page | Checkout Screen |
|-----------|-----------------|
| [![Screenshot of store homepage](./docs/img/hipster-shop-frontend-1.png)](./docs/img/hipster-shop-frontend-1.png) | [![Screenshot of checkout screen](./docs/img/hipster-shop-frontend-2.png)](./docs/img/hipster-shop-frontend-2.png) |

## Service Architecture

**Hipster Shop** is composed of many microservices written in different
languages that talk to each other over gRPC.

[![Architecture of
microservices](./docs/img/architecture-diagram.png)](./docs/img/architecture-diagram.png)

Find **Protocol Buffers Descriptions** at the [`./pb` directory](./pb).

| Service | Language | Description |
|---------|----------|-------------|
| [frontend](./src/frontend) | Go | Exposes an HTTP server to serve the website. Does not require signup/login and generates session IDs for all users automatically. |
| [cartservice](./src/cartservice) |  C# | Stores the items in the user's shipping cart in Redis and retrieves it. |
| [productcatalogservice](./src/productcatalogservice) | Go | Provides the list of products from a JSON file and ability to search products and get individual products. |
| [currencyservice](./src/currencyservice) | Node.js | Converts one money amount to another currency.  Uses real values fetched from European Central Bank. It's the highest QPS service. |
| [paymentservice](./src/paymentservice) | Node.js | Charges the given credit card info (hypothetically😇) with the given amount and returns a transaction ID. |
| [shippingservice](./src/shippingservice) | Go | Gives shipping cost estimates based on the shopping cart. Ships items to the given address (hypothetically😇) |
| [emailservice](./src/emailservice) | Python | Sends users an order confirmation email (hypothetically😇). |
| [checkoutservice](./src/checkoutservice) | Go | Retrieves user cart, prepares order and orchestrates the payment, shipping and the email notification. |
| [recommendationservice](./src/recommendationservice) | Python | Recommends other products based on what's given in the cart. |
| [adservice](./src/adservice) | Java | Provides text ads based on given context words. |
| [loadgenerator](./src/loadgenerator) | Python/Locust | Continuously sends requests imitating realistic user shopping flows to the frontend. |


## Features

- **[Kubernetes](https://kubernetes.io)/[GKE](https://cloud.google.com/kubernetes-engine/):**
  The app is designed to run on Kubernetes (both locally on "Docker for
  Desktop", as well as on the cloud with GKE).
- **[gRPC](https://grpc.io):** Microservices use a high volume of gRPC calls to
  communicate to each other.
- **[Istio](https://istio.io):** Application works on Istio service mesh.
- **[OpenCensus](https://opencensus.io/) Tracing:** Most services are
  instrumented using OpenCensus trace interceptors for gRPC/HTTP.
- **[Stackdriver APM](https://cloud.google.com/stackdriver/):** Many services
  are instrumented with **Profiling**, **Tracing** and **Debugging**. In
  addition to these, using Istio enables features like Request/Response
  **Metrics** and **Context Graph** out of the box. When it is running out of
  Google Cloud, this code path remains inactive.
- **[Skaffold](https://github.com/GoogleContainerTools/skaffold):** Application
  is deployed to Kubernetes with a single command using Skaffold.
- **Synthetic Load Generation:** The application demo comes with a background
  job that creates realistic usage patterns on the website using
  [Locust](https://locust.io/) load generator.

## Installation

> **Note:** that the first build can take up to 20-30 minutes. Consequent builds
> will be faster.

### Option 1: Running locally with “Docker for Desktop”

> 💡 Recommended if you're planning to develop the application.

1. Install tools to run a Kubernetes cluster locally:

   - kubectl (can be installed via `gcloud components install kubectl`)
   - Docker for Desktop (Mac/Windows): It provides Kubernetes support as [noted
     here](https://docs.docker.com/docker-for-mac/kubernetes/).
   - [skaffold](https://github.com/GoogleContainerTools/skaffold/#installation)

1. Launch “Docker for Desktop”. Go to Preferences and choose “Enable Kubernetes”.

1. Run `kubectl get nodes` to verify you're connected to “Kubernetes on Docker”.

1. Run `skaffold run` (first time will be slow, it can take ~20-30 minutes).
   This will build and deploy the application. If you need to rebuild the images
   automatically as you refactor he code, run `skaffold dev` command.

1. Run `kubectl get pods` to verify the Pods are ready and running. The
   application frontend should be available at http://localhost:80 on your
   machine.

### Option 2: Deploy on Google Kubernetes Engine with Istio

>  **Note:** you followed GKE deployment steps above, run `skaffold delete` first
> to delete what's deployed.

>  (Optional) If you'd like to enable **mTLS** in the demo app, you need to
> make a few changes to the deployment manifests:
>
> - `kubernetes-manifests/frontend.yaml`: delete "livenessProbe" and
>   "readinessProbe" fields.
> - kubernetes-manifests/loadgenerator.yaml`: delete "initContainers" field.

#### Create Cluster

```shell
# set region & zone
gcloud config set compute/region europe-west3 # Frankfurt
gcloud config set compute/zone europe-west3-c

# enable container & registry
gcloud services enable container.googleapis.com containerregistry.googleapis.com

gcloud container clusters create istio-workshop \
      --enable-autoscaling --min-nodes=2 --max-nodes=3 --num-nodes=2 \
      --machine-type n1-standard-2 --enable-autoupgrade \
      --scopes cloud-platform

gcloud auth configure-docker -q # no cloudshell

kubectl create clusterrolebinding cluster-admin-binding \
    --clusterrole=cluster-admin \
    --user=$(gcloud config get-value core/account)
```



#### Option 1: Install Istio via native GKE features

1. Create a GKE cluster (described above).

2. Use [Istio on GKE add-on](https://cloud.google.com/istio/docs/istio-on-gke/installing)
   to install Istio to your existing GKE cluster.

   ```
   gcloud beta container clusters update demo \
       --zone=us-central1-a \
       --update-addons=Istio=ENABLED \
       --istio-config=auth=MTLS_PERMISSIVE
   ```

3. (Optional) Enable Stackdriver Tracing/Logging with Istio Stackdriver Adapter
   by [following this guide](https://cloud.google.com/istio/docs/istio-on-gke/installing#enabling_tracing_and_logging).

4. Apply the manifests in [`./istio-manifests`](./istio-manifests) directory.

   ```
   kubectl apply -f ./istio-manifests
   ```

    This is required only once.



#### Option 2: Install Istio on plain GKE cluster

1. Install Istio without mTLS

   ```shell
   kubectl apply -f kiali/kiali-secrets.yaml
   
   cd $HOME
   curl -L https://git.io/getLatestIstio | sh -
   cd istio-1.0.5
   kubectl apply -f install/kubernetes/helm/istio/templates/crds.yaml
   kubectl create namespace istio-system
   helm template install/kubernetes/helm/istio \
     --set grafana.enabled=true --set tracing.enabled=true \
     --name istio --namespace istio-system > $HOME/istio.yaml
   kubectl apply -f $HOME/istio.yaml
   kubectl label namespace default istio-injection=enabled
   
   kubectl -n istio-system port-forward deployment/istio-tracing 16686:16686 &
   kubectl -n istio-system port-forward deployment/grafana 3000:3000 &
   
   kubectl apply -f ./istio-manifests
   
   helm template $HOME/istio-1.0.5/install/kubernetes/helm/istio \
       --set grafana.enabled=true --set tracing.enabled=true --set kiali.enabled=true \
       --set "kiali.dashboard.jaegerURL=http://localhost:16686" \
       --set "kiali.dashboard.grafanaURL=http://localhost:3000" \
       --name istio --namespace istio-system > $HOME/istio.yaml
   kubectl apply -f $HOME/istio.yaml
   
   kubectl -n istio-system port-forward deployment/kiali 20001:20001 &
   open http://localhost:20001
   ```




#### Both

1. In the root of this repository, run `skaffold run --default-repo=gcr.io/[PROJECT_ID]`, where [PROJECT_ID] is your GCP project ID.

   This command:

   - builds the container images
   - pushes them to GCR
   - applies the `./kubernetes-manifests` deploying the application to
     Kubernetes.

   **Troubleshooting:** If you get "No space left on device" error on Google Cloud Shell,
   you can build the images on Google Cloud Build:
   [Enable the Cloud Build API](https://console.cloud.google.com/flows/enableapi?apiid=cloudbuild.googleapis.com), then run `skaffold run -p gcb --default-repo=gcr.io/[PROJECT_ID]` instead.

2. Run `kubectl get pods` to see pods are in a healthy and ready state.

3. Find the IP address of your istio gateway Ingress or Service, and visit the
   application.

       INGRESS_HOST="$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')"
       
       echo "$INGRESS_HOST"
       
       curl -v "http://$INGRESS_HOST"

   without istio use

   ```shell
   kubectl get service frontend-external
   ```

   

**Troubleshooting:** A Kubernetes bug (will be fixed in 1.12) combined with a Skaffold [bug](https://github.com/GoogleContainerTools/skaffold/issues/887) causes load balancer to not to work even after getting an IP address. If you are seeing this, run `kubectl get service frontend-external -o=yaml | kubectl apply -f-` to trigger load balancer reconfiguration.



## Conferences featuring Hipster Shop 

- [Google Cloud Next'18 London – Keynote](https://youtu.be/nIq2pkNcfEI?t=3071) showing Stackdriver Incident Response Management
- Google Cloud Next'18 SF
  - [Day 1  Keynote](https://youtu.be/vJ9OaAqfxo4?t=2416) showing GKE On-Prem
  - [Day 3 – Keynote](https://youtu.be/JQPOPV_VH5w?t=815) showing Stackdriver APM (Tracing, Code Search, Profiler, Google Cloud Build)
  - [Introduction to Service Management with Istio](https://www.youtube.com/watch?v=wCJrdKdD6UM&feature=youtu.be&t=586)

---

This is not an official Google project. 
