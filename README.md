# Ansible Fedora Enrollment

Automated enrollment for Fedora 43 laptops with Azure Entra ID authentication.

## Quick Start

1. Install Fedora 43 with an administrator account
2. Install Ansible: `sudo dnf install ansible`
3. Configure [host_vars/localhost.yaml](host_vars/localhost.yaml):

```yaml
# Hostname
hostname_pretty: "Emily's 2nd dev laptop"
hostname_static: "emily-dev-2"

# Azure Entra ID
himmelblau_domain: "example.com"
himmelblau_app_id: "00000000-0000-0000-0000-000000000000"
himmelblau_pam_allow_groups:
  - "11111111-1111-1111-1111-111111111111"  # Group Object ID
himmelblau_sudo_group: "Linux-Admins"        # Group Display Name

# Offline mode
himmelblau_enable_offline: true
himmelblau_cache_users:
  - "user1@example.com"
```

4. Run enrollment:

```bash
ansible-playbook -i inventory/localhost.yaml enroll.yaml --ask-become-pass
```

## What Gets Configured

**Hostname**: Static and pretty hostname
**Packages**: fzf, terraform, k9s, lazygit, VSCode
**Himmelblau**: Azure Entra ID authentication with offline mode and SELinux support
**Sudo**: Azure Entra ID group-based sudo access

## After Enrollment

Users log in with their Azure Entra ID credentials:
- Username: `user@domain` (e.g., `john.doe@example.com`)
- Password: Azure Entra ID password

Useful commands:
```bash
sudo aad-tool status                           # Check status
sudo aad-tool auth-test user@domain            # Test authentication
sudo aad-tool enumerate                        # Cache users for offline
sudo aad-tool offline-breakglass --ttl 24h     # Enable offline mode
```

## Roles

| Role | Purpose |
|------|---------|
| `hostname` | Configure static and pretty hostname |
| `packages` | Install development tools |
| `himmelblau` | Azure Entra ID authentication |
| `sudo` | Grant sudo to Azure Entra ID group |
