## datou-k8-istio
---

### Summary
The goal of this project is to provide a guided walkthrough for Istio.

### Pre-requisite

You will need a cluster to run the steps in this project. If you are following the datou series, you will need these projects completed.

- datou-k8 - [set up a local cluster](https://github.com/datou-tech/datou-k8)

### Networking Primer

To get the full picture, an understanding of how networking works in a containerized world is important. 

*Coming Soon*

### Install Istio

1. Install Istio CLI - `brew install istioctl`
1. Install Istio Operator to cluster via istioctl (allows for declarative state of Istio) - `istioctl operator init`
1. Create the namespace for Istio resources - `kubectl create ns istio-system`
1. Install the default profile - 
```
kubectl apply -f - <<EOF
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  namespace: istio-system
  name: example-istiocontrolplane
spec:
  profile: default
EOF
```
5. Check the svc and pods for the installation `kubectl get svc -n istio-system` & `kubectl get pods -n istio-system`
6. Label the default namespace with istio - this activates the service mesh capabilities for this namespace - `kubectl label namespace default istio-injection=enabled --overwrite`

### Network Observability With Istio

1. Install Prometheus for metrics collection - `kubectl apply -f https://raw.githubusercontent.com/istio/istio/master/samples/addons/prometheus.yaml`
1. Install Grafana for a general metrics dashboard - `kubectl apply -f https://raw.githubusercontent.com/istio/istio/master/samples/addons/grafana.yaml`
1. Install Jaeger for general tracing instrumentation - `kubectl apply -f https://raw.githubusercontent.com/istio/istio/master/samples/addons/grafana.yaml`
1. Install Kiali by starting with a new namespace for the Kiali Operator - `kubectl create namespace kiali-operator`
1. Install Kiali Operator via Helm (the anonymous auth strategy is not to be used in a Production environment)
```
helm install \
    --set cr.create=true \
    --set cr.namespace=istio-system \
    --set auth.strategy="anonymous" \
    --namespace kiali-operator \
    --repo https://kiali.org/helm-charts \
    kiali-operator \
    kiali-operator
```
6. Log into dashboard - `istioctl dashboard kiali`


### Test Traffic

1. Install echo-server examples from the [datou-k8](https://github.com/datou-tech/datou-k8#exercise---first-deployment-14) project
1. Run this script to send traffic to pod - `for ((i=1;i<=10000;i++)); do   curl -v --header "Connection: keep-alive" "localhost:8080/user?uuid=52010&model_id=20&attr=0"; sleep 3; done`
1. Watch the traffic route on the [dashboard](http://localhost:59934/kiali/console/graph/namespaces/?edges=noLabel&graphType=versionedApp&namespaces=default&idleNodes=true&duration=60&refresh=10000&operationNodes=false&idleEdges=false&injectServiceNodes=true&layout=dagre)
1. TODO: circuit breaking example
1. TODO: traffic mirroring example
1. TODO: fault injection example
