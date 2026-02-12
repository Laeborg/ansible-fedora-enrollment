# Fedora Active Directory Enrollment

Automated AD domain join for Fedora 43 workstations with offline login support.

## Quick Start

### 1. Install Ansible
```bash
sudo dnf install ansible
```

### 2. Configure
Edit `host_vars/localhost.yaml`:

```yaml
# Hostname (must be FQDN)
hostname_static: "dao-nb-5000.daoas.local"

# Active Directory
ad_domain: "daoas.local"
ad_dns_servers:
  - "192.168.1.10"
ad_user: "administrator"
ad_password: "your-password"
ad_computer_ou: "OU=Linux,DC=daoas,DC=local"
ad_allowed_group: ""  # Empty = all AD users

# VPN (optional - for remote enrollment)
vpn_enabled: true
vpn_server: "vpn.dao.as"
vpn_authgroup: "dao.int"
vpn_username: "your-vpn-user"
vpn_password: "your-vpn-password"
```

### 3. Run
```bash
ansible-playbook -i inventory/localhost.yaml enroll.yaml --ask-become-pass
```

### 4. Login
Username: `username@daoas.local`
Password: Your AD password

## Features

- ✅ AD realm join with SSSD
- ✅ Offline login (cached credentials)
- ✅ Auto VPN during enrollment (if needed)
- ✅ Dynamic DNS updates
- ✅ Home directory auto-creation
- ✅ Sudo access for all AD users

## Verify Setup

```bash
# Check join status
sudo adcli testjoin

# Test user lookup
id username@daoas.local

# Check SSSD
sudo systemctl status sssd
```

## Troubleshooting

**Login fails with "Incorrect password":**
1. Check: `sudo adcli testjoin`
2. If fails: Delete computer account in AD and re-run playbook

**Can't find domain:**
```bash
# Check DNS
cat /etc/systemd/resolved.conf.d/ad-domain.conf
nslookup daoas.local
```

## Notes

- Hostname must be FQDN format
- Use uppercase domain for `kinit`: `kinit user@DOMAIN.LOCAL`
- Login screen uses lowercase: `user@domain.local`
- Offline login works after first successful login
- VPN only used during enrollment, not login
- All AD users automatically get sudo access via "Domain Users" group

## Optional: TPM2 Disk Encryption

Configure in `host_vars/localhost.yaml`:
```yaml
luks_device: "/dev/nvme0n1p3"
luks_passphrase: "your-passphrase"
```

The disk_encryption role is included in `enroll.yaml`. To disable it, comment it out:
```yaml
roles:
  - hostname
  - disk_encryption
  - realm
```

## Files
```
ansible-fedora-enrollment/
├── enroll.yaml                    # Main playbook
├── inventory/localhost.yaml       # Inventory
├── host_vars/localhost.yaml       # Configuration (edit this!)
└── roles/
    ├── hostname/                  # Set system hostname
    ├── realm/                     # AD realm join
    └── disk_encryption/           # TPM2 LUKS unlock
```
