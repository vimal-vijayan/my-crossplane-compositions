# Installing argocd on a kind cluster

# Helm installation

```bash
helm repo add https://argoproj.github.io/argo-helm

helm install argocd argo/argo-cd \
  --create-namespace \
  --namespace argocd 

# Accessing the ArgoCD UI on port 8080
kubectl port-forward service/argocd-server -n argocd 8080:80


# Accessing the argocd webui inital admin password

kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath='{.data.password}' | base64 --decode
```

## Access the argocd webui on localhost:8080
## Login with the username `admin` and the password from the previous command.


# Add new cluster to argocd

## Here I use the declarative way to add a new cluster to argocd, by creating a Kubernetes secret, Before that I need to get the cluster server and the certificate from the kind cluster, and also setup a auth token for the cluster.

# Setup a service account and a cluster role binding for argocd to access the cluster

```bash
kubectl create serviceaccount argocd-manager -n kube-system
kubectl create clusterrolebinding argocd-manager --clusterrole=cluster-admin --serviceaccount=kube-system:argocd-manager
```

# Get the bearer token for the service account

```bash
kubectl -n kube-system create token argocd-manager --decode
```

### The cluster server and the certificate are obtained from the kind cluster
```bash
kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}'
kubectl config view --raw --minify -o jsonpath='{.clusters[0].cluster.certificate-authority-data}' | base64 --decode
``` 

### The YAML file is here: [kind-cluster-secret.yaml](kind-cluster-secret.yaml)