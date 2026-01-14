# System Overview

This document explains **what the system is**, **why it exists**, and **how it is designed at a high level**. It intentionally avoids setup commands and low-level details.

If you want to *run* the system, see `02-installation.md`.

---

## 1. What This System Is

This is a **fully self-hosted AI automation and RAG platform** designed to run entirely on a local machine or private server.

It combines:
- Workflow automation
- Local large language models (LLMs)
- Retrieval-Augmented Generation (RAG)
- Persistent memory and storage

All components are:
- Free and open source
- Containerized using Docker
- Restart-safe and reproducible

There are **no cloud dependencies** and **no paid services**.

---

## 2. What Problems This System Solves

This system exists to solve the following problems:

1. **Running LLMs locally** without vendor lock-in
2. **Building AI agents** that can call tools and remember state
3. **Automating workflows** that involve AI, APIs, and data
4. **Implementing RAG** using your own documents
5. **Operating reliably** without fragile manual steps

It is designed to grow from experimentation into production without re-architecture.

---

## 3. Design Principles

The system follows a few strict principles. These are non-negotiable.

### 3.1 Containers Are Disposable

- Containers can be stopped, removed, or recreated at any time
- No important data lives inside containers
- Rebuilding containers must never cause data loss

### 3.2 Data Is Persistent and Explicit

- All state lives on the host filesystem via mounted volumes
- Every persistent directory is intentional and documented
- Backups are simple because data is centralized

### 3.3 The Host Is Dumb

- The host OS does not run applications directly
- Its only job is to:
  - Run Docker
  - Store volumes
  - Provide networking

### 3.4 One Source of Truth

- `docker-compose.yml` defines the system
- `.env` defines configuration and secrets
- No hidden configuration inside containers

---

## 4. High-Level Architecture

### Logical View

```
User / Browser
   ↓
Open WebUI        n8n
   ↓               ↓
Ollama (LLM)   Automation / Agents
        ↓         ↓
        Qdrant (Vectors)
             ↓
          MinIO (Files)
             ↓
        PostgreSQL / Redis
```

This diagram represents **logical dependencies**, not network topology.

---

## 5. Core Functional Areas

### 5.1 Automation Layer

- Implemented using **n8n**
- Acts as the system’s orchestration and control plane
- Responsible for:
  - Triggers (HTTP, schedule, events)
  - Tool execution
  - AI agent workflows

n8n is where business logic lives.

---

### 5.2 AI / LLM Layer

- Implemented using **Ollama**
- Runs local LLMs (CPU or GPU)
- Exposes a clean REST API

**Open WebUI** sits on top of this layer to provide:
- Human interaction
- Prompt testing
- Model selection

---

### 5.3 Retrieval (RAG) Layer

This layer enables Retrieval-Augmented Generation.

- **MinIO** stores raw documents (PDFs, text, audio, etc.)
- **Qdrant** stores vector embeddings and metadata

Together, they allow:
- Ingesting documents
- Searching semantically
- Injecting context into LLM prompts

---

### 5.4 State & Memory Layer

- **PostgreSQL** stores long-lived state
  - Workflows
  - Credentials
  - Agent memory

- **Redis** provides fast, short-lived memory
  - Caching
  - Queues
  - Temporary context

This separation prevents overloading a single datastore.

---

## 6. Networking Model (Conceptual)

- All services run on a private Docker bridge network
- Services discover each other via Docker DNS
- Internal communication uses service names (not IPs)

Example (inside containers):
```
http://qdrant:6333
http://minio:9000
http://ollama:11434
```

The host accesses services only via explicitly exposed ports.

---

## 7. Lifecycle Guarantees

The system is designed to survive:

- Container restarts
- Docker restarts
- Host reboots
- Power loss

This is achieved by:
- Persistent volumes
- Deterministic configuration
- Automatic restart policies

No manual intervention is required after setup.

---

## 8. What This Document Does NOT Cover

This document intentionally does **not** include:
- Installation commands
- Environment variables
- Backup procedures
- Security hardening
- RAG implementation details

Those are covered in later documents.

---

## 9. How to Read the Rest of the Docs

Recommended order:

1. `02-installation.md` – How the system is installed
2. `03-components.md` – What each service does in detail
3. `04-storage-and-data.md` – Where data lives and why
4. `05-networking.md` – How services communicate

This overview provides the mental model. The rest provide execution details.

---

## 10. Summary

This system is:
- Modular
- Deterministic
- Self-contained
- Production-capable

It prioritizes **clarity, control, and recoverability** over convenience.

If you understand this document, you understand the system.

