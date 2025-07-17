# Knative Hands on

### Create Kubernetes Cluster

Create a local Kubernetes cluster using either k3d or kind:

```sh
# Option 1: Using k3d
k3d cluster create k8s-cluster --servers 1 --agents 1 --port 9080:80@loadbalancer --port 9443:443@loadbalancer --api-port 6443 --k3s-arg "--disable=traefik@server:0"

# Option 2: Using kind
kind create cluster --name k8s-cluster

# Verify the cluster is running
kubectl cluster-info --context k8s-cluster
```

To delete the cluster when you're done:

```sh
# If using k3d
k3d cluster delete k8s-cluster

# If using kind
kind delete cluster --name k8s-cluster
```


### Deploy Nginx Ingress

```sh
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

#### Install demo app

```sh
kubectl create deployment demo --image=httpd --port=80

kubectl expose deployment demo

kubectl create ingress demo-localhost --class=nginx \
  --rule="demo.localdev.me/*=demo:80"


curl --resolve demo.localdev.me:8080:127.0.0.1 http://demo.localdev.me:8080

```

### Knative Installation

1. Install the CRDs:

```sh
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.18.1/serving-crds.yaml
```

2. Install core components

```sh
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.18.1/serving-core.yaml
```

check if was installed

```sh
kubectl get all -n knative-serving
```

3. Install knative ingress (internal)

Change service of ingress to [ClusterIP](https://github.com/knative/serving/issues/10417)

```sh
# install kourier
kubectl apply -f kourier.yaml

# update configmap
kubectl patch configmap/config-network \
  --namespace knative-serving \
  --type merge \
  --patch '{"data":{"ingress-class":"kourier.ingress.networking.knative.dev"}}'

# check kourier service status
kubectl --namespace kourier-system get service kourier
```

4. Install sample app

```sh
kubectl apply -f hello.yaml
```

Check  services
```sh
kn service list
```

5. Create rule on ingress

```sh
# create rule
kubectl create ingress hello-knative --class=nginx --rule="hello.knative.io/*=hello-00001:80"

# testing
curl --resolve hello.knative.io:8080:127.0.0.1 http://hello.knative.io:80

```