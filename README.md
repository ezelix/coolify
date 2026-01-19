# Coolify Server Setup with Cloudflare Tunnel

This Ansible playbook automates the setup of a Coolify server with Cloudflare Tunnel integration and firewalld configuration.

## Prerequisites

- Ansible installed on your local machine
- SSH access to the target server (or run locally)
- Cloudflare Tunnel token
- Sudo/root access on the target server

## Running the Playbook Locally

### Quick Start: One-Liner with Token

To run the playbook locally with your Cloudflare token included (no prompts):

```bash
ansible-playbook ansible.yaml -e "cloudflare_tunnel_token=YOUR_TOKEN_HERE"
```

**Preventing Bash History Storage:**

To prevent this command (with your sensitive token) from being stored in bash history, use one of these methods:

**Method 1: Start with a space** (requires `HISTCONTROL` to include `ignorespace`):
```bash
export HISTCONTROL=ignorespace
 ansible-playbook ansible.yaml -e "cloudflare_tunnel_token=YOUR_TOKEN_HERE"
```

**Method 2: Temporarily disable history:**
```bash
unset HISTFILE && ansible-playbook ansible.yaml -e "cloudflare_tunnel_token=YOUR_TOKEN_HERE" && export HISTFILE=~/.bash_history
```

**Method 3: Use HISTCONTROL and space prefix:**
```bash
HISTCONTROL=ignorespace ansible-playbook ansible.yaml -e "cloudflare_tunnel_token=YOUR_TOKEN_HERE"
```
(Note: The space before `ansible-playbook` is intentional)

### Simple Command (with prompts)

If you prefer to be prompted for the token:

```bash
ansible-playbook ansible.yaml
```

### Configuration Files

The project includes:
- `ansible.cfg` - Sets default inventory to `hosts.ini`
- `hosts.ini` - Defines `coolify_servers` as localhost with local connection

This allows you to run the playbook with just `ansible-playbook ansible.yaml` without needing to specify inventory or connection options.

### Running Against Remote Servers

To run against remote servers, modify `hosts.ini` or use a different inventory file:

```ini
[coolify_servers]
your-server-ip-or-hostname ansible_user=your_username
```

Then run:

```bash
ansible-playbook ansible.yaml
```

## Playbook Prompts

If you don't provide variables via `-e`, the playbook will prompt you for:

- **Cloudflare Tunnel token**: Your Cloudflare Tunnel authentication token
- **Coolify version**: The version of Coolify to install (defaults to "latest")

You can skip prompts by providing these as extra variables:
- `cloudflare_tunnel_token`: Your Cloudflare Tunnel token
- `coolify_version`: Coolify version (optional, defaults to "latest")

## What the Playbook Does

1. Updates the apt cache
2. Installs prerequisites (curl, wget, docker.io, docker-compose, firewalld)
3. Starts and enables Docker
4. Installs Coolify
5. Creates the Coolify user and adds them to the docker group
6. Installs cloudflared
7. Creates cloudflared configuration directories
8. Deploys cloudflared config and systemd service
9. Configures firewalld:
   - Enables firewalld
   - Restricts SSH access to Cloudflare IP ranges only
   - Configures Docker/Coolify ports
10. Starts and enables Coolify service

## Files

- `ansible.yaml` - Main Ansible playbook
- `ansible.cfg` - Ansible configuration (sets default inventory)
- `hosts.ini` - Default inventory file (defines localhost as coolify_servers)
- `cloud-init` - Cloud-init configuration for initial server setup
- `templates/config.yml.j2` - Cloudflared configuration template
- `templates/cloudflared.service.j2` - Systemd service template for cloudflared
- `fetch-token-examples.sh` - Examples for fetching Cloudflare tokens

## Notes

- The playbook uses `become: true`, so sudo/root access is required
- SSH access will be restricted to Cloudflare IP ranges after the playbook runs
- Make sure you have your Cloudflare Tunnel token ready before running the playbook

