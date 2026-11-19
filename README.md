@'

# Kubernetes Lab 1: Deploy MongoDB and Mongo Express

## 1. Architecture

```text
Browser ──▶ Mongo Express Service (NodePort :8081) ──▶ Mongo Express Pod ──▶ MongoDB Service (ClusterIP :27017) ──▶ MongoDB Pod
```

## 2. Project Files

- `mongodb-secret.yaml`: Stores MongoDB root credentials (`admin` / `ChangeMe123!`).
- `mongodb-deployment.yaml`: Deploys MongoDB (`mongo:7`) on port `27017`.
- `mongodb-service.yaml`: Exposes MongoDB internally within the cluster via `ClusterIP`.
- `mongo-express-configmap.yaml`: Stores the internal MongoDB Service hostname (`mongodb-service`).
- `mongo-express-deployment.yaml`: Deploys Mongo Express (`mongo-express:1.0.2`) on port `8081`.
- `mongo-express-service.yaml`: Exposes the Mongo Express web UI externally via `NodePort`.

---

## 3. In-Lab Student Tasks

### Step 3 – Why use Secrets instead of directly writing passwords inside application Deployment files?

Hardcoding passwords directly inside Deployment YAML files exposes sensitive credentials to anyone who can view the source code or Git repository. Kubernetes Secrets separate confidential data from application manifests, store values encoded (and optionally encrypted at rest in `etcd`), and allow strict Role-Based Access Control (RBAC) so only authorized Pods and administrators can access them.

### Step 4 – Identify the container image, container port, and number of replicas in the MongoDB Deployment

- **Container Image:** `mongo:7`
- **Container Port:** `27017`
- **Replicas:** `1`

### Step 6 – Why are we using `mongodb-service` instead of the MongoDB Pod IP address?

Kubernetes Pods are ephemeral—if a Pod crashes, restarts, or is rescheduled, Kubernetes destroys it and assigns a completely new IP address to the replacement Pod. A Kubernetes Service (`mongodb-service`) provides a permanent, unchanging DNS name and virtual IP address that automatically routes traffic to whichever Pod is currently healthy.

---

## 4. Troubleshooting Tasks (Section 14)

- **Challenge 1 (`ImagePullBackOff`):** Changing the MongoDB image to an invalid tag (`mongo:invalid-image-999`) caused the container runtime to fail downloading the image from Docker Hub (`ErrImagePull`) and enter `ImagePullBackOff` after repeated retry delays, while Kubernetes kept the original healthy Pod running to prevent downtime.
- **Challenge 2 (Service Selector Mismatch):** Changing the `mongodb-service` selector from `app: mongodb` to `app: database` caused `kubectl get endpoints mongodb-service` to return `<none>` because Services match Pods using labels, and no Pods carried the label `app: database`.
- **Challenge 3 (Deleting the MongoDB Pod):** Deleting the running MongoDB Pod caused Kubernetes to immediately spin up a brand-new replacement Pod because the Deployment's ReplicaSet continuously enforces the desired state of `replicas: 1`.
- **Challenge 4 (Scaling Mongo Express to 3 Replicas):** Changing `replicas` from `1` to `3` instructed the Deployment to update its ReplicaSet, which automatically created 2 additional Mongo Express Pods to match the new desired count.

---

## 5. Student Assessment Questions (Section 15)

1. **What is Kubernetes?**
   An open-source container orchestration platform that automates deploying, scaling, networking, and managing containerized applications across clusters of machines.

2. **What is the difference between a Pod and a Deployment?**
   A **Pod** is the smallest deployable unit in Kubernetes (a wrapper around one or more containers), whereas a **Deployment** is a higher-level controller that manages Pods—handling replica scaling, rolling updates, and automatic replacement if a Pod fails.

3. **What is a Kubernetes Service?**
   A stable networking abstraction that provides a consistent IP address and internal DNS name to expose and route traffic to a dynamic set of Pods.

4. **Why did we use `ClusterIP` for MongoDB?**
   `ClusterIP` restricts access strictly to internal cluster communication, keeping the database secure from direct external internet exposure.

5. **Why did we use `NodePort` for Mongo Express?**
   `NodePort` opens a static port on the cluster node's IP address, allowing external traffic from outside the cluster (such as a local web browser) to access the Mongo Express web UI.

6. **What is the difference between ConfigMap and Secret?**
   Both decouple configuration data from container images as key-value pairs, but **ConfigMaps** store non-sensitive plain-text settings (like hostnames or URLs), while **Secrets** store confidential credentials (passwords, tokens, API keys) in Base64-encoded format with stricter security controls.

7. **How does Mongo Express communicate with MongoDB?**
   Mongo Express reads the `ME_CONFIG_MONGODB_SERVER` environment variable (`mongodb-service`) injected from its ConfigMap. Kubernetes' internal DNS resolves `mongodb-service` to the `ClusterIP` of the MongoDB Service, which forwards traffic on port `27017` to the MongoDB Pod.

8. **What is the purpose of `containerPort`?**
   It documents and informs Kubernetes which network port the application inside the container is actively listening on.

9. **What is the difference between `port` and `targetPort`?**
   `port` is the network port exposed by the **Service** itself (what other applications connect to), while `targetPort` is the actual port inside the **Pod/container** where the Service forwards that traffic.

10. **What happens when a Pod managed by a Deployment is deleted?**
    The Deployment's underlying ReplicaSet detects that the actual number of running Pods is less than the desired `replicas` count and immediately creates a brand-new replacement Pod to restore the desired state.
    '@ | Set-Content -Encoding UTF8 README.md

Get-Content README.md -TotalCount 15
