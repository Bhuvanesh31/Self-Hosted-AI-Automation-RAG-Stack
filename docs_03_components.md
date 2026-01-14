# Components Reference

This document describes **each installed component in isolation**.

For every service, it explains:
- Why it exists
- What responsibility it owns
- Where its data is stored
- What depends on it
- What happens if it fails or is deleted

This is the **definitive reference** for understanding the system’s building blocks.

---

## 1. PostgreSQL

### Purpose
PostgreSQL is the **primary persistent database** for the system.

It stores:
- n8n workflows
- n8n credentials (encrypted)
- Agent state
- Long-lived metadata

### Why PostgreSQL
- ACID compliance
- Strong consistency
- Mature ecosystem
- Safe upgrades

### Storage Location
```
~/ai-stack/data/postgres/
```

### Depends On
- Docker volume mount

### Used By
- n8n

### Failure Impact
- n8n will not start
- All workflows and credentials become unavailable

### Deletion Impact
- Permanent loss of workflows and state

---

## 2. Redis

### Purpose
Redis provides **fast, in-memory storage** for short-lived data.

Used for:
- Caching
- Queues
- Temporary agent memory

### Persistence Model
- Append-only file (AOF) enabled
- Data survives restarts

### Storage Location
```
~/ai-stack/data/redis/
```

### Used By
- n8n
- Future background job systems

### Failure Impact
- Temporary performance degradation
- Some workflows may retry or fail

### Deletion Impact
- Loss of short-term memory only

---

## 3. MinIO

### Purpose
MinIO is the **object storage layer**.

It stores:
- PDFs
- Text documents
- Audio files
- Raw data for RAG ingestion

### Why MinIO
- S3-compatible API
- High performance
- No cloud dependency

### Ports
- 9000: API
- 9001: Web console

### Storage Location
```
~/ai-stack/data/minio/
```

### Used By
- RAG ingestion workflows
- n8n

### Failure Impact
- RAG ingestion stops
- Existing embeddings remain but source documents are inaccessible

### Deletion Impact
- All source documents lost
- Re-ingestion required

---

## 4. Qdrant

### Purpose
Qdrant is the **vector database**.

It stores:
- Vector embeddings
- Metadata
- Similarity indexes

This is the core of Retrieval-Augmented Generation (RAG).

### Why Qdrant
- Purpose-built for vectors
- Fast similarity search
- Strong filtering support

### Port
- 6333

### Storage Location
```
~/ai-stack/data/qdrant/
```

### Used By
- RAG query workflows
- AI agents

### Failure Impact
- Semantic search unavailable
- LLMs fall back to zero-context answers

### Deletion Impact
- All embeddings lost
- Full re-embedding required

---

## 5. Ollama

### Purpose
Ollama is the **local LLM runtime**.

It:
- Downloads models
- Runs inference locally
- Exposes a REST API

### Why Ollama
- Local execution
- Simple model management
- GPU and CPU support

### Port
- 11434

### Storage Location
```
~/ai-stack/data/ollama/
```

### Used By
- Open WebUI
- n8n (AI calls)

### Failure Impact
- No LLM responses
- Automations requiring AI fail

### Deletion Impact
- Models must be re-downloaded

---

## 6. Open WebUI

### Purpose
Open WebUI provides a **human-facing interface** for LLM interaction.

Used for:
- Prompt testing
- Model selection
- Debugging AI behavior

### Port
- 3000

### Storage Location
```
~/ai-stack/data/open-webui/
```

### Depends On
- Ollama

### Failure Impact
- No UI for human interaction
- APIs remain functional

### Deletion Impact
- UI preferences reset

---

## 7. n8n

### Purpose
n8n is the **automation and orchestration engine**.

It acts as:
- Control plane
- Agent executor
- Integration layer

Used for:
- Triggers
- Tool calling
- RAG pipelines
- Scheduled jobs

### Port
- 5678

### Storage Location
```
~/ai-stack/data/n8n/
```

### Primary State
- Stored in PostgreSQL

### Depends On
- PostgreSQL
- Redis

### Failure Impact
- All automations stop

### Deletion Impact
- Workflow configuration lost

---

## 8. Traefik

### Purpose
Traefik is the **reverse proxy and entry point**.

Currently used for:
- Routing
- Future HTTPS termination

### Port
- 80

### Storage
- Stateless in current configuration

### Failure Impact
- Services remain reachable via direct ports

### Deletion Impact
- No data loss

---

## 9. Component Dependency Summary

| Component | Depends On | Critical |
|--------|------------|----------|
| PostgreSQL | Docker | Yes |
| Redis | Docker | Medium |
| MinIO | Docker | Yes |
| Qdrant | Docker | Yes |
| Ollama | Docker | Yes |
| Open WebUI | Ollama | No |
| n8n | Postgres, Redis | Yes |
| Traefik | Docker | No |

---

## 10. Summary

Each component has a **single, clear responsibility**.

No component:
- Overlaps another’s role
- Stores data it should not
- Depends on hidden state

Understanding these components is required before modifying or scaling the system.

