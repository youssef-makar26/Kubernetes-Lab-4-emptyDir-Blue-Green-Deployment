# 🚀 Kubernetes Lab 4 — emptyDir & Blue-Green Deployment

## 📌 Overview

In this lab, we deployed a logging application inside a Kubernetes cluster to explore:

* 📦 How `emptyDir` volumes work
* 🔄 Behavior during container restart vs pod recreation
* 🚦 Blue-Green deployment strategy
* 🐞 Debugging a real-world configuration bug (`CrashLoopBackOff`)

This lab simulates real DevOps scenarios involving storage, deployments, and troubleshooting.

---

## 🏗️ Architecture

```
                +---------------------+
                |   logger-service    |
                |   (ClusterIP)       |
                +----------+----------+
                           |
                -----------------------
                |                     |
        +---------------+     +---------------+
        |  logger v1    |     |  logger v2    |
        | (1 replica)   |     | (2 replicas)  |
        +---------------+     +---------------+

Each Pod uses:
👉 emptyDir volume mounted at /log
👉 Container writes logs continuously
```

---

## ⚙️ Technologies Used

* Kubernetes (kubectl)
* YAML (Declarative configs)
* BusyBox container
* emptyDir volume
* ClusterIP Service

---

## 📁 Project Structure

```
Lab_4/
│
├── manifests/
│   ├── v1/
│   │   └── logger-deployment.yaml
│   │
│   ├── v2/
│   │   └── logger-deployment.yaml
│   │
│   └── service/
│       └── logger-service.yaml
│
└── README.md
```

---

## 🚀 Deployment Steps

### 1️⃣ Create Namespace

```bash
kubectl create namespace staging
```

### 2️⃣ Deploy Logger v1

```bash
kubectl apply -f manifests/v1/logger-deployment.yaml
kubectl apply -f manifests/service/logger-service.yaml
```

### 3️⃣ Verify Deployment

```bash
kubectl get all -n staging
```

### 4️⃣ Check Logs

```bash
kubectl exec -n staging -it <pod-name> -- cat /log/output.txt
```

---

## 📦 Understanding emptyDir Behavior

### 🔹 Case 1: Container Restart

```bash
kubectl exec -n staging -it <pod> -- /bin/sh
kill 1
```

📌 Result:

* Container restarts
* Pod remains the same
* ✅ Data is preserved

👉 Reason: `emptyDir` is tied to the Pod, not the container.

---

### 🔹 Case 2: Pod Deletion

```bash
kubectl delete pod -n staging <pod>
```

📌 Result:

* New Pod is created
* ❌ Old data is lost

👉 Reason: New Pod = new `emptyDir`.

---

## 🔄 Blue-Green Deployment

### Step 1 — Deploy v2

```bash
kubectl apply -f logger-v2.yaml
```

### Step 2 — Switch Traffic

```bash
kubectl patch svc logger-service -n staging -p '{"spec":{"selector":{"version":"v2"}}}'
```

📌 Traffic is now routed to v2 with zero downtime.

---

## 🐞 Bug Scenario — CrashLoopBackOff

### ❌ Problem

The container writes logs to:

```
/log/output.txt
```

But the volume was mounted to:

```
/var/log/app
```

📌 Result:

* `/log` does not exist
* Container crashes
* Kubernetes keeps restarting it
* → CrashLoopBackOff

---

## 🔍 Debugging

```bash
kubectl get pods -n staging
kubectl describe pod -n staging <pod>
kubectl logs -n staging <pod>
```

Example error:

```
can't create /log/output.txt: nonexistent directory
```

---

## ✅ Fix

Update the mount path:

```yaml
volumeMounts:
  - name: log-volume
    mountPath: /log
```

Then apply:

```bash
kubectl apply -f manifests/v2/logger-deployment.yaml
kubectl rollout restart deployment/logger-v2 -n staging
kubectl rollout status deployment/logger-v2 -n staging
```

---

## 🧪 Final Verification

```bash
kubectl get pods -n staging
kubectl get endpoints -n staging logger-service
kubectl logs -n staging <pod>
```

Expected output:

```
Sun Mar 17 12:00:00 UTC 2026 - Host: logger-v2-xxxx
```

---

## 🧠 Key Learnings

* `emptyDir` persists only during Pod lifecycle
* Container restart does NOT delete data
* Pod recreation deletes data
* Blue-Green enables zero downtime
* Labels control traffic routing
* Misconfigured volumes can break apps

---

## 💡 Conclusion

In this lab, we:

* Built a logging system using `emptyDir`
* Tested storage lifecycle behavior
* Implemented Blue-Green deployment
* Diagnosed and fixed a real bug

🎯 This lab builds strong real-world Kubernetes troubleshooting skills.

---

## 🙌 Author

**Mohannad Khairy**
DevOps Engineer 🚀


