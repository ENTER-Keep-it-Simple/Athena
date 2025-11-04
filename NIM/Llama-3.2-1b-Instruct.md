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
  storageClassName: ocs-storagecluster-cephfs  # CHANGE TO YOUR STORAGECLASS```

```oc command
oc apply -f pvc.yaml
```
