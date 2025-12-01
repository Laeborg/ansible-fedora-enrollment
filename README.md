# Ansible Fedora Enrollment

Modular Ansible setup for enrolling new Fedora 43 laptops.

## Prerequisites

- Fedora 43 installed on the machine
- Administrator account created
- Ansible installed (`sudo dnf install ansible`)

## Structure

```
.
├── enroll.yaml                # Main playbook
├── inventory/
│   └── localhost.yaml        # Inventory for local execution
├── host_vars/
│   └── localhost.yaml        # Configuration variables
└── roles/
    ├── hostname/             # Hostname configuration
    │   └── tasks/
    │       └── main.yaml
    ├── packages/             # Package installation
    │   └── tasks/
    │       ├── main.yaml
    │       └── tools/
    │           ├── fzf.yaml
    │           ├── terraform.yaml
    │           ├── k9s.yaml
    │           ├── lazygit.yaml
    │           └── lazyssh.yaml
    ├── himmelblau/           # Azure Entra ID authentication
    │   ├── tasks/
    │   │   ├── main.yaml
    │   │   ├── install.yaml
    │   │   └── configure.yaml
    │   ├── templates/
    │   │   └── himmelblau.conf.j2
    │   └── handlers/
    │       └── main.yaml
    └── sudo/                 # Sudo access configuration
        └── tasks/
            └── main.yaml
```

## Configuration

Edit [host_vars/localhost.yaml](host_vars/localhost.yaml) and update:

```yaml
# Hostname configuration
hostname_pretty: "Emily's 2nd dev laptop"  # Pretty hostname with spaces and special characters
hostname_static: "emily-dev-2"              # Static hostname (lowercase, numbers and hyphens only)

# Himmelblau configuration
himmelblau_domain: "example.com"                              # Your Azure Entra ID domain
himmelblau_app_id: "00000000-0000-0000-0000-000000000000"     # Azure App Registration ID
himmelblau_pam_allow_groups:                                   # Azure AD group Object IDs allowed to authenticate
  - "11111111-1111-1111-1111-111111111111"
  - "22222222-2222-2222-2222-222222222222"

# Sudo configuration
himmelblau_sudo_group: "Linux-Admins"  # Azure AD group name (not Object ID)
```

## Usage

Run the enrollment playbook:

```bash
ansible-playbook -i inventory/localhost.yaml enroll.yaml --ask-become-pass
```

You will be prompted to enter the administrator password for sudo access.

## Modules

### Hostname
Configures both static and pretty hostname on the machine.

- Static hostname: Used for network and system identification
- Pretty hostname: User-friendly name with spaces and special characters

### Packages
Installs development and system management tools:

- **fzf**: Fuzzy finder for command line
- **terraform**: Infrastructure as Code tool
- **k9s**: Kubernetes CLI management tool (via Copr)
- **lazygit**: Terminal UI for git commands (via Copr)
- **lazyssh**: Terminal UI for SSH connections (via Go install)

Note: lazyssh is installed to `~/go/bin/lazyssh` and requires Go to be installed on the system.

### Himmelblau
Configures Azure Entra ID (formerly Azure AD) authentication for Linux:

- Installs Himmelblau core packages (himmelblau, pam-himmelblau, nss-himmelblau)
- Installs optional packages (himmelblau-sso, o365, himmelblau-selinux)
- Configures PAM and NSS integration
- Sets up domain and Azure App Registration
- Configures group-based access control

**Features:**
- Azure Entra ID authentication with MFA support
- Browser Single Sign-On (SSO)
- Office 365 web apps integration
- SELinux module for enhanced security

**Configuration:**
- `himmelblau_domain`: Your Azure Entra ID domain
- `himmelblau_app_id`: Azure App Registration ID
- `himmelblau_pam_allow_groups`: List of Azure AD group Object IDs allowed to authenticate

**After Enrollment:**
Once the playbook completes, Himmelblau is ready to use. Users can log in with:
- Username: `username@domain` (e.g., `john.doe@contoso.com`)
- Password: Their Azure AD password

**Useful Commands:**
```bash
sudo aad-tool status              # Check daemon status
sudo aad-tool auth-test user@domain  # Test user authentication
sudo aad-tool enumerate           # Enumerate and cache users/groups
sudo aad-tool cache-clear         # Clear authentication cache
```

### Sudo
Configures sudo access for Azure AD users:

- Creates sudoers configuration for Azure AD groups or specific users
- Validates sudoers syntax before applying
- Supports both group-based and user-based access control

**Configuration:**
- `himmelblau_sudo_group`: Azure AD group name (display name, not Object ID) whose members get sudo access
- `himmelblau_sudo_users`: Optional list of specific usernames who get sudo access

**Example:**
```yaml
# Option 1: Grant sudo to all members of an Azure AD group
himmelblau_sudo_group: "Linux-Admins"

# Option 2: Grant sudo to specific users
himmelblau_sudo_users:
  - "john.doe"
  - "jane.smith"
```

**Note:** When users log in via Himmelblau, their username format is `username@domain`. The sudo configuration handles this automatically.

## Adding More Modules

To add new configuration modules:

1. Create a new directory under `roles/` (e.g., `roles/users/`)
2. Create `tasks/main.yaml` in the new role
3. Add the role to `enroll.yaml` under `roles:`
4. Add any variables to `host_vars/localhost.yaml`
