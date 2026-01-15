# Security Model and Hardening Notes

This document explains the **current security posture**, **explicit trust boundaries**, and **how to harden the system when required**.

The goal is not theoretical security. The goal is **clear, honest, and controllable security**.

---

## 1. Security Philosophy

This system follows three principles:

1. **Security is contextual** — local systems and public systems have different needs
2. **Clarity beats obscurity** — every exposed surface is intentional
3. **Hardening is incremental** — security is layered when risk increases

This avoids both false confidence and unnecessary complexity.

---

## 2. Current Trust Model (As Installed)

### Assumptions

- The system runs on a **local machine or private server**
- Access is restricted to:
  - The local user
  - The local network (LAN)
- No public internet exposure is assumed

Under these assumptions, the system is **appropriately secured**.

---

## 3. Exposed Surfaces (Current State)

The following services expose ports to the host:

| Service | Port | Purpose |
|------|------|--------|
| n8n | 5678 | Automation UI + API |
| Open WebUI | 3000 | LLM UI |
| Ollama | 11434 | LLM API |
| Qdrant | 6333 | Vector DB API |
| MinIO | 9000 | Object API |
| MinIO Console | 9001 | Admin UI |

All of these are reachable via `localhost`.

---

## 4. Authentication and Authorization

### n8n

- Basic authentication enabled
- Credentials stored encrypted in PostgreSQL

This is sufficient for local or LAN use.

---

### MinIO

- Username/password authentication
- Admin console protected by credentials

MinIO should not be publicly exposed without additional controls.

---

### Open WebUI

- No authentication by default
- Intended for trusted environments

If exposed publicly, authentication must be added.

---

### Qdrant

- No authentication by default
- Assumes trusted internal network

Qdrant must not be publicly exposed without a proxy or auth layer.

---

### Ollama

- No authentication
- API assumes trusted callers

This is acceptable only in local or internal contexts.

---

## 5. Secrets Management

### What Is Considered a Secret

- Database credentials
- n8n encryption key
- MinIO credentials

### How Secrets Are Managed

- Stored in `.env`
- Loaded at container startup
- Not hardcoded in images

### Required Practices

- `.env` must never be committed to Git
- `.env.example` must not contain real secrets

---

## 6. Network-Level Isolation

### Docker Network

- All services communicate over a private Docker bridge
- Internal DNS is not accessible from the host

This provides a basic isolation layer.

---

### Firewall

- Host firewall controls external access
- Ports should be restricted when not in use

On servers, firewall rules should explicitly allow only required ports.

---

## 7. What Is Intentionally NOT Secured (Yet)

The following are intentionally deferred:

- HTTPS / TLS termination
- OAuth / SSO
- Role-based access control
- API rate limiting
- Audit logging

These are unnecessary for local usage and add complexity prematurely.

---

## 8. Hardening Path (When Needed)

When the system needs stronger security, apply changes in this order.

### Step 1: Reduce Exposed Ports

- Remove direct port exposure
- Route all traffic through a reverse proxy

---

### Step 2: Add HTTPS

- Enable TLS via reverse proxy
- Use trusted certificates

---

### Step 3: Add Authentication Layer

- Protect UIs behind authentication
- Enforce auth before hitting internal services

---

### Step 4: Network Segmentation

- Bind services to localhost only
- Expose only proxy ports

---

### Step 5: Secrets Hardening

- Rotate credentials
- Move secrets to a secrets manager

---

## 9. Security Anti-Patterns

Avoid:

- Publicly exposing databases
- Hardcoding secrets
- Using default passwords
- Running containers as root on the host
- Assuming obscurity equals security

These create silent risk.

---

## 10. Incident Response (Basic)

If a compromise is suspected:

1. Stop the system
2. Rotate all credentials
3. Restore from a known-good backup
4. Review exposed ports
5. Harden before restarting

The architecture supports clean recovery.

---

## 11. Summary

Current state:
- Secure for local and LAN usage
- Explicitly trusted environment
- Minimal attack surface

Future-ready:
- Clear hardening path
- No architectural changes required

Security is intentional, not accidental.

