# 🏥 Hospital Internal Matrix Deployment

## Final Technical Specification v1.0

# 1️⃣ Purpose

Deploy a **single-node, non-federated Matrix communication system** for internal hospital use.

The system must:

* Be Docker-based (rootless preferred)
* Be non-federated
* Not use E2EE by default
* Be accessible only via Cloudflare Tunnel
* Be bound locally (no public ports)
* Support message retention (7 days default)
* Support audit-friendly architecture
* Use a dedicated system user
* Be reproducible via Ansible

---

# 2️⃣ High-Level Architecture

```
Cloudflare Edge (TLS)
        ↓
Cloudflare Tunnel (cloudflared)
        ↓
localhost:8080 (nginx)
        ↓
 ├─ /_matrix  → synapse
 ├─ /admin    → synapse-admin
 └─ /         → cinny (web client)
```

All containers run in an isolated Docker network.

Only nginx is published to `127.0.0.1`.

---

# 3️⃣ Required Ansible Input Variables

These must be required and validated:

```yaml
matrix_fqdn: chat.hospital.example
matrix_instance_name: "Hospital Internal Messaging"

matrix_admin_user: admin
matrix_admin_password: <generated or provided>

postgres_password: <generated or provided>

cloudflare_tunnel_token: <provided securely>

matrix_retention_days: 7
backup_retention_days: 14
```

---

# 4️⃣ System User Requirements

Ansible must:

* Create dedicated user: `matrix`
* No login shell
* Home: `/opt/matrix`
* Own all deployment files
* Run rootless Docker under this user
* All services must operate under this user

Directory layout:

```
/opt/matrix/
    docker-compose.yml
    .env
    homeserver.yaml
    nginx/
    cinny/
    data/
        postgres/
        synapse/
        media_store/
    backups/
```

Ownership: `matrix:matrix`

---

# 5️⃣ Services to Deploy (Docker Compose)

## Required Containers

1. Synapse
2. PostgreSQL
3. Synapse Admin
4. Cinny (custom static build container)
5. nginx
6. cloudflared
7. Backup container (cron-based)

---

# 6️⃣ Synapse Configuration

## Required settings

* Federation disabled
* E2EE not enabled by default
* Single process mode
* PostgreSQL backend
* Media storage local volume
* Retention policy enabled
* No published ports

### Required Synapse config entries:

```yaml
server_name: "{{ matrix_fqdn }}"
public_baseurl: "https://{{ matrix_fqdn }}/"

federation_enabled: false

retention:
  enabled: true
  default_policy:
    min_lifetime: 1d
    max_lifetime: "{{ matrix_retention_days }}d"
```

Encryption defaults must not be auto-enabled for rooms.

---

# 7️⃣ PostgreSQL

* Official postgres image
* Persistent volume
* Strong password
* Not exposed to host
* Only available on Docker network

---

# 8️⃣ Synapse Admin

* Accessible via `/admin`
* Behind nginx reverse proxy
* No direct exposure
* Authenticated via Matrix login

---

# 9️⃣ Cinny (Web Client)

Must:

* Be self-hosted
* Be built from source or prebuilt static
* Configured to use `https://{{ matrix_fqdn }}`
* Customizable:

  * Custom logo
  * Custom landing page text
  * Custom background
  * Display `matrix_instance_name`

Must not expose federation-related UI elements.

Hosted at `/`.

---

# 🔟 nginx Configuration

* Listen internally on port 80
* Published to:

```
127.0.0.1:8080:80
```

Routes:

```
/               → cinny
/_matrix        → synapse
/admin          → synapse-admin
```

No SSL.
No certbot.
Cloudflare handles TLS.

---

# 11️⃣ Cloudflare Tunnel

## Requirements

* Install `cloudflared`
* Configure via provided tunnel token
* Create systemd service
* Route:

```
https://{{ matrix_fqdn }}
→ http://localhost:8080
```

## Automation Policy

Ansible must:

* Install cloudflared
* Configure service
* Inject tunnel token securely

Ansible must NOT:

* Provision Cloudflare account
* Create DNS records via API
* Manage Cloudflare account lifecycle

Tunnel must be pre-created in Cloudflare dashboard.

---

# 12️⃣ Backup Requirements

## 1. PostgreSQL

* Daily `pg_dump`
* Stored in `/opt/matrix/backups/db`
* Retain `backup_retention_days`

## 2. Media Store

* Backup `/data/media_store`
* Daily incremental
* Retain `backup_retention_days`

Optional future enhancement:

* Remote offsite backup target

---

# 13️⃣ Firewall Requirements (Mandatory)

Host firewall must:

## Allow:

* Outbound 443 (Cloudflare)
* Outbound DNS
* Loopback traffic

## Deny inbound:

* TCP 8008
* TCP 8448
* All other unused ports

Default inbound policy: DROP

Firewall managed by Ansible using:

* nftables (preferred) OR
* ufw (simpler)

---

# 14️⃣ Docker Requirements

* Rootless Docker preferred
* Docker Compose plugin required
* No container must publish external ports except nginx
* All services must use internal Docker network

---

# 15️⃣ Security Constraints

* Non-federated
* No public ports
* No exposed database
* No local SSL
* Minimal attack surface
* Dedicated user
* Idempotent Ansible

---

# 16️⃣ Future Extensibility

Must support later addition of:

* LDAP / Active Directory
* OIDC SSO
* Remote backup
* Audit export pipeline

Without major architectural change.

---

# 17️⃣ Non-Goals

* Federation
* Public Matrix deployment
* Multi-node scaling
* Bridges
* E2EE-first design

---

# 18️⃣ Deliverables Required From Implementation LLM

The generated project must include:

* Full Ansible directory structure
* roles/
* inventory example
* templates for:

  * homeserver.yaml
  * nginx.conf
  * docker-compose.yml
* README.md
* Variable validation
* Idempotency
* Example `.env`

---
