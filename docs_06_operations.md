# Operations (Day-2 Runbook)

This document describes **how to operate the system after installation**.

It covers:
- Starting and stopping services
- Safe and unsafe commands
- Upgrading containers
- Basic troubleshooting

If installation is Day‑1, this is everything that happens after.

---

## 1. Operational Philosophy

The system is designed so that:

- Containers are replaceable
- Configuration is declarative
- Data persistence is explicit
- Manual intervention is minimized

Operations should always modify **configuration or images**, never running containers.

---

## 2. Normal Lifecycle Commands

All commands are run from the project root:

```
~/ai-stack/
```

### Start the System

```bash
docker compose up -d
```

Starts all services in detached mode.

---

### Stop the System (Safe)

```bash
docker compose down
```

- Containers stop
- Network removed
- Volumes preserved

No data loss.

---

### Restart the System

```bash
docker compose restart
```

Or:

```bash
docker compose down
docker compose up -d
```

Both are safe.

---

## 3. System Boot Behavior

Docker is enabled at system startup.

Because all services use:

```
restart: unless-stopped
```

This guarantees:

- Containers restart after Docker restart
- Containers restart after host reboot
- Crashed containers restart automatically

No manual action is required after boot.

---

## 4. Logs and Inspection

### List Running Containers

```bash
docker ps
```

---

### View Logs for a Service

```bash
docker logs <container-name>
```

Example:

```bash
docker logs ai-stack-n8n-1
```

---

### Follow Logs (Live)

```bash
docker logs -f ai-stack-ollama-1
```

---

## 5. Upgrading Services

Upgrades are image-based and non-destructive.

### Upgrade All Services

```bash
docker compose pull
docker compose up -d
```

Docker will:
- Pull newer images
- Recreate containers
- Reattach existing volumes

Data remains intact.

---

### Upgrade a Single Service

```bash
docker compose pull <service-name>
docker compose up -d <service-name>
```

Example:

```bash
docker compose pull n8n
docker compose up -d n8n
```

---

## 6. Configuration Changes

### Editing Configuration

- Edit `.env` or `docker-compose.yml`
- Never edit files inside containers

Apply changes with:

```bash
docker compose up -d
```

---

## 7. Safe vs Unsafe Operations

### Safe Operations

- Rebooting the host
- Pulling new images
- Restarting containers
- Deleting containers (without volumes)

---

### Unsafe Operations

- `docker rm -v`
- Deleting `data/`
- Editing files inside containers
- Running services without volume mounts

Unsafe operations break recoverability.

---

## 8. Failure Scenarios and Response

### Container Won’t Start

1. Check logs
2. Verify environment variables
3. Confirm volumes exist

---

### Service Is Slow or Unresponsive

1. Check container logs
2. Check system resources (CPU/RAM)
3. Restart the affected service

---

### Docker Networking Issues

1. Confirm containers are on the same network
2. Use service names inside containers
3. Use localhost only from the host

---

## 9. Health Verification Checklist

After any operation:

- `docker ps` shows all containers running
- n8n UI loads
- Open WebUI loads
- Ollama responds on port 11434
- Qdrant responds on port 6333
- MinIO console loads

---

## 10. Operational Anti-Patterns

Avoid:
- SSHing into containers to "fix" issues
- Storing data outside volumes
- Making undocumented changes

If a fix cannot be expressed in compose or env, it is not a real fix.

---

## 11. Summary

Day‑2 operations are intentionally boring:

- Start
- Stop
- Upgrade
- Observe

If operations feel complex, something is misconfigured.

