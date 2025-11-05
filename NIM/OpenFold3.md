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
