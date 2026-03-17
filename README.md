# 🧪 Kubernetes Volumes & Persistent Storage

## 📌 Overview

This document provides a concise overview of core Kubernetes storage concepts:

* `emptyDir`
* `hostPath`
* `PersistentVolume (PV)`
* `PersistentVolumeClaim (PVC)`
* Multi-volume Pods

---

## 📦 Volumes

### 🔹 emptyDir

Temporary shared storage between containers in the same Pod.

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

**Notes**

* Data is deleted when the Pod is removed
* Useful for caching and container communication

---

### 🔹 hostPath

Mounts a directory from the node into the Pod.

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

**Notes**

* Direct node filesystem access
* Suitable for development/testing only

---

## 💾 Persistent Storage

### 🔹 PersistentVolume (PV)

Cluster-level storage resource.

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

---

### 🔹 PersistentVolumeClaim (PVC)

Request for storage by a Pod.

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

---

### 🔹 Pod Using PVC

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

**Key Point**

* Data persists beyond Pod lifecycle

---

## 🔬 Multi-Volume Pod

Combining different volume types in a single Pod:

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

---

## 🧠 Summary

| Type     | Purpose            |
| -------- | ------------------ |
| emptyDir | Temporary storage  |
| hostPath | Node access        |
| PV/PVC   | Persistent storage |

---

## 🚀 Key Notes

* Storage flow: **Pod → PVC → PV**
* Persistence requires **PV + PVC**
* Use `emptyDir` for short-lived data only
