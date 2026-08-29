# E-Commerce Application Kubernetes Deployment

This repository contains the **Kubernetes manifests** required to run the full MERN‑stack e‑commerce application on a Kubernetes cluster.  The application consists of four backend services (Auth, Product, Cart, Order), a MongoDB replica‑set, and an NGINX Ingress controller that routes traffic from a single entry point to each service.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Directory Layout](#directory-layout)
3. [Deploy MongoDB](#deploy-mongodb)
4. [Deploy Backend Services](#deploy-backend-services)
5. [Deploy Ingress](#deploy-ingress)
6. [Testing the Deployment](#testing-the-deployment)
7. [Cleanup](#cleanup)
8. [Notes & Tips](#notes--tips)

---

## Prerequisites

- A running Kubernetes cluster (e.g., **minikube**, **kind**, GKE, AKS, EKS, etc.).
- `kubectl` configured to talk to that cluster.
- Access to the Docker images used by the services. The manifests reference images hosted on Docker Hub under the namespace `docker.io/siddhartha0709/`:
  - `auth-service`
  - `product-service`
  - `cart-service`
  - `order-service`
  - `mongo` (standard MongoDB image, used in the StatefulSet)
- (Optional) **NGINX Ingress Controller** installed in the cluster.  If you are using **minikube**, you can enable it with:
  ```bash
  minikube addons enable ingress
  ```

---

## Directory Layout

```
deployments/
├─ README.md                # ← This file
├─ ingress.yaml             # Ingress resource routing to services
├─ mongodb-deploy/          # MongoDB StatefulSet & Services
│   ├─ 01statefulSet.yaml
│   ├─ 02headlessService.yaml
│   └─ 03enable-replication.yaml
├─ auth-deploy/             # Authentication service
│   ├─ 01deployment.yaml
│   └─ 02services.yaml
├─ product-deploy/          # Product catalogue service
│   ├─ 01deployment.yaml
│   └─ 02services.yaml
├─ cart-deploy/             # Shopping‑cart service
│   ├─ 01deployment.yaml
│   └─ 02services.yaml
└─ order-deploy/            # Order service
    ├─ 01deployment.yaml
    └─ 02services.yaml
```

Each `01deployment.yaml` defines a **Deployment** with a single replica and the required environment variables (e.g., `MONGO_URI`, `ALLOWED_ORIGIN`).
Each `02services.yaml` creates a **NodePort** Service exposing the pod on a cluster‑wide port.

---

## Deploy MongoDB

MongoDB is deployed as a **StatefulSet** with a single replica (suitable for local development).  The replication configuration is applied after the StatefulSet is up.

```bash
# Apply the StatefulSet and the headless service
kubectl apply -f deployments/mongodb-deploy/01statefulSet.yaml
kubectl apply -f deployments/mongodb-deploy/02headlessService.yaml

# Enable replication (creates an init container that initiates the replica set)
kubectl apply -f deployments/mongodb-deploy/03enable-replication.yaml
```

> **Tip** – Verify the pod is ready:
> ```bash
> kubectl get pods -l app=mongo
> ```

---

## Deploy Backend Services

All backend services use the same pattern – a Deployment followed by a NodePort Service.

```bash
# Auth service
kubectl apply -f deployments/auth-deploy/01deployment.yaml
kubectl apply -f deployments/auth-deploy/02services.yaml

# Product service
kubectl apply -f deployments/product-deploy/01deployment.yaml
kubectl apply -f deployments/product-deploy/02services.yaml

# Cart service
kubectl apply -f deployments/cart-deploy/01deployment.yaml
kubectl apply -f deployments/cart-deploy/02services.yaml

# Order service
kubectl apply -f deployments/order-deploy/01deployment.yaml
kubectl apply -f deployments/order-deploy/02services.yaml
```

Each service expects the MongoDB URI to be reachable at `mongodb://mongo:27017/<db‑name>` where `<db‑name>` matches the service (e.g., `auth-db`).  The manifests already set this variable, so no further configuration is required unless you change the MongoDB DNS name.

---

## Deploy Ingress

The **Ingress** object routes external HTTP traffic to the appropriate backend based on the request path.  The provided `ingress.yaml` assumes an NGINX Ingress controller is present.

```bash
kubectl apply -f deployments/ingress.yaml
```

The Ingress rules map the following prefixes:

- `/auth` → Auth service (port **3001**)
- `/product` → Product service (port **3002**)
- `/order` → Order service (port **3004**)
- `/cart` → Cart service (port **3003**)

If you are running locally with **minikube**, you can obtain the Ingress IP with:

```bash
minikube ip   # returns the node IP
```
Then access the API, e.g., `http://$(minikube ip)/auth/login`.

---

## Testing the Deployment

1. **Check all pods are running**

   ```bash
   kubectl get pods
   ```

2. **Port‑forward a service** (optional, for quick curl testing without Ingress):

   ```bash
   # Example: expose the auth service locally on port 3001
   kubectl port-forward svc/auth 3001:3001
   curl http://localhost:3001/health   # or any endpoint your service provides
   ```

3. **Use the Ingress** (once the controller IP is known):

   ```bash
   curl http://<INGRESS_IP>/auth/ping
   curl http://<INGRESS_IP>/product/ping
   curl http://<INGRESS_IP>/cart/ping
   curl http://<INGRESS_IP>/order/ping
   ```

   Adjust the paths according to the actual routes implemented in each service.

---

## Cleanup

To remove everything created by these manifests:

```bash
kubectl delete -f deployments/ingress.yaml
kubectl delete -f deployments/auth-deploy
kubectl delete -f deployments/product-deploy
kubectl delete -f deployments/cart-deploy
kubectl delete -f deployments/order-deploy
kubectl delete -f deployments/mongodb-deploy
```

---

## Notes & Tips

- **Image tags** – The manifests pin specific Git SHA tags.  When you push a new version of a service, update the `image:` field in the corresponding `01deployment.yaml` and re‑apply the file.
- **Scaling** – For production you will likely want multiple replicas and a proper MongoDB replica‑set with three members.  Adjust the `replicas:` field and the StatefulSet spec accordingly.
- **Secret management** – Credentials (e.g., MongoDB password) are currently hard‑coded for simplicity.  In a real deployment, replace them with Kubernetes **Secrets** and reference them via `envFrom` or `valueFrom`.
- **Ingress class** – The manifest uses `ingressClassName: nginx`.  If your cluster uses a different controller (e.g., GKE’s GLBC), modify this field or remove it.
- **NodePort range** – The services expose ports `30007‑30010`.  Ensure these ports are allowed by your cloud provider or local VM firewall.

---

*Happy hacking!* 
