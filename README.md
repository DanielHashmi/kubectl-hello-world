# Kubernetes Hello World Project

This guide demonstrates how to deploy a simple "Hello World" application using Kubernetes and `kubectl`.

## Project Overview

We will deploy a simple Nginx web server. When accessed, it will serve a default "Welcome to nginx!" page. This is the standard "Hello World" for Kubernetes infrastructure.

## Prerequisites

- `kubectl` installed and configured to talk to a cluster (Minikube, Docker Desktop, Kind, etc.).

## The Steps

### 1. Define the Application (The "Spec")

We define our application in a YAML file. This tells Kubernetes *what* we want running.
We will create a file named `hello-world.yaml`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-world-pod
  labels:
    app: hello-world
spec:
  containers:
  - name: web-server
    image: nginx
    ports:
    - containerPort: 80
```

### 2. Deploy the Application (`kubectl apply`)

We send the file to the Kubernetes API server.

```bash
kubectl apply -f hello-world.yaml
```

*   `apply`:  Manages applications through files defining Kubernetes resources.
*   `-f`: Specifies the filename.

### 3. Verify Deployment (`kubectl get` & `kubectl describe`)

Check if the Pod is running.

```bash
kubectl get pods
```

If it's not running yet (status `Pending` or `ContainerCreating`), wait a few seconds and run it again.

To see detailed events (useful for debugging):

```bash
kubectl describe pod hello-world-pod
```

### 4. Access the Application (`kubectl port-forward`)

By default, Pods are isolated. To access the web server from your local machine (without creating a full Service/Ingress setup), we use port forwarding.

```bash
# Forward local port 8080 to the Pod's port 80
kubectl port-forward hello-world-pod 8080:80
```

Then open your browser to `http://localhost:8080` or use `curl`.

### 5. Cleanup (`kubectl delete`)

Remove the resources when done.

```bash
kubectl delete -f hello-world.yaml
```

---

## Part 2: Deep Dive - The "Real" Work

Now that the app is running, let's see what `kubectl` can really do.

### 1. The "Live Patch" (Modifying the container)

You can run commands *inside* the container. Let's change the website content while it's running.

```bash
# Overwrite the index.html file inside the container
kubectl exec hello-world-pod -- sh -c "echo '<h1>Hello from Kubernetes!</h1><p>I updated this using kubectl.</p>' > /usr/share/nginx/html/index.html"
```

**Verify:** Refresh your browser at `http://localhost:8080`. You should see the text change instantly!

### 2. The "Detective" (Streaming Logs)

Watch traffic come in real-time.

```bash
# The -f flag follows the log stream (like 'tail -f')
kubectl logs -f hello-world-pod
```

**Action:** Refresh your browser a few times. You will see new log lines appear in your terminal for every request.
*(Press `Ctrl+C` to stop watching)*

### 3. The "Inventory Manager" (Labels)

In big clusters, you have thousands of Pods. Labels help you find them.

```bash
# Add a new label 'env=production'
kubectl label pod hello-world-pod env=production

# List only pods that match that label
kubectl get pods -l env=production
```
