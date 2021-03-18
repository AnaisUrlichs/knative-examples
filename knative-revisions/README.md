## Example using Knative for traffic splitting

### Prerequisites
* Kubernetes Cluster set-up -- make sure that it can be configured with Istio
* [kubectl installed](https://kubernetes.io/docs/reference/kubectl/overview/)
* [istioctl installed](https://istio.io/latest/docs/ops/diagnostic-tools/istioctl/)
* [ArgoCD cli installed](https://argoproj.github.io/argo-cd/getting_started/)
* optional: [Knative cli installed](https://knative.dev/docs/install/install-kn/)

## Preparing your cluster

Installing Knative Serving

```
kubectl apply --filename https://github.com/knative/serving/releases/download/v0.21.0/serving-crds.yaml
```

```
kubectl apply --filename https://github.com/knative/serving/releases/download/v0.21.0/serving-core.yaml
```

Installing Istio

```
istioctl install
```

```
kubectl label namespace default istio-injection=enabled
```

```
kubectl label namespace knative-serving istio-injection=enabled
```

Then we have to apply the following file
```
kubectl apply -f https://github.com/knative/net-istio/releases/download/v0.21.0/release.yaml
```

Apply the following config.yaml

```
apiVersion: "security.istio.io/v1beta1"
kind: "PeerAuthentication"
metadata:
  name: "default"
  namespace: "knative-serving"
spec:
  mtls:
    mode: PERMISSIVE
```

```
klubectl apply -f config.yaml
```

get the url for istio service

```
kubectl get svc -n istio-service 
```

and insert it in the following config-domain.yaml

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-domain
  namespace: knative-serving
data:
  <cluster url>.xip.io: |
```

then apply it

```
kubectl apply -f config-domain.yaml
```

install prometheus

```
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.9/samples/addons/prometheus.yaml
```

Access Prometheus (if you want to) -- in this case, we need Prometheus to access metrics from Istio in Kiali

```
kubectl port-forward service/prometheus 9090 -n istio-system
```

Install Kiali

```
istioctl dashboard kiali
```

This will open up Kiali.

Now you can deploy your application through Knative services either applying the manifests directly or using ArgoCD.

## Applying the manifests directly

Apply revision one -- note that you can change the container image used.
```
cd v1
kubectl apply -f release-sample
```

Apply revision two
```
cd v2
kubectl apply -f sample-update.yaml
```

Observe traffic-splitting between revisions
```
cd traffic-splitting
kubectl apply -f traffic-splitting.yaml
```


## Using ArgoCD

Follow the [instructions on installing ArgoCD](https://argoproj.github.io/argo-cd/getting_started/)

```
argocd app create react-example-first --repo tbd --path ./ --dest-server https://kubernetes.default.svc --dest-namespace default

argocd app sync release-sample
```