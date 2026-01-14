# Storage and Data Model

This document explains **where all data is stored**, **what type of data it is**, and **what happens if it is modified or deleted**.

This is the **most critical operational document**. Misunderstanding storage is the fastest way to lose data.

---

## 1. Core Principle

> **Containers are disposable. Data is not.**

- Containers can be recreated at any time
- Images can be upgraded safely
- The entire system can be rebuilt

As long as the **data directory is intact**, the system survives.

---

## 2. Single Source of Persistent Data

All persistent data lives under **one directory** on the host:

```
~/ai-stack/data/
```

Nothing outside this directory is required to recover the system.

---

## 3. Directory-Level Breakdown

Each subdirectory maps **one-to-one** with a service.

---

### 3.1 `data/postgres/`

**Contains**
- PostgreSQL database files
- Tables for n8n workflows
- Encrypted credentials
- Agent state

**Written by**
- PostgreSQL container

**Deletion Impact**
- ❌ All workflows lost
- ❌ All credentials lost
- ❌ Agent memory lost

**Severity**: Critical

---

### 3.2 `data/redis/`

**Contains**
- Append-only file (AOF)
- Redis persistence data

**Written by**
- Redis container

**Deletion Impact**
- Loss of short-term memory
- Cache rebuilt automatically

**Severity**: Medium

---

### 3.3 `data/minio/`

**Contains**
- Buckets
- Raw documents (PDFs, text, audio, etc.)

**Written by**
- MinIO container

**Deletion Impact**
- ❌ All source documents lost
- RAG ingestion must be redone

**Severity**: Critical

---

### 3.4 `data/qdrant/`

**Contains**
- Vector embeddings
- Index files
- Metadata filters

**Written by**
- Qdrant container

**Deletion Impact**
- ❌ All embeddings lost
- Semantic search broken

**Severity**: Critical

---

### 3.5 `data/ollama/`

**Contains**
- Downloaded LLM models
- Model cache

**Written by**
- Ollama container

**Deletion Impact**
- Models must be re-downloaded
- No data corruption

**Severity**: Low

---

### 3.6 `data/n8n/`

**Contains**
- n8n user configuration
- Encryption artifacts
- Execution metadata

**Primary state lives in PostgreSQL**.

**Deletion Impact**
- UI settings reset
- Database-backed workflows remain

**Severity**: Medium

---

### 3.7 `data/open-webui/`

**Contains**
- UI preferences
- Conversation history

**Deletion Impact**
- UI resets
- No system-wide impact

**Severity**: Low

---

## 4. What Is NOT Persistent

The following are **intentionally ephemeral**:

- Docker containers
- Docker images
- Docker network
- Container logs (unless explicitly configured)

These can be safely removed or rebuilt.

---

## 5. Safe vs Unsafe Actions

### Safe Actions

- `docker compose down`
- `docker compose up -d`
- `docker compose pull`
- Rebooting the host
- Upgrading container images

---

### Unsafe Actions

- Deleting `data/`
- Deleting individual subfolders without intent
- `docker rm -v`
- Running containers without volumes

---

## 6. Backup Model

### What Must Be Backed Up

At minimum:

```
~/ai-stack/data/
```

This single directory is sufficient to recover the system.

---

### Manual Backup Command

```bash
tar czvf ai-stack-backup-$(date +%F).tar.gz ~/ai-stack/data
```

---

### Restore Procedure (High Level)

1. Install Docker
2. Restore `data/` directory
3. Restore `.env`
4. Run `docker compose up -d`

System state will be restored.

---

## 7. Storage Anti-Patterns (Do Not Do These)

- Storing files in PostgreSQL
- Storing embeddings in Redis
- Writing application state outside `data/`
- Editing files inside containers

These break the recovery model.

---

## 8. Mental Model Summary

- `data/` = the system’s memory
- Containers = execution shells
- Deleting containers is safe
- Deleting data is destructive

If you protect `data/`, you protect the system.

---

## 9. Summary

This storage model ensures:
- Simple backups
- Predictable recovery
- Clear blast radius for failures

Understand this document before touching disk-level data.

