# NGINX Deployment with Kustomize and Argo CD

This repository contains Kustomize configuration files to deploy an NGINX server on a Kubernetes cluster. It also includes instructions to deploy the configuration using **Argo CD** for GitOps-based continuous deployment.

## Prerequisites

- Kubernetes cluster (local or cloud)
- `kubectl` command-line tool
- `kustomize` (built into `kubectl` starting from version 1.14+)
- **Argo CD** installed on your Kubernetes cluster

## Setup

### 1. Clone the repository
```bash
git clone <repository-url>
cd <repository-directory>
```

### 2. Review the configuration

- `deployment.yaml`: Defines the NGINX `Deployment` with 3 replicas and resource limits.
- `service.yaml`: Defines the `Service` to expose NGINX on port 80.
- `kustomization.yaml`: Defines how resources are customized and managed together.

## Deploying with Kustomize

### 1. Deploy to Kubernetes

To deploy the NGINX server using Kustomize, use `kubectl`:

```bash
kubectl apply -k ./nginx-deployment/
```

This will create:
- **Deployment**: A deployment named `nginx-deployment` with 3 replicas.
- **Service**: A service named `nginx-service` that exposes NGINX on port 80.

### 2. Verify Deployment

Check if the pods are running:
```bash
kubectl get pods
```

Check the service:
```bash
kubectl get svc
```

### 3. Accessing the NGINX Service

By default, the service type is `ClusterIP`, meaning it is accessible only inside the cluster. If you're using a local cluster (like Minikube), you can access it via:
```bash
minikube service nginx-service
```

## Deploying with Argo CD

Argo CD provides a declarative, GitOps-based continuous delivery system for Kubernetes. Here’s how to deploy this NGINX configuration using Argo CD.

### 1. Install Argo CD

Follow the official documentation to [install Argo CD](https://argo-cd.readthedocs.io/en/stable/getting_started/). After installation, expose the Argo CD API server and access its web UI.

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Access the Argo CD UI (via port forwarding or an external service):
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Log into Argo CD:
```bash
argocd login <ARGO_CD_SERVER>
```

### 2. Create a New Argo CD Application

You can create an Argo CD Application either via the web UI or CLI. Here, we'll create it using the CLI.

```bash
argocd app create nginx-app \
  --repo <repository-url> \
  --path ./nginx-deployment \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --sync-policy automated
```

- `--repo`: URL of your Git repository.
- `--path`: Path to the Kustomize directory in your repo (`nginx-deployment`).
- `--dest-server`: URL of your Kubernetes API server (default: `https://kubernetes.default.svc`).
- `--dest-namespace`: Kubernetes namespace where the resources will be deployed (`default`).
- `--sync-policy automated`: Automatically sync changes from the Git repository to the cluster.

### 3. Sync the Application

After creating the application, sync it using the CLI or the Argo CD UI. This will deploy the NGINX server based on the Kustomize configuration in your repository.

```bash
argocd app sync nginx-app
```

### 4. Verify the Deployment

You can verify that Argo CD has successfully deployed the NGINX application by checking the status of the application.

```bash
argocd app get nginx-app
```

Alternatively, check the Kubernetes resources using `kubectl`:

```bash
kubectl get pods
kubectl get svc
```

### 5. Monitor Changes

Once Argo CD is set up, any changes to the Kustomize configuration in the Git repository will automatically sync and apply to the Kubernetes cluster. You can monitor these changes via the Argo CD UI or CLI:

```bash
argocd app history nginx-app
```

### 6. Delete the Application

To delete the application and its associated resources:

```bash
argocd app delete nginx-app
```

Alternatively, you can delete the deployment using `kubectl`:

```bash
kubectl delete -k ./nginx-deployment/
```

## Accessing the NGINX Service

Since the service type is `ClusterIP`, the service will only be accessible inside the cluster. You can expose it using `kubectl port-forward` or change the service type to `NodePort` or `LoadBalancer` based on your use case.

## Customize the Configuration

Feel free to modify the configuration files to suit your needs:
- Change the number of replicas in `deployment.yaml`.
- Modify resource requests and limits.
- Update service types or ports in `service.yaml`.


