# k8s-disaster-recovery-velero-minio
A production-grade setup for Kubernetes Disaster Recovery using Velero and MinIO as an S3-compatible local storage. Features automated scheduled backups and full-state restoration for cluster resources and persistent volumes.


# Kubernetes Disaster Recovery (Velero & MinIO)

This repository demonstrates a robust Disaster Recovery (DR) strategy for Kubernetes clusters using **Velero** and **MinIO**. It focuses on an on-premise/hybrid approach where backups are stored on a separate dedicated VM instead of public cloud buckets.

![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![Velero](https://img.shields.io/badge/Velero-VMware-blue?style=for-the-badge)
![MinIO](https://img.shields.io/badge/MinIO-S3-red?style=for-the-badge&logo=minio&logoColor=white)

## 🛠 Features
- **Full Cluster Backup:** Captures all Kubernetes resources and metadata.
- **Persistent Volume Backup:** Uses Velero Node-Agent (Restic/Kopia) for File-System backups.
- **On-Premise S3 Storage:** Leverages MinIO as a high-performance S3-compatible backend.
- **Automated Scheduling:** Daily backup cron jobs for zero-touch protection.

## 🏗 Architecture
The setup consists of:
1. **Source Cluster:** Any K8s cluster (EKS, K3s, or Kubeadm).
2. **Backup Server:** A separate VM running MinIO Server.
3. **Velero:** Installed on the cluster to orchestrate the backup/restore process.

## 🚀 Quick Start

### 1. Setup MinIO (Backup Server)
On your storage VM, run the installation script:
```bash
# Example command to run your custom script
./scripts/install-minio.sh
