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
  - [Step 3: Restore & Recovery](#step-4-restore--recovery)

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

## 📖 Step-by-Step Implementation

### Step 1: Setup MinIO Storage
Run these commands on your **Storage VM** to prepare the S3-compatible backend.

**1. Download and Install MinIO**
```
wget [https://dl.min.io/server/minio/release/linux-amd64/minio](https://dl.min.io/server/minio/release/linux-amd64/minio)
chmod +x minio
sudo mv minio /usr/local/bin/
```

**2. Start MinIO Server (Replace with your storage path)**
```
sudo mkdir -p /mnt/minio_data
sudo chown minio-user:minio-user /mnt/minio_data
```
**3. Preparing the configuration file**
We will create a file in which we define the Access Key and the Secret Key (which Velero will use):
```
sudo vim /etc/default/minio
```
Copy these settings into it:
```
MINIO_ROOT_USER=minioadmin
MINIO_ROOT_PASSWORD=minioadmin
MINIO_VOLUMES="/mnt/minio_data"
MINIO_OPTS="--address :9000 --console-address :9001"
```
**4.Create a Systemd Service**
So that MinIO runs as a background service:
```
sudo nano /etc/systemd/system/minio.service
```
Copy this content:
```
[Unit]
Description=MinIO
Documentation=https://docs.min.io
Wants=network-online.target
After=network-online.target

[Service]
User=minio-user
Group=minio-user
EnvironmentFile=/etc/default/minio
ExecStart=/usr/local/bin/minio server $MINIO_OPTS $MINIO_VOLUMES
Restart=always

[Install]
WantedBy=multi-user.target
```
**5. Server startup**
```
sudo systemctl daemon-reload
sudo systemctl enable minio
sudo systemctl start minio
```
**6. Final Step (Very Important)**
Now open your browser and go to: localhost:9001

 1. Log in as minioadmin /minioadmin.

 2. From the sidebar, select Buckets.

 3. Click Create Bucket and name it velero-backups.

### step-2-install-velero-cli--cluster-setup
  
**1. The machine you use to control the cluster**
```
wget https://github.com/vmware-tanzu/velero/releases/download/v1.12.0/velero-v1.12.0-linux-amd64.tar.gz
tar -xvf velero-v1.12.0-linux-amd64.tar.gz
sudo mv velero-v1.12.0-linux-amd64/velero /usr/local/bin/
```
 **2. Install Velero inside the cluster.**

This is the "main" command that will link the cluster to the other machines (those running MinIO).
Note: Replace 192.168.1.100 with the IP address of the other machine's "destination VM ".
```
velero install \
  --use-node-agent \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.9.0 \
  --bucket velero-backups \
  --secret-file ./credentials-velero \
  --use-volume-snapshots=false \
  --backup-location-config region=minio,s3ForcePathStyle="true",s3Url=http://192.168.1.100:9000
```
**The first thing we need to do is make sure that the "pipe" is properly connected between the cluster and the other machine:**

```
   velero backup-location get
 ```
**If you want to back up the entire cluster (with all namespaces) and take a backup with File System Backup enabled.**

```
velero backup create full-cluster-with-data --default-volumes-to-fs-backup
```
 **If you want to back up a specific application (one namespace):**
 Replace my-app with your namespace.
 
```
velero backup create app-backup --include-namespaces my-app
```

**To confirm that the backup was completed successfully:**
```
velero backup get
```

