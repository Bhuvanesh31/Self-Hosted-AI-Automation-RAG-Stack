# RAG (Retrieval-Augmented Generation) Flow

This document explains **how data flows through the system for Retrieval‑Augmented Generation (RAG)**.

It focuses on **conceptual clarity and operational correctness**, not framework‑specific code. The same flow applies whether queries are triggered via UI, API, or automation.

---

## 1. What RAG Means in This System

Retrieval‑Augmented Generation (RAG) is the process of:

1. **Storing knowledge externally** (documents)
2. **Retrieving relevant context** at query time
3. **Injecting that context into an LLM prompt**

In this system:
- **MinIO** stores raw documents
- **Qdrant** stores vector embeddings
- **Ollama** generates responses
- **n8n** orchestrates the flow

---

## 2. High‑Level RAG Architecture

```
Document Source
      ↓
   MinIO (raw files)
      ↓
Ingestion Pipeline (n8n)
      ↓
Embeddings (Ollama or embedding model)
      ↓
Qdrant (vectors + metadata)
      ↓

Query Time
      ↓
User / Agent Query
      ↓
n8n Orchestrator
      ↓
Qdrant Similarity Search
      ↓
Relevant Chunks
      ↓
Prompt Construction
      ↓
Ollama (LLM)
      ↓
Final Answer
```

---

## 3. Data Ingestion Flow (Indexing)

### 3.1 Source Documents

Documents may come from:
- Manual uploads
- APIs
- File sync jobs
- Internal systems

All source files are stored **unchanged** in MinIO.

MinIO acts as the **source of truth** for raw content.

---

### 3.2 Chunking

Large documents are split into smaller chunks.

Typical chunking rules:
- Fixed token size (e.g. 500–1,000 tokens)
- Optional overlap to preserve context

Chunking is deterministic so embeddings can be regenerated.

---

### 3.3 Embedding Generation

Each chunk is converted into a vector embedding using:
- A local embedding model via Ollama

The output is a numeric vector representing semantic meaning.

---

### 3.4 Vector Storage (Qdrant)

Each chunk is stored in Qdrant with:
- Vector embedding
- Metadata:
  - Document ID
  - Chunk index
  - Source reference
  - Optional tags

Qdrant becomes the **semantic index**.

---

## 4. Query Flow (Retrieval + Generation)

### 4.1 Query Initiation

Queries can originate from:
- Open WebUI
- n8n workflows
- External APIs
- AI agents

The raw query text enters n8n.

---

### 4.2 Query Embedding

The user query is embedded using the same embedding model.

This ensures compatibility between query and stored vectors.

---

### 4.3 Similarity Search

n8n queries Qdrant with:
- Query vector
- Top‑K parameter (e.g. top 5 results)
- Optional metadata filters

Qdrant returns the most relevant chunks.

---

### 4.4 Context Assembly

Retrieved chunks are:
- Ranked
- Cleaned
- Concatenated

They are inserted into a prompt template.

Example structure:

```
SYSTEM: You are an assistant...

CONTEXT:
<retrieved chunks>

QUESTION:
<user query>
```

---

### 4.5 Generation (Ollama)

The constructed prompt is sent to Ollama.

Ollama:
- Runs the LLM locally
- Generates a response grounded in retrieved context

---

### 4.6 Response Handling

The response is:
- Returned to the user
- Optionally logged
- Optionally stored as memory

No document state is modified during query time.

---

## 5. Role of Each Component in RAG

| Component | Role |
|--------|------|
| MinIO | Raw document storage |
| Qdrant | Vector similarity search |
| Ollama | Embeddings + text generation |
| n8n | Orchestration & control |
| PostgreSQL | Workflow & metadata |
| Redis | Short‑term execution state |

---

## 6. Stateless vs Stateful Boundaries

### Stateless
- Query execution
- Prompt construction
- Generation

### Stateful
- Stored documents (MinIO)
- Embeddings (Qdrant)
- Workflow state (Postgres)

Understanding this separation prevents accidental corruption.

---

## 7. Re‑indexing Strategy

Re‑indexing is required when:
- Embedding model changes
- Chunking strategy changes
- Source documents change

Re‑indexing means:
1. Re‑chunk documents
2. Regenerate embeddings
3. Replace Qdrant collections

MinIO remains unchanged.

---

## 8. Failure Scenarios

### Qdrant Down
- Queries fail or return no context
- LLM may fall back to generic answers

### MinIO Down
- New ingestion fails
- Existing embeddings still queryable

### Ollama Down
- No embeddings or generation

Failures are isolated by design.

---

## 9. Anti‑Patterns to Avoid

- Storing full documents in Qdrant
- Storing embeddings in PostgreSQL
- Mixing multiple embedding models in one collection
- Mutating source documents during queries

These break correctness.

---

## 10. Summary

This RAG flow ensures:
- Deterministic ingestion
- Reproducible embeddings
- Clean separation of concerns
- Local, private AI reasoning

The system remains explainable, debuggable, and scalable.

