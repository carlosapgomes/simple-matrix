# Hospital Internal Matrix Deployment (Ansible)

This project deploys a single-node, non-federated Matrix stack for internal hospital use on Ubuntu 24.04. It uses rootless Docker, Cloudflare Tunnel, and local-only nginx exposure on `127.0.0.1:8080`.

## Prerequisites

1. Ansible installed on your control machine.
2. Target host is Ubuntu 24.04 LTS with SSH access.
3. Cloudflare Tunnel already created in the Cloudflare dashboard.
4. Install required Ansible collection:

```bash
ansible-galaxy collection install community.general
```

## Configure Inventory Variables

Edit `group_vars/all.yml` and set required values:

- `matrix_fqdn`
- `matrix_instance_name`
- `matrix_admin_user`
- `matrix_admin_password`
- `postgres_password`
- `cloudflare_tunnel_token`
- `matrix_retention_days`
- `backup_retention_days`

Optional Cinny branding:

- `cinny_custom_logo_path`
- `cinny_custom_background_path`
- `cinny_landing_text`

Security best practice: store secrets in Ansible Vault rather than plaintext.

## Cloudflare Tunnel Setup (Token)

This project expects a pre-created Cloudflare Tunnel and uses the tunnel token only.
It does not create or manage Cloudflare resources automatically.

Steps to get the token:

1. Log in to the Cloudflare dashboard.
2. Go to `Zero Trust` → `Access` → `Tunnels`.
3. Create a new tunnel (type: Cloudflared).
4. In the tunnel details, copy the token for your tunnel.
5. Set `cloudflare_tunnel_token` in `group_vars/all.yml` (preferably via Ansible Vault).

Routing requirement:

- Public hostname `https://<matrix_fqdn>` should point to `http://localhost:8080` via the tunnel.

Security best practice:

- Treat the tunnel token as a secret. Rotate it if leaked.

## Deploy

```bash
ansible-playbook -i inventory.yml playbook.yml
```

## Running With Sudo User vs Root

Common sudo user (recommended):

```bash
ansible-playbook -i inventory.yml playbook.yml -u ubuntu -K
```

Root account:

```bash
ansible-playbook -i inventory.yml playbook.yml -u root
```

Notes:

- `-K` prompts for the sudo password. If your sudo user has passwordless sudo, you can omit it.
- Replace `ubuntu` with your actual SSH user.

## What Gets Deployed

- Dedicated `matrix` system user
- Rootless Docker + Docker Compose
- Synapse + PostgreSQL
- Synapse Admin UI under `/admin`
- Cinny web client under `/`
- nginx reverse proxy bound to `127.0.0.1:8080`
- Cloudflare Tunnel systemd service
- Host firewall via `ufw`
- Backup cron container (DB + media)
 - Automatic initial admin user creation (idempotent)

## Operational Notes

- All services run under `/opt/matrix` and are owned by the `matrix` user.
- Only nginx binds to localhost; no public ports are opened.
- Cloudflare handles TLS; nginx runs without SSL locally.
- Cloudflared runs as a systemd service and forwards `https://chat.hospital.example` to `http://localhost:8080`.
- Backups are stored under `/opt/matrix/backups`.
- Cinny branding files (logo/background) are mounted from `/opt/matrix/cinny/custom` and referenced in `config.json`.
 - Rootless Docker runs as a systemd user service for `matrix`.

## Security Best Practices

- Keep `matrix_admin_password` and `postgres_password` in Vault.
- Do not expose ports other than localhost `8080`.
- Restrict SSH access and verify firewall rules before enabling ufw.
- Rotate Cloudflare tunnel tokens if leaked.

## Troubleshooting

- Check systemd status: `systemctl status cloudflared-matrix`
- Check rootless Docker: `sudo -u matrix systemctl --user status docker`
- Check containers: `sudo -u matrix docker compose -f /opt/matrix/docker-compose.yml ps`
