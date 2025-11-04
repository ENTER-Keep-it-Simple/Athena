# NVIDIA NIM Deployment Guide: Llama 3.2 1B Instruct on OpenShift (GPU)

## Overview
Deploy NVIDIA NIM for Llama 3.2 1B Instruct on OpenShift with GPU using NIMService.
This guide includes all correct and missing steps to avoid hours of debugging.

## Prerequisites
| Requirement | Command / Action |
|----------|----------|
| OpenShift cluster  | oc login  |
| NVIDIA GPU Operator  | Installed & GPU visible (oc describe node) |
| StorageClass (ReadWriteMany or local)  | e.g., ocs-storagecluster-cephfs or hostpath  |
| NGC API Key  | nvapi-... from https://ngc.nvidia.com/setup  |
| Model Access  | Accept license at https://build.nvidia.com/meta/llama-3.2-1b-instruc  |

## Step 1: Create Namespace
```oc command
oc create namespace llama-nim-project
```

## Step 2: Create PersistentVolumeClaim (PVC)
```yaml
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: llama-3-2-1b-instruct-pvc
  namespace: llama-nim-project
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 30Gi
  storageClassName: ocs-storagecluster-cephfs  # CHANGE TO YOUR STORAGECLASS
```

```oc command
oc apply -f pvc.yaml
```

## Step 3: Create Secrets (CORRECT FORMAT)

### 3.1 NGC API Key (Model Download)
```oc command
oc create secret generic ngc-api-secret \
  --namespace=llama-nim-project \
  --from-literal=NGC_API_KEY=<YOUR_API_KEY>
```
### 3.2 Docker Registry Secret (Image Pull)
```oc command
oc create secret docker-registry ngc-registry-secret \
  --namespace=llama-nim-project \
  --docker-server=nvcr.io \
  --docker-username='$oauthtoken' \ # Specified in NGC  model's deploy tab
  --docker-password=<YOUR_API_KEY> \
  --dry-run=client -o yaml | oc apply -f -
```

