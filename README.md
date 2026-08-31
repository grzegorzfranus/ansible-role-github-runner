# Ansible Role: GitHub Runner

|Source|Version|CI|License|
|------|-------|--|-------|
|[![Source Code](https://img.shields.io/badge/source-github-blue.svg)](https://github.com/grzegorzfranus/ansible-role-github-runner)|[![Version](https://img.shields.io/github/v/release/grzegorzfranus/ansible-role-github-runner)](https://github.com/grzegorzfranus/ansible-role-github-runner/releases)|[![CI](https://github.com/grzegorzfranus/ansible-role-github-runner/actions/workflows/ci.yml/badge.svg)](https://github.com/grzegorzfranus/ansible-role-github-runner/actions/workflows/ci.yml)|[![Repository License](https://img.shields.io/badge/license-apache2.0-brightgreen.svg)](LICENSE)|

This Ansible role installs and configures GitHub Actions Runner for repository and organization-level automation. It provides a comprehensive solution for self-hosted runner deployment with dedicated system user management, latest version installation, and robust systemd service configuration.

## ✨ Features

- **Multi-Instance Support**: Run multiple isolated runners concurrently on a single server
- **Automatic Latest Version**: Downloads and installs the latest GitHub Runner or specific versions
- **Dedicated User Management**: Creates system users with optional sudo privileges for secure execution
- **Dual Mode Support**: Configures runners for both repository-level and organization-level automation
- **Security Hardening**: Implements systemd security features and file permission controls
- **Service Management**: Complete systemd integration with restart policies and resource limits
- **Comprehensive Logging**: Detailed logging with journal integration and debug capabilities
- **Network Configuration**: Proxy support and SSL verification controls
- **Health Verification**: Built-in verification and connectivity testing
- **Package Dependencies**: Automatic installation of required and optional dependencies
- **Testing Framework**: Full verification suite for deployment validation

## 🎯 Architecture

The role provides flexible GitHub Actions Runner deployment supporting both:

- **Repository Mode**: Runners dedicated to specific GitHub repositories
- **Organization Mode**: Shared runners available across organization repositories
- **Multi-Instance Mode**: Multiple isolated runners running concurrently on the same server
- **Enterprise Support**: Compatible with GitHub Enterprise Server instances

**Single Runner Architecture:**
```
GitHub Repository/Organization ←→ Self-Hosted Runner ←→ Local Resources
         (source)                    (this host)         (execution)
```

**Multi-Instance Architecture:**
```
GitHub Organization ←→ Runner-01 (github-runner-01) ←→ Local Resources
                    ←→ Runner-02 (github-runner-02) ←→ Local Resources
                    ←→ Runner-03 (github-runner-03) ←→ Local Resources
```

## 📋 Requirements

- **Ansible**: 2.20 or higher
- **Network**: Internet access for GitHub connectivity and runner downloads
- **Privileges**: sudo/root access on target hosts
- **Collections**: `community.crypto` (for SSH key generation feature)

### Supported operating systems
List of officially supported operating systems:
| OS Family | Version | Status |
|-----------|---------|---------|
| Ubuntu | 24.04 (Noble) | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) |
| Ubuntu | 22.04 (Jammy) | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) |
| Debian | 12 (Bookworm) | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) |

### Ansible version

Ansible >= 2.20

### Python version

Python >= 3.9

### Setup module
The role uses facts gathered by Ansible on the remote host. If you disable the Setup module in your playbook, the role will not work properly.

### Root access
This role requires root access for some tasks. Make sure that you are using a user with root privileges.

## 🚀 Quick Start

### 1. Basic Repository Runner Setup

```yaml
---
- name: Configure GitHub Repository Runner
  hosts: all
  become: true
  roles:
    - role: grzegorzfranus.github_runner
      vars:
        github_runner_repository_url: "owner/repository"
        github_runner_access_token: "ghp_your_personal_access_token"
        github_runner_name: "{{ ansible_facts['hostname'] }}-repo-runner"
        github_runner_labels: "ubuntu-24.04,self-hosted,production"
```

### 2. Organization Runner Configuration  

```yaml
---
- name: Configure GitHub Organization Runner  
  hosts: runners
  become: true
  roles:
    - role: grzegorzfranus.github_runner
      vars:
        github_runner_organization: "my-organization"
        github_runner_access_token: "ghp_your_personal_access_token"
        github_runner_name: "{{ ansible_facts['hostname'] }}-org-runner"
        github_runner_group: "production"
        github_runner_user_sudo_enabled: true
```

### 3. Runner with SSH Key Generation (for deployments)

```yaml
---
- name: Configure GitHub Runner with SSH Key for Deployment
  hosts: deployment-runners
  become: true
  roles:
    - role: grzegorzfranus.github_runner
      vars:
        github_runner_organization: "my-organization"
        github_runner_access_token: "ghp_your_personal_access_token"
        github_runner_name: "{{ ansible_facts['hostname'] }}-deploy-runner"
        github_runner_labels: "deployment,production,{{ ansible_facts['distribution'] | lower }}"
        
        # Enable SSH key generation for deployment
        github_runner_user_generate_ssh_key: true
        github_runner_user_ssh_key_type: "ed25519"
        github_runner_user_ssh_key_comment: "github-runner@{{ ansible_facts['hostname'] }}-deploy"
        
        # Enable sudo for deployment tasks
        github_runner_user_sudo_enabled: true
        github_runner_user_sudo_nopasswd: true
```

### 4. Multi-Instance Runner Setup

```yaml
---
- name: Configure Multiple Runners on Single Server
  hosts: powerful-runners
  become: true
  roles:
    - role: grzegorzfranus.github_runner
      vars:
        github_runner_organization: "my-organization"
        github_runner_access_token: "ghp_your_personal_access_token"
        
        # Define multiple isolated runner instances
        github_runner_instances:
          - id: "01"
            name: "{{ ansible_facts['hostname'] }}-prod-01"
            labels: "linux,production,docker"
          - id: "02"
            name: "{{ ansible_facts['hostname'] }}-prod-02"
            labels: "linux,production,node"
          - id: "03"
            name: "{{ ansible_facts['hostname'] }}-prod-03"
            labels: "linux,production,python"
```

### 5. Runner with Professional RSyslog Logging

```yaml
---
- name: Configure GitHub Runner with Professional RSyslog Logging
  hosts: production-runners
  become: true
  roles:
    - role: grzegorzfranus.github_runner
      vars:
        github_runner_organization: "my-organization"
        github_runner_access_token: "ghp_your_personal_access_token"
        github_runner_name: "{{ ansible_facts['hostname'] }}-production-runner"
        github_runner_labels: "production,{{ ansible_facts['distribution'] | lower }},self-hosted"
        
        # Enable professional RSyslog-based logging (recommended)
        github_runner_logging_enabled: true
        github_runner_rsyslog_enabled: true
        github_runner_log_dir: "/var/log/github-runner"
        github_runner_syslog_identifier: "github-runner"
        
        # Custom logrotate settings for production
        github_runner_logrotate_options:
          enabled: true
          frequency: "daily"
          rotate_count: 30
          size: "500M"
          compress: true
```

### 5. Install required collections (for SSH key generation)

```bash
ansible-galaxy collection install community.crypto
```

### 6. Run the playbook

```bash
ansible-playbook -i inventory github-runner-setup.yml
```

### 7. Uninstall the runner

```yaml
---
- name: Uninstall GitHub Runner
  hosts: runners
  become: true
  roles:
    - role: grzegorzfranus.github_runner
      vars:
        github_runner_state: "absent"
        github_runner_organization: "my-org"
        github_runner_access_token: "{{ vault_github_pat }}"
```

Removal contacts the GitHub API to obtain a removal token and unregisters the runner before anything is deleted locally, so the access token stays required during teardown. If no token can be obtained, or `config.sh remove` fails, the play stops rather than deleting a runner that would survive in the organization as a permanently offline entry. Set `github_runner_remove_require_unregistration` to `false` to tear the host down anyway.

## ⚙️ Configuration

### Default Configuration

The role comes with production-ready defaults:

```yaml
# Runner installation
github_runner_version: "latest"
github_runner_enabled: true

# User management
github_runner_create_user: true
github_runner_user: "github-runner"
github_runner_user_sudo_enabled: false

# Service configuration
github_runner_service_enabled: true
github_runner_service_state: "started"
```

### Advanced Configuration

Customize for specific requirements:

```yaml
---
- name: Advanced GitHub Runner Configuration
  hosts: all
  become: true
  vars:
    # Installation Configuration
    github_runner_version: "latest"
    github_runner_install_dir: "/opt/actions-runner"
    github_runner_force_reinstall: false
    
    # User Configuration with Sudo
    github_runner_create_user: true
    github_runner_user_sudo_enabled: true
    github_runner_user_sudo_nopasswd: true
    
    # Service Configuration
    github_runner_service_restart_policy: "always"
    github_runner_service_limits:
      memory_max: "4G"
      cpu_quota: "300%"
      tasks_max: 8192
    
    # Security Configuration
    github_runner_verify_ssl: true
    github_runner_disable_telemetry: true
      
      # Download integrity (optional)
      github_runner_verify_checksum: true
      github_runner_checksums_file: "sha256sums.txt"
    
    # Custom Environment
    github_runner_custom_env:
      NODE_VERSION: "18"
      DOCKER_BUILDKIT: "1"
  roles:
    - role: grzegorzfranus.github_runner
```

## 📊 Variables

### General Options

| Variable | Description | Default |
|----------|-------------|---------|
| `github_runner_role_action` | Define which parts of the role to execute (Options: 'all', 'prerequisites', 'user', 'install', 'logging', 'configure', 'service') | `"all"` |
| `github_runner_enabled` | Enable/disable GitHub Runner installation | `true` |
| `github_runner_debug` | Enable debug mode for troubleshooting | `false` |
| `github_runner_test_mode` | Enable test mode (used by Molecule tests to skip real GitHub API calls) | `false` |

### Installation Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `github_runner_version` | GitHub Runner version to install (use 'latest' for latest version) | `"latest"` |
| `github_runner_architecture` | Runner architecture (auto-detected) | Auto-detected |
| `github_runner_os` | Runner operating system | `"linux"` |
| `github_runner_install_dir` | Installation directory | `"/opt/actions-runner"` |
| `github_runner_temp_dir` | Temporary directory for downloads | `"/tmp/github-runner-install"` |
| `github_runner_force_reinstall` | Force reinstallation even if already installed | `false` |
| `github_runner_cleanup` | Clean up temp files after installation | `true` |
| `github_runner_download_timeout` | GitHub API timeout for downloads (seconds) | `300` |

### GitHub Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `github_runner_server_url` | GitHub instance URL | `"https://github.com"` |
| `github_runner_repository_url` | GitHub repository URL (leave empty for organization runner) | `""` |
| `github_runner_organization` | GitHub organization name (leave empty for repository runner) | `""` |
| `github_runner_access_token` | GitHub personal access token | `""` |
| `github_runner_repo_url_direct` | Direct repository URL (alternative to repository_url) | `""` |
| `github_runner_token_direct` | Registration token (alternative to access_token) | `""` |
| `github_runner_name` | Runner name | `"{{ ansible_facts['hostname'] }}"` |
| `github_runner_group` | Runner group name | `""` |
| `github_runner_labels` | Runner labels (comma-separated) | Auto-generated |
| `github_runner_work_dir` | Runner work directory | `"{{ github_runner_install_dir }}/_work"` |
| `github_runner_replace` | Replace existing runner with same name | `false` |
| `github_runner_ephemeral` | Runner as ephemeral (removed after job completion) | `false` |
| `github_runner_disable_update` | Disable automatic runner updates | `false` |
| `github_runner_state` | Desired state of runner: `present` or `absent` | `present` |
| `github_runner_remove_require_unregistration` | Fail the teardown instead of deleting a runner that could not be unregistered from GitHub | `true` |

> When `github_runner_state` is `absent`, the role acquires a removal token from the GitHub API and unregisters the runner before deleting anything locally. Set `github_runner_remove_require_unregistration` to `false` only for a teardown that must proceed without GitHub connectivity; the runner then stays in the organization as an offline entry that has to be deleted by hand.

### Multi-Instance Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `github_runner_instances` | List of runner instances (empty list equals single-runner legacy mode) | `[]` |
| `github_runner_instances[].id` | 2-digit instance identifier (e.g. "01") | required if using instances |
| `github_runner_instances[].name` | Custom runner name | `{{ hostname }}-{{ id }}` |
| `github_runner_instances[].labels` | Instance-specific labels | `github_runner_labels` |
| `github_runner_instances[].group` | Runner group in GitHub | `github_runner_group` |
| `github_runner_instances[].uid` | Optional explicit user ID for this instance | `null` (OS auto-assign) |
| `github_runner_instances[].gid` | Optional explicit group ID for this instance | `null` (OS auto-assign) |
| `github_runner_instances[].ephemeral` | Ephemeral mode per instance | `github_runner_ephemeral` |
| `github_runner_instances[].service_limits` | Per-instance resource limits | `github_runner_service_limits` |

### User Management

| Variable | Description | Default |
|----------|-------------|---------|
| `github_runner_create_user` | Create a dedicated user for GitHub Runner | `true` |
| `github_runner_user` | User name | `"github-runner"` |
| `github_runner_user_group` | User group | `"github-runner"` |
| `github_runner_user_uid` | User UID | `null` (autoselect) |
| `github_runner_group_gid` | Group GID | `null` (autoselect) |
| `github_runner_tool_cache` | Optional tool cache directory (RUNNER_TOOL_CACHE) | `""` |
| `github_runner_user_home` | User home directory | `"/home/{{ github_runner_user }}"` |
| `github_runner_user_shell` | User shell | `"/bin/bash"` |
| `github_runner_user_system` | Create as system user | `true` |
| `github_runner_user_sudo_enabled` | Enable sudo privileges | `false` |
| `github_runner_user_sudo_nopasswd` | Sudo without password | `true` |
| `github_runner_user_sudo_commands` | Allowed sudo commands | `"ALL"` |
| `github_runner_user_sudo_logfile_enabled` | Write a per-user sudo logfile (keep `false` on sudo-rs / Ubuntu 25.10+) | `false` |
| `github_runner_user_ssh_key` | Public SSH key content | `""` |
| `github_runner_user_ssh_key_file` | Path to SSH key file | `""` |
| `github_runner_user_generate_ssh_key` | Generate new SSH key pair for deployment | `false` |
| `github_runner_user_ssh_key_type` | SSH key type (rsa, ed25519, ecdsa) | `"ed25519"` |
| `github_runner_user_ssh_key_bits` | Key bits (for RSA keys) | `4096` |
| `github_runner_user_ssh_key_comment` | SSH key comment | Auto-generated |


### Service Management

| Variable | Description | Default |
|----------|-------------|---------|
| `github_runner_service_enabled` | Enable service on boot | `true` |
| `github_runner_service_state` | Service state | `"started"` |
| `github_runner_service_name` | Systemd service name | Auto-generated |
| `github_runner_service_restart_policy` | Service restart policy | `"always"` |
| `github_runner_service_restart_delay` | Restart delay in seconds | `15` |
| `github_runner_service_environment` | Service environment variables | See defaults |
| `github_runner_service_log_level` | Service logging level (error, warn, info, debug) | `"info"` |
| `github_runner_service_limits` | Resource limits dictionary | See defaults |
| `github_runner_max_jobs` | Maximum number of concurrent jobs | `1` |

### Logging Configuration

#### Basic Logging Options

| Variable | Description | Default |
|----------|-------------|---------|
| `github_runner_logging_enabled` | Enable dedicated log files instead of systemd journal | `false` |
| `github_runner_log_dir` | Log directory path | `"/var/log/github-runner"` |
| `github_runner_log_file` | Main log file name | `"runner.log"` |
| `github_runner_log_error_file` | Error log file name | `"runner-error.log"` |
| `github_runner_log_dir_mode` | Log directory permissions | `"0755"` |
| `github_runner_log_file_mode` | Log file permissions | `"0644"` |

#### RSyslog Integration (Professional Approach)

| Variable | Description | Default |
|----------|-------------|---------|
| `github_runner_rsyslog_enabled` | Enable RSyslog integration for professional logging | `true` |
| `github_runner_syslog_identifier` | Syslog program identifier for filtering | `"github-runner"` |
| `github_runner_rsyslog_config_file` | RSyslog configuration file name | `"49-github-runner.conf"` |
| `github_runner_log_user` | Log file owner for RSyslog approach | `"syslog"` |
| `github_runner_log_group` | Log file group for RSyslog approach | `"adm"` |

#### Logrotate Options (`github_runner_logrotate_options`)

| Option | Description | Default |
|--------|-------------|---------|
| `enabled` | Enable logrotate configuration | `true` |
| `frequency` | Rotation frequency (daily, weekly, monthly) | `"daily"` |
| `rotate_count` | Number of rotated files to keep | `30` |
| `size` | Maximum file size before rotation | `"100M"` |
| `compress` | Compress rotated logs | `true` |
| `delaycompress` | Delay compression by one cycle | `true` |
| `missingok` | Don't error if log file is missing | `true` |
| `notifempty` | Only rotate if log file is not empty | `true` |
| `copytruncate` | Use copytruncate instead of moving files | `false` |
| `dateext` | Use date extension for rotated files | `false` |
| `dateformat` | Date format for rotated files | `".%Y-%m-%d"` |
| `olddir` | Directory to move old log files to | `""` |
| `create_mode` | File permissions for newly created logs | `"0640"` when RSyslog is enabled, else `github_runner_log_file_mode` |
| `create_owner` | Owner for newly created log files | `github_runner_log_user` when RSyslog is enabled, else `github_runner_user` |
| `create_group` | Group for newly created log files | `github_runner_log_group` when RSyslog is enabled, else `github_runner_user_group` |

> The `create_*` values follow `github_runner_rsyslog_enabled` because they must match the process that appends to the log files. With RSyslog integration active, `rsyslogd` writes them and drops privileges to the syslog user on Debian and Ubuntu, so a rotated file created for the runner service account is unwritable to it and logging stops at the first rotation.

### Security Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `github_runner_install_dir_mode` | Installation directory permissions | `"0755"` |
| `github_runner_verify_ssl` | Enable SSL certificate verification | `true` |
| `github_runner_verify_checksum` | Verify download against checksums | `false` |
| `github_runner_checksums_file` | Checksum file name in releases | `"sha256sums.txt"` |
| `github_runner_disable_telemetry` | Disable telemetry collection | `true` |
| `github_runner_disable_analytics` | Disable analytics collection | `true` |
| `github_runner_proxy_url` | HTTP proxy URL if needed | `""` |
| `github_runner_no_proxy` | Comma-separated list of hosts to bypass proxy | `""` |
| `github_runner_service_no_new_privileges` | Prevent processes from gaining new privileges (blocks `sudo` if `true`) | `false` |
| `github_runner_service_private_tmp` | Use private `/tmp` namespace for the runner service | `false` |
| `github_runner_service_protect_kernel_tunables` | Restrict access to kernel tunables (implicitly forces `NoNewPrivileges=yes` if `true`) | `false` |
| `github_runner_service_protect_kernel_modules` | Restrict access to kernel modules (implicitly forces `NoNewPrivileges=yes` if `true`) | `false` |
| `github_runner_service_protect_control_groups` | Restrict access to control groups (implicitly forces `NoNewPrivileges=yes` if `true`) | `false` |
| `github_runner_service_restrict_realtime` | Restrict realtime scheduling (implicitly forces `NoNewPrivileges=yes` if `true`) | `false` |

### Advanced Options

| Variable | Description | Default |
|----------|-------------|---------|
| `github_runner_custom_env` | Custom environment variables dictionary | `{}` |
| `github_runner_pre_job_hook` | Script to run before each job | `""` |
| `github_runner_post_job_hook` | Script to run after each job | `""` |
| `github_runner_dependencies` | Required package dependencies | See defaults |
| `github_runner_min_disk_space_gb` | Minimum required disk space in GB | `10` |
| `github_runner_min_memory_mb` | Minimum required memory in MB | `512` |

## 📌 Role Properties

| Property | Value | Description |
|----------|-------|-------------|
| **Idempotent** | ✅ Yes | Running the role multiple times with the same parameters produces the same result. Configuration steps and service status updates are skipped if no changes are detected. |
| **Atomic** | ❌ No | The role can be partially applied. A failure mid-execution may leave the system in an intermediate state (e.g., packages installed but runner not registered). |
| **Check Mode** | ✅ Supported | Most tasks work in check mode. Mutating commands (such as runner registration/unregistration) are skipped during check runs. |
| **Diff Mode** | ✅ Supported | Template tasks support diff mode for configuration changes preview. |

## 📤 Role Output

This role does not set any public output facts. All internal facts use the `__github_runner_` double-underscore prefix and are not part of the public interface.

To inspect diagnostic data at runtime, run the playbook with `-v` or `-vv` verbosity flags — the role provides detailed debug output at `verbosity: 1`.

## 🔍 Verification

After deployment, verify the runner installation and connectivity:

### Check Runner Status

```bash
# Check service status (Single instance)
sudo systemctl status actions.runner.*

# Check service status (Multi-instance example)
sudo systemctl status actions.runner.*.runner-prod-01
sudo systemctl status actions.runner.*.runner-prod-02

# View runner processes
ps aux | grep Runner

# Check runner configuration
sudo -u github-runner cat /opt/actions-runner/.runner
```

### Verify GitHub Connectivity

```bash
# Test GitHub API connectivity
curl -H "Authorization: token YOUR_TOKEN" \
     https://api.github.com/user

# Check runner registration
curl -H "Authorization: token YOUR_TOKEN" \
     https://api.github.com/repos/OWNER/REPO/actions/runners
```

### Check Logs

```bash
# View service logs (systemd journal - always available)
sudo journalctl -u actions.runner.* -f

# View dedicated log files (when logging_enabled: true)
sudo tail -f /var/log/github-runner/runner.log
sudo tail -f /var/log/github-runner/runner-error.log

# Check RSyslog status and configuration (when rsyslog_enabled: true)
sudo systemctl status rsyslog
sudo cat /etc/rsyslog.d/49-github-runner.conf

# Check runner diagnostics
ls -la /opt/actions-runner/_diag/

# Monitor runner activity
tail -f /opt/actions-runner/_diag/*.log

# Check logrotate status
sudo logrotate -d /etc/logrotate.d/github-runner

# Verify RSyslog is processing GitHub Runner logs
sudo grep "github-runner" /var/log/syslog | tail -5
```

## 🛡️ Security Features

- **Dedicated System User**: Isolated execution environment with minimal privileges
- **Systemd Security**: Comprehensive security restrictions and sandboxing
- **File Permissions**: Secure file ownership and access controls
- **Resource Limits**: Memory, CPU, and process limits to prevent resource exhaustion
- **Network Security**: Proxy support and SSL verification controls
- **Optional Sudo**: Configurable privilege escalation with logging

#### Secrets handling
- Store `github_runner_access_token` in Ansible Vault. Example:
  - Create a vaulted variable file: `ansible-vault create group_vars/all/github_tokens.yml`
  - Add: `github_runner_access_token: "ghp_xxx..."`
  - Use with `--ask-vault-pass` or a vault password file.
  - The role validates token format and never logs token values.

### Enhanced Security Configuration

```yaml
# Minimal privileges (recommended for production)
github_runner_user_sudo_enabled: false

# Resource constraints
github_runner_service_limits:
  memory_max: "2G"
  cpu_quota: "200%"
  tasks_max: 4096

# Network security
github_runner_verify_ssl: true
github_runner_disable_telemetry: true
```

## 🔒 Security considerations

- Use Ansible Vault for `github_runner_access_token` and any secrets.
- The `config.sh` runner registration task uses `no_log: true` to prevent token leakage in logs.
- Dedicated user execution model restricts runner processes' system access.

## 🧪 Check mode behavior

- Most informational and verification tasks run in check mode.
- Mutating commands (such as downloading/installing the runner, or registering it to GitHub) are skipped in check mode.

## 🏷️ Tags usage

- Use `--tags` to run selective parts: `prerequisites`, `user`, `install`, `logging`, `configure`, `service`, `verify`.

## 🌐 Network resilience

- Network operations (like downloading packages, registering runner, querying GitHub API) use timeouts and retries. Consider setting HTTP/HTTPS proxy environment variables (`github_runner_proxy_url`, `github_runner_no_proxy`) if needed.

## 🔧 Troubleshooting

### Service won't start

```bash
# Check service status and logs
sudo systemctl status actions.runner.*
sudo journalctl -u actions.runner.* --no-pager

# Verify configuration files
sudo -u github-runner /opt/actions-runner/config.sh --check

# Test runner manually
sudo -u github-runner /opt/actions-runner/run.sh
```

### Runner registration issues

```bash
# Verify GitHub connectivity
curl -I https://github.com

# Check access token permissions
curl -H "Authorization: token YOUR_TOKEN" \
     https://api.github.com/user

# Validate repository/organization access
curl -H "Authorization: token YOUR_TOKEN" \
     https://api.github.com/repos/OWNER/REPO
```

### Performance problems

```bash
# Check resource usage
sudo systemctl show actions.runner.* --property=MemoryCurrent,CPUUsageNSec

# Monitor system resources
htop -u github-runner

# Adjust service limits
sudo systemctl edit actions.runner.*
```

### Permission issues

```bash
# Check file ownership
ls -la /opt/actions-runner/

# Verify user groups
groups github-runner

# Check sudo configuration
sudo -l -U github-runner
```

## 📁 File Structure

```
ansible-role-github-runner/
├── .github/
│   ├── ISSUE_TEMPLATE/                # Issue templates for bug, feature, task
│   │   ├── bug_report.yml
│   │   ├── config.yml
│   │   ├── feature_request.yml
│   │   └── task.yml
│   ├── PULL_REQUEST_TEMPLATE/         # Pull request description template
│   │   └── pull_request_template.md
│   ├── workflows/
│   │   ├── ci.yml                     # CI pipeline (reusable ansible-ci.yml)
│   │   └── release.yml                # Release Please + Galaxy publish
│   └── dependabot.yml                 # Dependabot configuration for GitHub Actions
├── defaults/
│   └── main.yml             # Default configuration variables
├── handlers/
│   └── main.yml             # Service restart and reload handlers
├── meta/
│   ├── argument_specs.yml   # Role argument specification, validated before execution
│   └── main.yml             # Role metadata and Galaxy information
├── tasks/
│   ├── main.yml             # Main task orchestration
│   ├── instance.yml         # Per-instance orchestration for multi-runner support
│   ├── assert.yml           # Variable validation and compatibility checks
│   ├── prerequisites.yml    # System requirements and dependency installation
│   ├── user.yml             # User and group management
│   ├── install.yml          # GitHub Runner installation
│   ├── logging.yml          # Dedicated logging configuration and logrotate setup
│   ├── configure.yml        # Runner registration and configuration
│   ├── service.yml          # Systemd service management
│   ├── verify.yml           # Installation verification and health checks
│   └── remove.yml           # Runner uninstallation (state: absent)
├── templates/
│   ├── github-runner.service.j2    # Systemd service configuration
│   ├── service-override.conf.j2    # Service resource limits
│   ├── runner-environment.j2       # Environment variables
│   ├── sudoers.j2                  # Sudo configuration
│   ├── rsyslog/
│   │   └── github-runner.conf.j2   # RSyslog configuration for professional logging
│   └── logrotate/
│       └── github-runner.j2        # Logrotate configuration for log management
├── vars/
│   └── main.yml             # Internal role variables and constants
├── CHANGELOG.md             # Version history and changes
├── LICENSE                  # Apache-2.0 license
└── README.md               # This documentation file
```

## 🏷️ Tags

- `always` - Tasks that always run (variable loading and validation)
- `setup` - Setup tasks including prerequisites, installation, and configuration
- `init` - Initial setup tasks
- `validate` - Variable validation tasks
- `check` - Validation and verification tasks
- `prerequisites` - System requirements verification
- `user` - User management tasks
- `install` - Package installation tasks
- `logging` - Dedicated logging configuration and logrotate setup
- `configure` - Runner configuration tasks
- `service` - Service management tasks
- `verify` - Verification and health check tasks

## Example Playbook

```yaml
---
- name: Configure GitHub Actions Runners
  hosts: all
  roles:
    - role: grzegorzfranus.github_runner
      vars:
        # Basic Configuration
        github_runner_enabled: true
        github_runner_version: "latest"
        
        # GitHub Configuration
        github_runner_repository_url: "myorg/myrepo"
        github_runner_access_token: "{{ github_token }}"
        github_runner_name: "{{ ansible_facts['hostname'] }}-runner"
        github_runner_labels: "ubuntu-22.04,self-hosted,production"
        
        # User Configuration
        github_runner_create_user: true
        github_runner_user_sudo_enabled: true
        github_runner_user_sudo_nopasswd: true
        
        # Service Configuration
        github_runner_service_enabled: true
        github_runner_service_restart_policy: "always"
        
        # Resource Limits
        github_runner_service_limits:
          memory_max: "4G"
          cpu_quota: "200%"
          tasks_max: 8192
        
        # Security Configuration
        github_runner_verify_ssl: true
        github_runner_disable_telemetry: true
        
        # Custom Environment
        github_runner_custom_env:
          NODE_VERSION: "18"
          PYTHON_VERSION: "3.11"
          DOCKER_BUILDKIT: "1"
        
        # Dedicated Logging Configuration
        github_runner_logging_enabled: true
        github_runner_log_dir: "/var/log/github-runner"
        github_runner_logrotate_options:
          frequency: "daily"
          rotate_count: 30

# Example of using the role with different settings for different host groups
- name: Configure Repository Runners
  hosts: repo_runners
  roles:
    - role: grzegorzfranus.github_runner
      vars:
        github_runner_repository_url: "myorg/myrepo"
        github_runner_access_token: "{{ github_repo_token }}"
        github_runner_labels: "repository,ubuntu-22.04,self-hosted"
        github_runner_user_sudo_enabled: false

- name: Configure Organization Runners
  hosts: org_runners
  roles:
    - role: grzegorzfranus.github_runner
      vars:
        github_runner_organization: "myorg"
        github_runner_access_token: "{{ github_org_token }}"
        github_runner_labels: "organization,ubuntu-22.04,self-hosted,shared"
        github_runner_group: "production"
        github_runner_user_sudo_enabled: true
        github_runner_ephemeral: true
```

## 🧪 Testing

This role includes comprehensive testing infrastructure using Molecule and GitHub Actions CI/CD.

### Molecule Testing Framework

The role features a professional Molecule testing suite that validates functionality across multiple Linux distributions:

#### Test Structure
```
molecule/
└── default/
    ├── molecule.yml     # Test configuration with Docker and distribution matrix
    ├── prepare.yml      # Environment preparation and dependency installation
    ├── converge.yml     # Role execution with test configuration
    └── verify.yml       # Comprehensive validation testing (8 categories, 15+ tests)
```

#### Supported Test Distributions
- **Ubuntu 24.04** (Noble Numbat)
- **Ubuntu 22.04** (Jammy Jellyfish)
- **Debian 12** (Bookworm)

#### Test Categories
1. **User & Group Management**: Verify user/group creation, home directory, shell configuration
2. **Directory Structure**: Check installation and log directory creation with proper permissions
3. **RSyslog Integration**: Validate configuration files, content, and service status
4. **Logrotate Configuration**: Test rotation rules, frequency settings, and file paths
5. **Package Dependencies**: Verify required package installation (curl, wget, git, sudo)
6. **Service Configuration**: Test systemd service status and RSyslog operation
7. **File Permissions**: Validate directory and file permission settings (0755, 0644)
8. **Template Processing**: Verify configuration template generation and content

### Running Tests Locally

#### Prerequisites
```bash
# Install testing dependencies
pip3 install molecule molecule-plugins[docker] docker ansible-lint yamllint
```

#### Execute Tests
```bash
# Run full test suite
molecule test

# Test specific distribution
MOLECULE_DISTRO=ubuntu2404 molecule test
MOLECULE_DISTRO=debian12 molecule test

# Run individual test stages
molecule create      # Create test environment
molecule converge    # Run the role
molecule verify      # Run verification tests
molecule destroy     # Clean up test environment
```

#### Quality Validation
```bash
# YAML formatting validation
yamllint .

# Ansible best practices validation
ansible-lint .
```

### CI/CD Pipeline

#### CI Pipeline

Runs on every Pull Request via centralized reusable workflow:

1. **Branch Name Lint** — enforces source branch naming conventions (`feature/`, `bugfix/`, etc.)
2. **PR Title Lint** — enforces Conventional Commits naming conventions on PR titles
3. **YAML Lint** — validates all YAML files
4. **Ansible Lint** — enforces best practices
5. **Security Scan** — TruffleHog secret detection
6. **Molecule Tests** — matrix across Ubuntu 24.04, Ubuntu 22.04, and Debian 12
7. **Merge Check** — aggregated status gate for branch protection

#### Release & Publish

Automated via [Release Please](https://github.com/googleapis/release-please):

1. Merge to `main` → Release Please creates a Release PR with changelog
2. Merge Release PR → creates Git tag + GitHub Release
3. Galaxy publish triggers automatically on release

#### Test Environment Features
- **Mock GitHub Authentication**: Safe testing without real GitHub API calls
- **Isolated Test Directories**: Separate paths for testing (`/opt/actions-runner-test`)
- **Service State Management**: Services enabled but stopped for safe testing
- **Comprehensive Verification**: 15+ individual tests across 8 categories
- **Professional Standards**: Following collection patterns and enterprise practices

### Test Configuration Examples

#### Basic Test Run
```bash
# Quick validation
molecule test --scenario-name default

# With specific distribution
MOLECULE_DISTRO=ubuntu2404 molecule test
```

#### Advanced Test Configuration
```yaml
# Custom test variables in molecule.yml
provisioner:
  inventory:
    group_vars:
      all:
        github_runner_logging_enabled: true
        github_runner_rsyslog_enabled: true
        github_runner_debug: true
```

### Quality Standards

The role maintains production-level quality through:
- **100% Test Coverage**: All role functionality validated
- **Multi-Distribution Support**: Tested across major Linux distributions
- **Professional Patterns**: Following established collection standards
- **Automated Quality Gates**: Prevents regression through CI/CD
- **Mock Testing**: Safe testing without external dependencies

## 🤝 Contributing

Contributions, bug reports, and feature requests are welcome!

- Fork the repository and create your branch from `main`
- Use [Conventional Commits](https://www.conventionalcommits.org/) for commit messages
- Ensure your code passes all CI checks (YAML lint, Ansible lint, Molecule tests)
- Submit a pull request describing your changes (a template is available under `.github/PULL_REQUEST_TEMPLATE/pull_request_template.md` to help structure your PR description)
- For major changes, please open an issue first to discuss what you would like to change (issue templates for bug reports, feature requests, and tasks are available under `.github/ISSUE_TEMPLATE/`)


## 📝 License

This project is licensed under the Apache-2.0 License - see the LICENSE file for details.

## 👥 Author Information

This role was created by [Grzegorz Franus](https://github.com/grzegorzfranus).
