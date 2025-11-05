# OpenFold3 NIM Deployment Documentation

official source: 

## Model Overview.

OpenFold3 is an NVIDIA NIM (NVIDIA Inference Microservice) for protein structure prediction, based on the OpenFold framework (inspired by AlphaFold3). It accelerates biomolecular modeling using TensorRT for GPU-optimized inference. The model predicts 3D structures from protein sequences, including multi-chain complexes, ligands, and modifications. Supports input formats like FASTA sequences and outputs PDB structures with confidence scores (pLDDT, PAE, iPTM).

Key Features:

- Input: Protein sequences (FASTA), optional MSA (Multiple Sequence Alignment) for accuracy.
- Output: PDB files, confidence metrics, and timings.
- GPU Optimization: TensorRT engine for ~10x speedup on NVIDIA GPUs (e.g., RTX 4000 Ada, A100/H100).
- API: OpenAI-compatible endpoints for biology tasks.

This documentation is derived from NVIDIA's official NIM Operator and OpenFold3 deploy guide, adapted for OpenShift with RTX 4000 Ada GPU examples.

Prerquisites.

- NVIDIA GPU: Ada Lovelace or newer (e.g., RTX 4000 Ada with 20GB VRAM; sufficient for small-medium proteins).
- OpenShift Cluster: Version 4.12+ with NVIDIA GPU Operator installed (driver 535+).
- NIM Operator: Installed via OperatorHub (v1.0+).
- NGC Account: API key with OpenFold3 entitlement .
- Storage: Shared RWX StorageClass (e.g., ocs-storagecluster-cephfs) or local (RWO for single-node).
- Resources: 1 GPU, 6–8 CPU, 32Gi RAM, 50Gi+ storage (model ~40GB).
- Access: Accept model license at build.nvidia.com/openfold/openfold3.

Veify GPU:
```oc command
oc describe node <gpu-node> | grep nvidia.com/gpu
# Expected: Allocatable: 1
```

## Step 1: Create Namespace
```oc command
oc new-project openfold3-nim-project
```
## Step 2: Create Secrets
```oc command and bash
# Replace with your nvapi-... key
NVAPI_KEY=<YOUR_NVAPI_KEY>

# API Key Secret (model download)
oc create secret generic ngc-api-secret \
  --namespace=openfold3-nim-project \
  --from-literal=NGC_API_KEY=$NVAPI_KEY

# Docker Registry Secret (nvcr.io pull)
oc create secret docker-registry ngc-registry-secret \
  --namespace=openfold3-nim-project \
  --docker-server=nvcr.io \
  --docker-username=oauth2 \
  --docker-password=$NVAPI_KEY \
  --dry-run=client -o yaml | oc apply -f -
```

## Step 3: Create PersistentVolumeClaim (PVC)
```yaml
# openfold-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: openfold3-pvc
  namespace: openfold3-nim-project
spec:
  accessModes:
    - ReadWriteOnce  # For local-registry
  resources:
    requests:
      storage: 50Gi
  storageClassName: local-registry  # Or ocs-storagecluster-cephfs for shared
```

```oc command
oc apply -f openfold-pvc.yaml
oc get pvc -n openfold3-nim-project  # Wait for Bound
```

## Step 4: Deploy NIMService
```yaml
# openfold-nim.yaml
apiVersion: apps.nvidia.com/v1alpha1
kind: NIMService
metadata:
  name: openfold3
  namespace: openfold3-nim-project
spec:
  image:
    repository: nvcr.io/nim/openfold/openfold3
    tag: latest
    pullPolicy: IfNotPresent
    pullSecrets:
      - ngc-registry-secret
  authSecret: ngc-api-secret
  resources:
    limits:
      nvidia.com/gpu: "1"
      cpu: "6"
      memory: "32Gi"
    requests:
      nvidia.com/gpu: "1"
      cpu: "6"
      memory: "32Gi"
  storage:
    pvc:
      create: false
      name: openfold3-pvc
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                  - sno  # Your GPU node
```

```oc command
oc apply -f openfold-nim.yaml
```

## Step 5: Monitor Deployment
```oc command
oc get pods -n openfold3-nim-project -w
oc logs -n openfold3-nim-project -f $(oc get pod -l app.kubernetes.io/name=openfold3 -n openfold3-nim-project -o name)
```
Verify model cached:
```oc command
oc exec -n openfold3-nim-project $(oc get pod -l app.kubernetes.io/name=openfold3 -n openfold3-nim-project -o name) -- df -h /model-store
# ~40GB used = success
```

## Step 6: Expose Service
```oc command
oc port-forward svc/openfold3 8000:8000 -n openfold3-nim-project
```

For production route:
```oc command
oc create route passthrough openfold-route --service=openfold3 --port=8000 -n openfold3-nim-project
```

## Step 7: API Reference
OpenFold3 uses biology-specific endpoints (OpenAI-compatible base).

List Models (Optional)
```oc command
curl http://localhost:8000/v1/models  # May return 404; use health checks
```

Predict Structure (Main Endpoint)
POST /biology/openfold/openfold3/predict
Request Body (JSON):
```oc command
{
  "request_id": "my_prediction",
  "inputs": [
    {
      "input_id": "protein1",
      "molecules": [
        {
          "type": "protein",
          "id": "A",
          "sequence": "MQIFVKTLTGKTITLEVEPSDTIENVKAKIQDKEGIPPDQQRLIFAGKQLEDGRTLSDYNIQKESTLHLVLRLRGG",
          "msa": {
            "main_db": {
              "csv": {
                "alignment": "key,sequence\n-1,MQIFVKTLTGKTITLEVEPSDTIENVKAKIQDKEGIPPDQQRLIFAGKQLEDGRTLSDYNIQKESTLHLVLRLRGG",
                "format": "csv"
              }
            }
          }
        }
      ],
      "output_format": "pdb"
    }
  ]
}
```
