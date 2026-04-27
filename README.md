# k8s-disaster-recovery-velero-minio
A production-grade setup for Kubernetes Disaster Recovery using Velero and MinIO as an S3-compatible local storage. Features automated scheduled backups and full-state restoration for cluster resources and persistent volumes.


# Kubernetes Disaster Recovery: Velero & MinIO Integration

![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![Velero](https://img.shields.io/badge/Velero-VMware-blue?style=for-the-badge)
![MinIO](https://img.shields.io/badge/MinIO-S3-red?style=for-the-badge&logo=minio&logoColor=white)

A comprehensive guide to implementing a **Disaster Recovery (DR)** strategy for Kubernetes clusters. This project utilizes **Velero** for cluster-state backups and **MinIO** as an on-premise, S3-compatible storage backend.

---

## 📋 Table of Contents
- [Architecture Overview](#-architecture-overview)
- [Prerequisites](#-prerequisites)
- [Project Structure](#-project-structure)
- [Step-by-Step Implementation](#-step-by-step-implementation)
  - [Step 1: Setup MinIO Storage](#step-1-setup-minio-storage)
  - [Step 2: Install Velero CLI & Cluster Setup](#step-2-install-velero-cli--cluster-setup)
  - [Step 3: Perform Backup Operations](#step-3-perform-backup-operations)
  - [Step 4: Restore & Recovery](#step-4-restore--recovery)
- [Best Practices](#-best-practices)

---

## 🏗 Architecture Overview
The backup workflow captures both Kubernetes API resources and Persistent Volume data, streaming them securely to a remote MinIO instance.

---

## 🛠 Prerequisites
Before starting, ensure you have the following:
- **Kubernetes Cluster** (v1.20+)
- **Dedicated VM** for Storage (Ubuntu 22.04 recommended)
- **kubectl** configured for your cluster
- **Network connectivity** between the cluster and the Storage VM on ports 9000/9001

---

## 📂 Project Structure
```text
.
├── scripts/
│   ├── install-minio.sh       # Automates MinIO Server installation
│   └── velero-install.sh      # Velero cluster configuration script
├── manifests/
│   ├── credentials-velero     # IAM-style credentials template
│   └── test-app.yaml          # Sample workload for testing backups
└── README.md                  # Main documentation

## 📖 Step-by-Step Implementation

### Step 1: Setup MinIO Storage
Run these commands on your **Storage VM** to prepare the S3-compatible backend.

```bash
# 1. Download and Install MinIO
wget [https://dl.min.io/server/minio/release/linux-amd64/minio](https://dl.min.io/server/minio/release/linux-amd64/minio)
chmod +x minio
sudo mv minio /usr/local/bin/

# 2. Start MinIO Server (Replace with your storage path)
mkdir -p ~/minio_data
minio server ~/minio_data --console-address ":9001"
