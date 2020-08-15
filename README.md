## Istio - Kubenernets Service Mesh

version: istio-1.6.7

### Install istio minikube with addons Development
1. ~~Increase minikube Node to handle istio recommendations.~~
1. Start minikube with enough resources to run Istio

` minikube start --driver virtualbox --memory 8192 --cpus 4 -p devNode `

2. Activate the istio addons.

```
minikube addons enable istio-provisioner -p {$server_name}
minikube addons enable istio -p {$server_name}
```

### Install istio Production
(Istio : Getting started)[https://istio.io/latest/docs/setup/getting-started/#download]

1. Download Istio speific (release)[https://github.com/istio/istio/releases/tag/1.6.7] version. Command below downloads the latest. (istio-1.6.7)

` curl -L https://istio.io/downloadIstio | sh -`

2. Move into the directory for the download.

```
~~cd istio-1.6.7~~
export PATH=$PWD/istio-1.6.7/bin:$PATH
```

TODO: Add this to ~/.bashrc

3. Install istio using a configuration profile.

` istioctl install --set profile=demo`

### Configure Istio

1. Configure Istio to automatically inject inject Envoy sidecar proxies into spceific namespaces.

``` 
kubectl create namespace {$namespace_name}
kubectl label namespace {$namespace_name} istio-injection=enabled
```

2. Get and Set the ingress IP and Port.

```
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
export INGRESS_HOST=$(minikube ip -p devNode)
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
echo http://$GATEWAY_URL/productpage
```

NOTE: using direnv to manage env variables.  See .envrc file.

3. Access the kaili dashboard

` istioctl dashboard kiali`

4. Test traffic to Application.

` i=1; while [ $i -le 10 ]; do curl http://192.168.99.100:30290/productpage; i=$((i++)) echo "$i"; done`

### Deploy Sample Application

1. Configure Istio to automatically inject inject Envoy sidecar proxies into spceific namespaces.

``` 
kubectl create namespace bookinfo
kubectl label namespace bookinfo istio-injection=enabled
```
NOTE: If pods are already created without istio-injection enabled.  Delete the pod and the sidecar will be injected when the replicator creates the pod again.

``` 
kubectl -n bookinfo get pod -l app=productpage
kubectl -n bookinfo delete pod -l app=productpage
kubectl -n bookinfo get pod -l app=productpage
```


2. Deploy Sample application.

` kubectl apply -f istio-1.6.7/samples/bookinfo/platform/kube/bookinfo.yaml -n bookinfo`

3. Monitor the deployment.

```
kubectl get pods -n bookinfo
kubectl get services -n bookinfo 
kubectl exec -it -n bookinfo $(kubectl -n bookinfo get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}') -c ratings -- curl productpage:9080/productpage | grep -o "<title>.*</title>"
```

4. Open the application to outside traffic by accociating this application with the Istio gateway.

` kubectl apply -f istio-1.6.7/samples/bookinfo/networking/bookinfo-gateway.yaml -n bookinfo`

5. Check the deployment.

` istioctl analyze -n bookinfo`


### Uninstall Istio 

1. Uninstall istioctl.

` istioctl manifest generate --set profile=demo | kubectl delete -f -`

2. Delete the namespace.

` kubectl delete namespace istio-system`
