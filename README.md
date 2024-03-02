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
