# Hospital Internal Matrix Deployment (Ansible)

This project deploys a single-node, non-federated Matrix stack for internal hospital use on Ubuntu 24.04. It uses rootless Docker, Cloudflare Tunnel, and nginx exposure on host port `8080` protected by host firewall policy.

## Prerequisites

1. Ansible installed on your control machine.
2. Target host is Ubuntu 24.04 LTS with SSH access.
3. Cloudflare Tunnel already created in the Cloudflare dashboard.
4. Install required Ansible collection:

```bash
ansible-galaxy collection install community.general
```

## Deploy

Terminology:

- Control node: the machine where you run Ansible.
- Managed node: the target server where Matrix will be installed.

Steps (common workflow):

1. On the control node, clone this repository:

```bash
git clone https://github.com/carlosapgomes/simple-matrix.git
cd simple-matrix
```

2. Update `inventory.yml` to point to your managed node (or add its hostname/IP).

Example `inventory.yml`:

```yaml
all:
  hosts:
    matrix1:
      ansible_host: 203.0.113.10
      ansible_user: ubuntu
```

3. Fill out `group_vars/all.yml` with real values (use Vault if desired).

4. Run the playbook from the control node.

Sudo user (recommended):

```bash
ansible-playbook -i inventory.yml playbook.yml -u ubuntu -K
```

The `-K` flag prompts for the sudo password on the managed node.

If you authenticate with SSH password (no SSH keys), use:

```bash
ansible-playbook -i inventory.yml playbook.yml -u ubuntu -k -K
```

The `-k` flag prompts for the SSH login password.

Root account:

```bash
ansible-playbook -i inventory.yml playbook.yml -u root
```

If you encrypted `group_vars/all.yml`, add `--ask-vault-pass`:

```bash
ansible-playbook -i inventory.yml playbook.yml -u ubuntu -K --ask-vault-pass
```

Notes:

- `-K` prompts for the sudo password. If your sudo user has passwordless sudo, you can omit it.
- Replace `ubuntu` with your actual SSH user.

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

Optional web client asset settings:

- `matrix_web_assets_url_path` (default: `/matrix-assets`; static assets URL prefix served by nginx)
- `matrix_web_assets_host_path` (default: `/opt/matrix/web-assets`; source folder mounted read-only into nginx)
- `matrix_web_assets_sync_from_controller` (default: `true`; copy assets from control node if local folder exists)
- `matrix_web_assets_local_path` (default: `{{ playbook_dir }}/assets/matrix-web`; control-node folder copied to `matrix_web_assets_host_path`)

Element Classic deployment:

- `element_classic_image` (default: `docker.io/vectorim/element-web:latest`)
- `element_classic_upstream_port` (default: `80`)
- `element_classic_config_container_path` (default: `/app/config.json`)
- `element_classic_config_json` (default sets `default_server_config` to `matrix_fqdn` and disables custom URLs)

Controller-driven asset sync (optional):

- Put files on the control node under `assets/matrix-web/` in this repository (or override `matrix_web_assets_local_path`).
- On playbook run, if that local folder exists, its contents are copied to `matrix_web_assets_host_path` on the managed host.

Optional Docker rootless tuning:

- `docker_packages_state` (default: `latest`)
- `docker_min_server_version` (default: `28.0.1`)
- `docker_rootlesskit_port_driver` (default: `builtin`)
- `matrix_network_internal` (default: `false`)

Optional Synapse Admin tuning:

- `synapse_admin_build_enabled` (default: `true`; build Synapse Admin from source during `docker compose up`)
- `synapse_admin_build_context` (default: `https://github.com/carlosapgomes/etkecc-synapse-admin.git`)
- `synapse_admin_build_dockerfile` (default: `Dockerfile`)
- `synapse_admin_image` (default: `matrix-synapse-admin:local`; tag to assign to the built image)
- `synapse_admin_upstream_port` (default: `8080`)
- `synapse_admin_restrict_baseurl` (default: `https://{{ matrix_fqdn }}`; published via `/.well-known/matrix/client` as `cc.etke.synapse-admin.restrictBaseUrl`)
- `synapse_admin_well_known_homeserver` (default: `https://{{ matrix_fqdn }}`; published via `/.well-known/matrix/client` as `io.famedly.login.homeserver`)
- `synapse_admin_config_container_path` (default: `/var/public/config.json`)
- `synapse_admin_config_json` (default sets `restrictBaseUrl`; rendered to `/opt/matrix/synapse-admin/config.json` and mounted into container)

Optional registration policy:

- `synapse_enable_registration` (default: `false`)
- `synapse_enable_registration_without_verification` (default: `false`)

Optional reset script deployment:

- `matrix_reset_script_enabled` (default: `true`)
- `matrix_reset_script_path` (default: `/opt/matrix/reset-matrix.sh`)
- `matrix_reset_remove_backups` (default: `true`)

Security best practice: store secrets in Ansible Vault rather than plaintext.

## Using Ansible Vault (Recommended)

This project includes long‑lived secrets (admin password, DB password, tunnel token). Even for a single-use deployment, it is safer to store them encrypted.

Encrypt your inventory variables:

```bash
ansible-vault encrypt group_vars/all.yml
```

Run the playbook:

```bash
ansible-playbook -i inventory.yml playbook.yml --ask-vault-pass
```

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


## What Gets Deployed

- Dedicated `matrix` system user
- Rootless Docker + Docker Compose
- Synapse + PostgreSQL
- Synapse Admin UI under `/admin` (built from `https://github.com/carlosapgomes/etkecc-synapse-admin.git` by default)
- Matrix web client (Element Classic) under `/`
- nginx reverse proxy published on host port `8080`
- Cloudflare Tunnel systemd service
- Host firewall via `ufw`
- Backup cron container (DB + media)
 - Automatic initial admin user creation (idempotent)

## Operational Notes

- All services run under `/opt/matrix` and are owned by the `matrix` user.
- Only nginx is published on host port `8080`; host firewall policy keeps external access blocked.
- Rootless Docker daemon listens on a Unix socket (`/run/user/<uid>/docker.sock`), not a TCP port.
- Cloudflare handles TLS; nginx runs without SSL locally.
- Cloudflared runs as a systemd service and forwards `https://chat.hospital.example` to `http://localhost:8080`.
- Backups are stored under `/opt/matrix/backups`.
 - Rootless Docker runs as a systemd user service for `matrix`.

## Resetting a Dev/Test Deployment

The role deploys a reset helper script on the managed node (default path: `/opt/matrix/reset-matrix.sh`).

Run on the managed node as root (or via sudo):

```bash
sudo /opt/matrix/reset-matrix.sh
```

This script will:

- Stop the stack (`docker compose down`) if compose file exists
- Remove `/opt/matrix/data/postgres`
- Remove `/opt/matrix/data/synapse`
- Remove `/opt/matrix/data/media_store`
- Remove `/opt/matrix/.secrets`
- Optionally remove `/opt/matrix/backups` when `matrix_reset_remove_backups=true`

After running it, execute the Ansible playbook again from the control node to provision a fresh instance.

## Security Best Practices

- Keep `matrix_admin_password` and `postgres_password` in Vault.
- Do not expose ports other than localhost `8080`.
- Restrict SSH access and verify firewall rules before enabling ufw.
- Rotate Cloudflare tunnel tokens if leaked.

## Troubleshooting

- Check systemd status: `systemctl status cloudflared-matrix`
- Check rootless Docker: `sudo -u matrix systemctl --user status docker`
- Check containers: `sudo -u matrix docker compose -f /opt/matrix/docker-compose.yml ps`
- Check nginx published port: `sudo -u matrix XDG_RUNTIME_DIR=/run/user/$(id -u matrix) DOCKER_HOST=unix:///run/user/$(id -u matrix)/docker.sock docker compose -f /opt/matrix/docker-compose.yml -p matrix port nginx 80`
