# Ansible Fedora Enrollment

Automated enrollment for Fedora 43 laptops with Azure Entra ID or Active Directory authentication.

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

## Active Directory Realm Join (Alternative to Himmelblau)

For traditional Active Directory environments, you can use realm join instead of Himmelblau.

1. Configure [host_vars/localhost.yaml](host_vars/localhost.yaml):

```yaml
# AD Realm Join
ad_domain: "daoas.local"
ad_user: "administrator"
ad_password: "your-ad-password"
ad_computer_ou: "OU=Linux,DC=daoas,DC=local"
ad_allowed_group: "Linux-Users"
```

2. In [enroll.yaml](enroll.yaml), comment out `himmelblau` and uncomment `realm`:

```yaml
roles:
  - hostname
# - himmelblau  # Azure Entra ID
  - realm       # Active Directory
  - sudo
```

3. Run enrollment:

```bash
ansible-playbook -i inventory/localhost.yaml enroll.yaml --ask-become-pass
```

**What it does:**
- Installs realmd, sssd, and required packages
- Joins the Active Directory domain
- Configures SSSD with:
  - Dynamic DNS updates (A and PTR records)
  - SSH public key from AD (altSecurityIdentities attribute)
  - Group-based access control
  - Automatic home directory creation

**After enrollment:**
- Users log in with: `DOMAIN\username` or `username@domain`
- Test with: `realm list` and `id username@domain`
- SSH keys can be stored in AD's altSecurityIdentities attribute

## TPM2 Automatic Disk Decryption (Optional)

Setup automatic LUKS disk decryption using TPM2 to eliminate the need for manual passphrase entry at boot.

1. Configure [host_vars/localhost.yaml](host_vars/localhost.yaml):

```yaml
# Disk encryption
luks_device: "/dev/nvme0n1p3"
luks_passphrase: "your-disk-encryption-passphrase-here"
tpm2_pcr_bank: "sha256"
tpm2_pcr_ids: "7"
```

2. Run the disk encryption setup:

```bash
ansible-playbook -i inventory/localhost.yaml setup-disk-encryption.yaml --ask-become-pass
```

**What it does:**
- Installs Clevis TPM2 packages
- Binds your LUKS encrypted disk to TPM2 (PCR 7 - Secure Boot state)
- Configures GRUB to use TPM2 for automatic decryption
- Regenerates initramfs and GRUB configuration
- Disables NetworkManager-wait-online to speed up boot

**Important notes:**
- Requires TPM 2.0 hardware
- PCR 7 is tied to Secure Boot state - if you disable Secure Boot, automatic decryption will fail
- Your disk encryption passphrase is still required as a fallback
- Make sure to test rebooting after setup to verify it works

## Roles

| Role | Purpose |
|------|---------|
| `hostname` | Configure static and pretty hostname |
| `packages` | Install development tools |
| `himmelblau` | Azure Entra ID authentication |
| `realm` | Active Directory realm join (alternative to himmelblau) |
| `sudo` | Grant sudo to Azure Entra ID group |
| `disk_encryption` | TPM2 automatic disk decryption |
