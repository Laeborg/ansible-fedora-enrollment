# Fedora AD Enrollment

Automated Active Directory join for Fedora workstations.

## Installation

```bash
# 1. Install Ansible
sudo dnf install ansible

# 2. Edit host_vars/localhost.yaml
# Fill in AD credentials and domain info

# 3. Run playbook
ansible-playbook -i inventory/localhost.yaml playbooks.yaml --ask-become-pass

# 4. Reboot and login with AD user
# Username: username@domain.local
```

## What it does

- Joins AD domain
- Sets DNS to domain controllers
- Offline login (cached credentials)
- Auto-creates home directories
- All AD users get sudo access
- Automatic security updates
- TPM2 disk unlock (if configured)

## Verification

```bash
sudo adcli testjoin          # Check AD join
id username@domain.local     # Test user lookup
sudo systemctl status sssd   # Check SSSD
```

## Troubleshooting

**Login not working:**
```bash
sudo adcli testjoin  # If error: delete computer in AD and re-run playbook
```

**Timeout when logging in without VPN:**
SSSD timeouts are set to 5 seconds - if too long, adjust in `roles/realm/tasks/sssd_config.yaml`
