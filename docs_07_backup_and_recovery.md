# Backup and Recovery

This document defines **how the system is backed up**, **how it is restored**, and **what guarantees exist during failure scenarios**.

If this document is followed, **total system loss is recoverable**.

---

## 1. Recovery Philosophy

The system is designed so that:

- All critical state lives in one place
- Backups are simple and verifiable
- Recovery does not depend on running containers

> If you have the backup and Docker, you can recover the system.

---

## 2. What Must Be Backed Up

### Mandatory Backup Scope

Only **one directory** is required:

```
~/ai-stack/data/
```

This directory contains:
- Databases
- Vector indexes
- Documents
- Models
- Runtime state

Backing up anything else is optional.

---

### Optional Backup Files

These files are recommended but not strictly required:

- `.env` – credentials and configuration
- `docker-compose.yml` – system definition

Without these, the system can still be rebuilt, but manual reconfiguration is required.

---

## 3. Backup Types

### 3.1 Manual Snapshot Backup

Used for:
- Before upgrades
- Before major changes
- Periodic safety snapshots

#### Command

```bash
tar czvf ai-stack-backup-$(date +%F).tar.gz ~/ai-stack/data
```

This creates a compressed archive of all persistent data.

---

### 3.2 Automated Scheduled Backup (Recommended)

Use cron to schedule daily backups.

Example:

```bash
0 2 * * * tar czvf /backups/ai-stack-$(date +\%F).tar.gz /home/<user>/ai-stack/data
```

Store backups on:
- External disk
- Network storage
- Another machine

Do not store backups only on the same disk.

---

## 4. Backup Verification

A backup is useless if it cannot be restored.

Verification steps:

1. Ensure archive exists
2. Check archive integrity
3. Periodically test restore on another machine

Example:

```bash
tar tzf ai-stack-backup-YYYY-MM-DD.tar.gz > /dev/null
```

---

## 5. Recovery Scenarios

### 5.1 Container Failure

**Scenario:** One or more containers crash.

**Action:**

```bash
docker compose up -d
```

Data is unaffected.

---

### 5.2 Host Reboot or Power Loss

**Scenario:** Machine restarts unexpectedly.

**Action:** None.

Docker restarts automatically and brings all services up.

---

### 5.3 Docker Reinstallation

**Scenario:** Docker is removed or corrupted.

**Action:**

1. Reinstall Docker
2. Navigate to `ai-stack`
3. Run:

```bash
docker compose up -d
```

Volumes are reused automatically.

---

### 5.4 Full Host Failure (New Machine)

**Scenario:** Disk failure or migration to a new server.

**Recovery Steps:**

1. Install Ubuntu 22.04 LTS
2. Install Docker
3. Restore `ai-stack/data/`
4. Restore `.env`
5. Restore `docker-compose.yml`
6. Run:

```bash
docker compose up -d
```

System state is restored.

---

## 6. What Is NOT Backed Up Automatically

- Docker images (can be re-pulled)
- Containers (ephemeral)
- Docker networks

These are recreated automatically.

---

## 7. Disaster Recovery Drill (Recommended)

At least once:

1. Stop the system
2. Move `data/` aside
3. Restore from backup
4. Start the system
5. Verify services

This validates recovery assumptions.

---

## 8. Common Backup Mistakes

- Backing up containers instead of data
- Forgetting vector database data
- Not testing restores
- Keeping backups on the same disk

These invalidate recovery guarantees.

---

## 9. Summary

If you back up:

```
~/ai-stack/data/
```

You can recover from:
- Container loss
- Docker loss
- Host loss

Without backups, data loss is permanent.

