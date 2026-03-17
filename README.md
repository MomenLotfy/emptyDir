# 🧪 Kubernetes Volumes & Persistent Storage Lab (CKA Level)

## 📌 Overview

This lab covers core Kubernetes storage concepts required for the CKA exam:

* `emptyDir`
* `hostPath`
* `PersistentVolume (PV)`
* `PersistentVolumeClaim (PVC)`
* Multi-volume Pods

⏱ Duration: 50 minutes
📊 Total Score: 100 points
🎯 Level: CKA

---

# 📦 Part 1 — Volumes (30 pts)

## 1️⃣ emptyDir — Share Data Between Containers ⭐

### 🎯 Objective

Create a Pod with two containers sharing a temporary volume.

### ⚙️ Key Concepts

* Lifecycle tied to Pod
* Data is deleted when Pod is removed
* Used for temporary storage and inter-container communication

### 📄 YAML: `01-emptydir-volume.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-demo
spec:
  containers:
  - name: writer
    image: busybox
    command: ["/bin/sh", "-c"]
    args:
      - while true; do date >> /shared/log.txt; sleep 5; done
    volumeMounts:
    - name: shared-volume
      mountPath: /shared

  - name: reader
    image: busybox
    command: ["/bin/sh", "-c"]
    args:
      - while true; do cat /shared/log.txt; sleep 10; done
    volumeMounts:
    - name: shared-volume
      mountPath: /shared

  volumes:
  - name: shared-volume
    emptyDir: {}
```

### 🧪 Verification

```bash
kubectl apply -f 01-emptydir-volume.yaml
kubectl logs emptydir-demo -c reader
kubectl exec emptydir-demo -c writer -- cat /shared/log.txt
```

### ❗ Important

```bash
kubectl delete pod emptydir-demo && kubectl apply -f 01-emptydir-volume.yaml
```

➡️ Data will be lost after recreation

---

## 2️⃣ hostPath — Access Node Files ⭐⭐

### 🎯 Objective

Serve a static HTML file from the node using NGINX.

### ⚙️ Key Concepts

* Mounts node filesystem into Pod
* Not recommended for production
* Useful for testing/debugging

### 🔧 Prepare Node

```bash
minikube ssh
mkdir -p /tmp/webdata
echo '<h1>From the Node!</h1>' > /tmp/webdata/index.html
chmod -R 777 /tmp/webdata
```

### 📄 YAML: `02-hostpath-volume.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-demo
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: host-volume
      mountPath: /usr/share/nginx/html

  volumes:
  - name: host-volume
    hostPath:
      path: /tmp/webdata
      type: Directory
```

### 🧪 Verification

```bash
kubectl apply -f 02-hostpath-volume.yaml
kubectl exec hostpath-demo -- curl localhost
```

### 🔁 Live Update Test

```bash
minikube ssh
echo '<h1>Updated!</h1>' > /tmp/webdata/index.html
```

```bash
kubectl exec hostpath-demo -- curl localhost
```

---

# 💾 Part 2 — Persistent Storage (50 pts)

## 3️⃣ PersistentVolume ⭐

### 📄 YAML: `03-persistentvolume.yaml`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /tmp/pv-data
  persistentVolumeReclaimPolicy: Retain
```

### 🧪 Apply

```bash
kubectl apply -f 03-persistentvolume.yaml
kubectl get pv
```

---

## 4️⃣ PersistentVolumeClaim ⭐⭐

### 📄 YAML: `04-persistentvolumeclaim.yaml`

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

### 🧪 Apply

```bash
kubectl apply -f 04-persistentvolumeclaim.yaml
kubectl get pv,pvc
```

➡️ Expected: `Bound`

---

## 5️⃣ Pod Using PVC ⭐⭐

### 📄 YAML: `05-pod-with-pvc.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-demo
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: pvc-volume
      mountPath: /usr/share/nginx/html

  volumes:
  - name: pvc-volume
    persistentVolumeClaim:
      claimName: my-pvc
```

### 🧪 Test Persistence

```bash
kubectl apply -f 05-pod-with-pvc.yaml
kubectl exec pvc-demo -- sh -c 'echo persistent > /usr/share/nginx/html/test.txt'

kubectl delete pod pvc-demo
kubectl apply -f 05-pod-with-pvc.yaml

kubectl exec pvc-demo -- cat /usr/share/nginx/html/test.txt
```

➡️ Data persists after Pod deletion ✅

---

# 🔬 Part 3 — Multi-Volume Pod (20 pts)

## 6️⃣ Multi-Volume Pod ⭐⭐⭐

### 🔧 Setup

```bash
kubectl create configmap app-config --from-literal=KEY=value
```

### 📄 YAML: `07-multi-volume-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-volume-demo
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c"]
    args: ["sleep 3600"]

    volumeMounts:
    - name: cache-vol
      mountPath: /tmp/cache

    - name: data-vol
      mountPath: /data

    - name: config-vol
      mountPath: /etc/config

  volumes:
  - name: cache-vol
    emptyDir:
      medium: Memory

  - name: data-vol
    persistentVolumeClaim:
      claimName: my-pvc

  - name: config-vol
    configMap:
      name: app-config
```

### 🧪 Verification

```bash
kubectl apply -f 07-multi-volume-pod.yaml
kubectl exec multi-volume-demo -- df -h
kubectl exec multi-volume-demo -- ls /etc/config
```

---

# 🧠 Key Takeaways

| Type     | Use Case                     |
| -------- | ---------------------------- |
| emptyDir | Temporary data sharing       |
| hostPath | Node-level access (dev only) |
| PV + PVC | Persistent production data   |

---

# 🚀 Pro Tips (CKA)

* Always debug using:

  ```bash
  kubectl get pods
  kubectl describe pod <name>
  ```

* Remember:

  * Pod → PVC → PV
  * Data persists only with PV/PVC

---

# 🎯 Conclusion

This lab demonstrates the full lifecycle of Kubernetes storage:

* Ephemeral → `emptyDir`
* Node-bound → `hostPath`
* Persistent → `PV/PVC`

Mastering these concepts is critical for both the CKA exam and real-world DevOps workflows.
