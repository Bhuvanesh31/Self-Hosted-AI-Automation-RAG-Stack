# Networking Model

This document explains **how networking works in the system**, including:
- How services find each other
- Why some URLs work only inside containers
- How host access differs from container access

Most operational confusion in Docker-based systems comes from misunderstanding networking. This document removes that ambiguity.

---

## 1. Core Networking Principles

The system uses **Docker’s bridge networking** with **Docker DNS**.

Key rules:

1. Containers communicate using **service names**, not IP addresses
2. Docker DNS works **only inside Docker networks**
3. The host uses **localhost and exposed ports**
4. `localhost` inside a container refers to the container itself

---

## 2. Docker Network Overview

### Network Name

```
ai-stack_ai_net
```

This network is automatically created by Docker Compose.

All services are attached to this single private network.

---

### What the Network Provides

- Private communication between services
- Automatic DNS resolution
- Network isolation from the host

No service needs to know IP addresses.

---

## 3. Internal (Container-to-Container) Networking

Inside Docker, services talk to each other using:

```
http://<service-name>:<port>
```

These names come from the `docker-compose.yml` service keys.

### Examples

- Qdrant: `http://qdrant:6333`
- MinIO API: `http://minio:9000`
- Ollama: `http://ollama:11434`
- PostgreSQL: `postgres:5432`
- Redis: `redis:6379`

These addresses **only work inside containers**.

---

## 4. Host-to-Container Networking

The host cannot use Docker DNS.

Instead, services must **explicitly expose ports**.

### Exposed Ports in This System

| Service | Host Port | Container Port |
|------|----------|----------------|
| n8n | 5678 | 5678 |
| Open WebUI | 3000 | 8080 |
| Ollama | 11434 | 11434 |
| Qdrant | 6333 | 6333 |
| MinIO API | 9000 | 9000 |
| MinIO Console | 9001 | 9001 |
| Traefik | 80 | 80 |

Host access always uses:

```
http://localhost:<port>
```

---

## 5. Why `localhost` Behaves Differently

### On the Host

```
localhost → your machine
```

### Inside a Container

```
localhost → that container only
```

This means:

- `curl localhost:6333` inside n8n ❌ (unless qdrant is in same container)
- `curl qdrant:6333` inside n8n ✅
- `curl qdrant:6333` on host ❌
- `curl localhost:6333` on host ✅

This distinction is critical.

---

## 6. Service Discovery Summary

| Context | Correct Address | Incorrect Address |
|------|----------------|------------------|
| Container → Qdrant | `qdrant:6333` | `localhost:6333` |
| Container → MinIO | `minio:9000` | `localhost:9000` |
| Host → Qdrant | `localhost:6333` | `qdrant:6333` |
| Host → MinIO | `localhost:9000` | `minio:9000` |

---

## 7. Traefik and Routing (Conceptual)

Traefik acts as a **reverse proxy**.

Currently:
- It exposes port 80
- It can route traffic based on hostnames

Future capabilities:
- HTTPS (Let’s Encrypt)
- Authentication
- Rate limiting

Traefik does not change internal Docker DNS behavior.

---

## 8. Common Networking Mistakes

- Using `localhost` inside containers
- Using service names from the host
- Exposing ports unnecessarily
- Hardcoding IP addresses

All of these break portability.

---

## 9. Debugging Checklist

When something "can’t connect":

1. Is the container running?
2. Is it attached to `ai-stack_ai_net`?
3. Are you using the correct address for the context?
4. Is the port exposed (for host access)?

Most issues resolve by answering these.

---

## 10. Summary

- Docker DNS works only inside Docker
- The host uses localhost and ports
- Service names are the stable contract
- IP addresses should never be used

If you understand this document, Docker networking will stop being confusing.

