# Self-Hosted AI Automation & RAG Stack

> **Status: reference guide, not a runnable repo.** This is written documentation for a local, Docker-based AI/RAG setup — the `docker-compose.yml` and configs described below aren't committed here yet. Use it as a build plan and adapt the compose file to your own environment.

Local, free, Docker-based platform for **AI agents, automations, and RAG systems**.

This repository documents a **restart-safe, non-breakable** setup using only open-source components. Everything runs locally via Docker. No cloud dependencies. No paid services.

---

## What This Repo Provides

* **Workflow automation** using n8n
* **Local LLM runtime** using Ollama
* **Human UI for LLMs** via Open WebUI
* **RAG infrastructure** with Qdrant (vectors) + MinIO (documents)
* **Reliable state & memory** via PostgreSQL + Redis
* **Single-command lifecycle** using Docker Compose

Designed for:

* Internal tools
* AI agents
* RAG systems
* Experimentation that can safely evolve into production

---

## High-Level Architecture

```
User / Browser
   ↓
Open WebUI        n8n
   ↓               ↓
Ollama (LLM)   Automations / Agents
        ↓         ↓
        Qdrant (Vectors)
             ↓
          MinIO (Files)
             ↓
        PostgreSQL / Redis
```

**Key principle:** Containers are disposable. Data is not.

---

## Quick Start (Local)

```bash
git clone <your-repo-url>
cd ai-stack
cp .env.example .env
docker compose up -d
```

That’s it. No post-boot steps.

---

## Access URLs

| Service       | URL                                              |
| ------------- | ------------------------------------------------ |
| n8n           | [http://localhost:5678](http://localhost:5678)   |
| Open WebUI    | [http://localhost:3000](http://localhost:3000)   |
| Ollama API    | [http://localhost:11434](http://localhost:11434) |
| Qdrant        | [http://localhost:6333](http://localhost:6333)   |
| MinIO Console | [http://localhost:9001](http://localhost:9001)   |

---

## Repository Structure

```
ai-stack/
├── README.md
├── docker-compose.yml
├── .env.example
├── data/                 # runtime data (gitignored)
└── docs/
    ├── 01-system-overview.md
    ├── 02-installation.md
    ├── 03-components.md
    ├── 04-storage-and-data.md
    ├── 05-networking.md
    ├── 06-operations.md
    ├── 07-backup-and-recovery.md
    ├── 08-rag-flow.md
    └── 09-security.md
```

---

## Documentation Index

Start here, in order:

1. **System overview** → `docs/01-system-overview.md`
2. **Installation guide** → `docs/02-installation.md`
3. **Installed components** → `docs/03-components.md`
4. **Data & storage model** → `docs/04-storage-and-data.md`
5. **Networking & ports** → `docs/05-networking.md`
6. **Day-2 operations** → `docs/06-operations.md`
7. **Backup & recovery** → `docs/07-backup-and-recovery.md`
8. **RAG data flow** → `docs/08-rag-flow.md`
9. **Security notes** → `docs/09-security.md`

---

## Design Guarantees

* Fully self-hosted
* Free & open source
* Restart-safe (reboot-proof)
* No manual intervention after setup
* Clear separation of concerns

---

## What This Repo Is NOT

* A managed cloud service
* A click-and-forget SaaS
* A beginner abstraction layer

This repo assumes you value **control, clarity, and ownership** over convenience.

---

## Next Steps

* Read the docs in order
* Build your first RAG ingestion pipeline
* Add AI agents via n8n
* Harden security when exposing beyond localhost

---

## License

Open-source components only. No proprietary dependencies.
