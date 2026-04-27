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
### The setup consists of:
1. **Source Cluster:** Any K8s cluster (EKS, K3s, or Kubeadm).
2. **Backup Server:** A separate VM running MinIO Server.
3. **Velero:** Installed on the cluster to orchestrate the backup/restore process.

##📖 Step-by-Step Implementation
Step 1: Prepare the Backup Storage (MinIO)
On your Storage VM (e.g., 192.168.1.100), we install MinIO to act as our S3-Compatible vault.

Install MinIO Binary:

Bash
wget [https://dl.min.io/server/minio/release/linux-amd64/minio](https://dl.min.io/server/minio/release/linux-amd64/minio)
chmod +x minio
sudo mv minio /usr/local/bin/
Setup Systemd Service: (Check scripts/install-minio.sh for full automation).

Configure Access:

Console: http://192.168.1.100:9001

Default Login: minioadmin / minioadmin

Action Required: Create a bucket named velero-backups.

Step 2: Install Velero on Kubernetes
On your Master Node/Control Plane, configure Velero to point to your MinIO server.

Create Credentials File (credentials-velero):

Ini, TOML
[default]
aws_access_key_id = minioadmin
aws_secret_access_key = minioadmin
Execute Installation:

Bash
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.9.0 \
  --bucket velero-backups \
  --secret-file ./credentials-velero \
  --use-node-agent \
  --use-volume-snapshots=false \
  --backup-location-config region=minio,s3ForcePathStyle="true",s3Url=[http://192.168.1.100:9000](http://192.168.1.100:9000)
Note: --use-node-agent is mandatory to back up Persistent Volume data via File-System Backup.

Step 3: Performing Backups
Capture the entire cluster state or specific workloads.

Manual Full Backup (Resources + Data):

Bash
velero backup create full-cluster-backup --default-volumes-to-fs-backup
Check Backup Status:

Bash
velero backup describe full-cluster-backup
Step 4: The Recovery (Restore)
If your cluster fails or you need to migrate to a new environment:

Connect to the New Cluster.

Re-install Velero using the same command in Step 2.

Trigger the Restoration:

Bash
# View available backups synced from MinIO
velero backup get

# Restore everything from the specific backup
velero restore create --from-backup full-cluster-backup
🛡 Best Practices & Maintenance
TTL (Time to Live): Save storage by setting an expiration date (e.g., 30 days):
--ttl 720h0m0s

Schedules: Automate backups with Cron syntax:

Bash
velero schedule create daily-bkp --schedule="0 2 * * *"
Monitoring: Check for volume-level errors:

Bash
velero backup logs <backup-name>
