# Installation Guide

This document explains **how the system is installed from scratch** in a deterministic, repeatable way.

If followed exactly, the result is a **restart-safe, non-breakable system** that requires **no manual steps after startup**.

---

## 1. Installation Philosophy

The installation follows these rules:

- Install the **minimum required on the host**
- Run **everything else in containers**
- Make all state explicit and persistent
- Avoid one-off or interactive steps

If installation is done correctly, the system:
- Survives reboots
- Survives Docker restarts
- Survives container recreation

---

## 2. Prerequisites

### Hardware (Minimum Recommended)

- CPU: 4 cores (8 preferred)
- RAM: 16 GB (8 GB workable, tight)
- Disk: 100 GB SSD minimum
- GPU: Optional (NVIDIA supported for Ollama)

### Operating System

- Ubuntu Server 22.04 LTS
- Fresh install preferred

No desktop environment is required.

---

## 3. Host System Preparation

### 3.1 System Update

```bash
sudo apt update && sudo apt upgrade -y
```

### 3.2 Required Host Packages

```bash
sudo apt install -y \
  ca-certificates \
  curl \
  gnupg \
  lsb-release
```

These are used only to install Docker. They do not participate in runtime behavior.

---

## 4. Docker Installation (Foundation)

### 4.1 Remove Old Docker Versions (if any)

```bash
sudo apt remove -y docker docker.io docker-engine containerd runc
```

### 4.2 Install Docker Engine

```bash
curl -fsSL https://get.docker.com | sudo sh
```

### 4.3 Enable Docker at Boot

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

### 4.4 Run Docker Without sudo

```bash
sudo usermod -aG docker $USER
newgrp docker
```

Log out and back in if needed.

### 4.5 Verify Docker

```bash
docker run hello-world
```

If this fails, stop here and fix Docker before continuing.

---

## 5. Project Directory Setup

### 5.1 Create Root Directory

```bash
mkdir -p ~/ai-stack
cd ~/ai-stack
```

This directory represents the entire system.

---

### 5.2 Create Persistent Data Directories

```bash
mkdir -p data/{postgres,redis,qdrant,minio,ollama,n8n,open-webui}
```

All persistent state will live here.

**Rule:** Never delete `data/` unless you intentionally want to destroy system state.

---

## 6. Environment Configuration

### 6.1 Create `.env` File

```bash
cp .env.example .env
```

Edit `.env` and set:
- Database credentials
- n8n auth credentials
- Encryption key
- MinIO credentials

This file is the **single source of truth** for configuration.

---

## 7. Docker Compose Definition

### 7.1 Required Files

The following files must exist:

- `docker-compose.yml`
- `.env`

`docker-compose.yml` defines:
- All services
- Volumes
- Networks
- Restart behavior

---

## 8. First Boot

### 8.1 Pull Images

```bash
docker compose pull
```

### 8.2 Start the System

```bash
docker compose up -d
```

Docker will:
- Create a private network
- Start all services
- Attach volumes

---

## 9. Verification

Wait ~60 seconds, then verify.

### 9.1 Container Status

```bash
docker ps
```

All containers should be in `Up` state.

---

### 9.2 Service Access (Host)

| Service | URL |
|------|-----|
| n8n | http://localhost:5678 |
| Open WebUI | http://localhost:3000 |
| Qdrant | http://localhost:6333 |
| MinIO Console | http://localhost:9001 |
| Ollama API | http://localhost:11434 |

---

## 10. Restart Safety Test (Mandatory)

This confirms the system is non-breakable.

```bash
docker compose down
docker compose up -d
```

Then reboot the host:

```bash
sudo reboot
```

After reboot, all services should be available at the same URLs without manual intervention.

---

## 11. What Installation Does NOT Do

Installation intentionally does NOT:
- Secure public access
- Configure HTTPS
- Create RAG pipelines
- Configure AI agents

Those are layered on top after the system is stable.

---

## 12. Common Installation Mistakes

- Running applications on the host
- Editing containers manually
- Forgetting to mount volumes
- Deleting `data/`

Avoiding these keeps the system stable.

---

## 13. Next Steps

Once installation is complete:

- Read `03-components.md` to understand each service
- Review `04-storage-and-data.md` before touching files
- Proceed to RAG or agent workflows only after verification

---

## 14. Summary

If this document is followed exactly:

- Installation is deterministic
- Restarts are safe
- No hidden state exists

The system is now ready for real workloads.

