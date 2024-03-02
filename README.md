# barcamp20-2024

## Setup Mac

Install Kubetcl

```
brew install kubectl
```

Create alias `k` for `kubectl`

```zsh
alias k=kubectl
```

Install Helm

```zsh
brew install helm
```

## Create a cluster on Digital Ocean

https://cloud.digitalocean.com/kubernetes/clusters

## Connect to the cluster

Download the kubeconfig file from Digital Ocean

If you dont have any previous Kubernetes configuration, move the file to `~/.kube/config`.

If you do, you can add a new context to the file.

Backup original config file

```zsh
cp ~/.kube/config ~/.kube/config.$(date +%Y-%m-%d_%H-%M-%S).backup
```

Add new context

```zsh
KUBECONFIG=kubeconfig-new.yml:~/.kube/config kubectl config view --raw > ~/.kube/config.merge.yml && cp ~/.kube/config.merge.yml ~/.kube/config
```

## Check the connection

```zsh
kubectl get nodes
```

## Install Ingress Nginx

```zsh
helm upgrade --install \
  ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --create-namespace \
  --namespace ingress-nginx \
  --set controller.ingressClassResource.default=true \
  --wait
```

## Install cert-manager

```zsh
helm upgrade --install \
  cert-manager cert-manager \
  --repo https://charts.jetstack.io \
  --create-namespace \
  --namespace cert-manager \
  --set installCRDs=true \
  --wait
```

## Create a ClusterIssuer

```zsh
kubectl apply -f clusterissuer-letsencrypt.yml
```

## Get IP address of the Load Balancer

```zsh
kubectl get svc -n ingress-nginx
```

## Point the domain to the Load Balancer

Go to Cloudflare and create an A record pointing to the Load Balancer IP address.

## Test the Cluster

Install Hello World app

```zsh
helm upgrade --install \
  hello-world hello-world \
  --repo https://helm.sikalabs.io \
  --set host=hello-world.barcamp20.sikademo.com \
  --set TEXT="Hello Barcamp 2.0" \
  --wait
```

## Backend App

We have a app in the Docker image `ondrejsika/counter` which listen on port 80 and have those configuration environment variables:

- `REDIS` - Redis host (default: 127.0.0.1)
- `EXTRA_TEXT` - Extra text to display (default: '')

Source: https://github.com/ondrejsika/counter

## Create Deployment for Backend App

```zsh
kubectl apply -f backend-deploy.yml
```

## See the Deployment

```zsh
kubectl get deploy
```

## See the Pods

```zsh
kubectl get pods
```

## See Deloyments & Pods

```zsh
kubectl get deploy,pods
```

## See the Logs

```zsh
kubectl logs -f <pod-name>
```

```zsh
kubectl logs -f deployment/backend
```

## Create Service for Backend App

```zsh
kubectl apply -f backend-svc.yml
```

## See the Service

```zsh
kubectl get svc
```

## Create Ingress for Backend App

```zsh
kubectl apply -f backend-ing.yml
```

## See the Ingress

```zsh
kubectl get ing
```

## Deploy Redis

```zsh
kubectl apply -f redis-sts.yml
kubectl apply -f redis-svc.yml
```

and configure Hedis host in backend app

```zsh
kubectl apply -f backend-deploy-2.yml
```

## Frontend App

We have a frontend app in the Docker image `ghcr.io/ondrejsika/counter-frontend` which listen on port 3000 and have those configuration environment variables:

- `API_ORIGIN` - Origin of the API server (default: http://127.0.0.1)
- `FONT_COLOR` - Color of the counter text (default: #000000, black)
- `BACKGROUND_COLOR` - Color of the counter background (default: #ffffff, white)

Source: https://github.com/ondrejsika/counter-frontend

## Deploy Frontend App

```zsh
kubectl apply -f frontend-deploy.yml
kubectl apply -f frontend-svc.yml
kubectl apply -f frontend-ing.yml
```

## See All Workload and services

```zsh
kubectl get all
```

## See All Workload and services with Ingresses

```zsh
kubectl get all,ing
```
