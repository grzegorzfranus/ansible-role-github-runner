# Changelog

All notable changes to this GitHub Runner Ansible role will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.7] - 2025-08-10

### Added ✅
- Token validation and secrecy: assert PAT format with regex; ensured sensitive operations keep `no_log: true`.
- Runner update control: added `--disableupdate` flag support via `github_runner_disable_update`.
- Idempotent uninstall: new `github_runner_state: present|absent` with `tasks/remove.yml` to unregister and clean files.
- UID/GID defaults: `github_runner_user_uid` and `github_runner_group_gid` now default to `null` and are omitted unless set.
- Download integrity: optional checksum verification (`github_runner_verify_checksum`, `github_runner_checksums_file`) and checksum parsing.
- RSyslog handler: switched to graceful `state: reloaded` for rsyslog changes.
- Documentation: README updated with Vault guidance, new variables, and checksum usage.

### Changed 🔄
- Runner environment: optional `RUNNER_TOOL_CACHE` variable support via `github_runner_tool_cache`.

### Fixed 🔧
- RSyslog template now uses valid RainerScript and severity filter for error logs.

---

## [1.0.6] - 2025-06-30

### Added ✅
- **Comprehensive Molecule Testing Framework** - Professional test suite for role validation
  - Complete Molecule test configuration with Docker-based testing environment
  - Four-stage testing pipeline: prepare → converge → verify → destroy
  - Support for multiple Linux distributions (Ubuntu, Debian, Rocky Linux)
  - Mock GitHub authentication for safe testing without real GitHub API calls
  - Comprehensive verification of all role components and features

- **CI/CD Pipeline Implementation** - GitHub Actions workflows for automated testing and publishing
  - **Test & Validation Pipeline** (`test-and-validation.yml`) - Automated testing on push/PR
    - YAML linting with yamllint for code quality assurance
    - Molecule testing across multiple distributions (Ubuntu 24.04, Debian 12)
    - Matrix strategy for parallel testing of different scenarios
    - Proper environment variable configuration for test execution
  - **Galaxy Publishing Pipeline** (`publish-to-galaxy.yml`) - Automated role publishing
    - Triggered on GitHub release creation
    - Automatic role import to Ansible Galaxy
    - Secure API key management through GitHub secrets

### Test Suite Features 🧪
- **molecule.yml Configuration**:
  - Docker driver with privileged containers for systemd testing
  - Configurable distribution matrix using environment variables
  - Professional ansible-lint integration for code quality
  - Mock GitHub configuration (token, organization, labels)
  - Dedicated logging configuration for comprehensive testing

- **prepare.yml - Environment Preparation**:
  - System preparation with permission fixes and directory creation
  - Package installation (curl, wget, git, rsyslog, logrotate)
  - Service initialization and systemd readiness verification
  - Test directory creation with proper ownership
  - Mock GitHub API response generation for isolated testing

- **converge.yml - Role Execution**:
  - Selective role action execution (prerequisites, user, install, logging)
  - Test-specific configuration with separated paths
  - Mock authentication setup for safe testing
  - Service state management (enabled but stopped for testing)
  - Debug mode activation for detailed test output

- **verify.yml - Comprehensive Validation** (8 test categories, 15+ individual tests):
  - **User & Group Management**: Test user/group creation, home directory, shell configuration
  - **Directory Structure**: Verify installation and log directory creation with proper permissions
  - **RSyslog Integration**: Check configuration files, content validation, service status
  - **Logrotate Configuration**: Validate rotation rules, frequency settings, file paths
  - **Package Dependencies**: Verify required package installation (curl, wget, git, sudo)
  - **Service Configuration**: Test systemd service status and RSyslog service operation
  - **File Permissions**: Validate directory and file permission settings (0755, 0644)
  - **Template Processing**: Verify configuration template generation and content

### Technical Implementation 🔧
- **Test Environment Isolation**:
  - Separate test directories (`/opt/actions-runner-test`, `/var/log/github-runner-test`)
  - Dedicated test user (`test-runner`) with isolated configuration
  - Mock GitHub authentication preventing actual API calls
  - Safe configuration override for testing scenarios

- **Quality Assurance Integration**:
  - YAML formatting validation with yamllint (fixed all trailing spaces and newlines)
  - Ansible best practices validation with ansible-lint (production profile)
  - Comprehensive test coverage for all role functionality
  - Matrix testing across multiple Linux distributions

- **Professional Testing Standards**:
  - Following established patterns from `ansible-role-tailscale` and collection standards
  - Comprehensive assertion-based testing with clear pass/fail messages
  - Proper test documentation with emoji status indicators
  - Enterprise-grade test organization and structure

### CI/CD Features 🚀
- **Automated Quality Gates**:
  - Pull request validation with comprehensive testing
  - Branch protection through automated testing
  - Multi-distribution compatibility verification
  - Lint and test failure prevention for main branch

- **Release Automation**:
  - Automatic Galaxy publishing on tagged releases
  - Secure credential management through GitHub Actions secrets
  - Professional release workflow with proper version management

### Documentation Updates 📚
- **Test Documentation**: Added comprehensive testing information to README.md
- **CI/CD Documentation**: Documented automated workflows and testing procedures
- **Quality Standards**: Updated quality assurance information with test coverage details

### Development Workflow Enhancement 🛠️
- **Pre-commit Quality Checks**: Automated linting and validation
- **Distribution Testing**: Multi-OS compatibility verification
- **Mock Testing Environment**: Safe testing without external dependencies
- **Professional Standards**: Enterprise-grade testing and validation framework

### Migration and Compatibility ✅
- **Zero Breaking Changes**: All existing functionality preserved
- **Backward Compatibility**: Full compatibility with existing configurations
- **Optional Testing**: Molecule tests don't affect production usage
- **Standard Compliance**: Follows Ansible Galaxy and community best practices

This major enhancement establishes professional testing and CI/CD standards for the GitHub Runner role, ensuring reliability, quality, and compatibility across multiple Linux distributions while maintaining production-ready stability.

## [1.0.5] - 2025-06-29

### Added ✅
- **Professional RSyslog Integration** - Implemented enterprise-grade logging with RSyslog support
  - Added RSyslog package installation and configuration
  - Created RSyslog template (`rsyslog/github-runner.conf.j2`) for GitHub Runner log routing
  - Implemented systemd service override for SyslogIdentifier configuration
  - Added automatic RSyslog service restart handlers
  - Provides professional logging approach following collection standards

- **Enhanced Logging Architecture** - Comprehensive logging system with multiple approaches
  - **RSyslog approach (recommended)**: Service → systemd journal → RSyslog → dedicated files
  - **Legacy approach**: Service → direct file output (for backward compatibility)
  - Automatic log ownership management (syslog:adm for RSyslog, github-runner:github-runner for legacy)
  - Intelligent logrotate service restart (rsyslog reload for RSyslog approach, service restart for legacy)

### Configuration Variables 📊
- **RSyslog Integration**:
  - `github_runner_rsyslog_enabled` - Enable RSyslog integration (default: `true`)
  - `github_runner_syslog_identifier` - Syslog program identifier (default: `"github-runner"`)
  - `github_runner_rsyslog_config_file` - RSyslog config file name (default: `"49-github-runner.conf"`)
  - `github_runner_log_user` - Log file owner for RSyslog (default: `"syslog"`)
  - `github_runner_log_group` - Log file group for RSyslog (default: `"adm"`)

### Enhanced Features 🚀
- **Intelligent Service Configuration** - Systemd service template automatically adapts to logging approach
  - RSyslog enabled: StandardOutput/StandardError → journal (RSyslog handles file routing)
  - RSyslog disabled: StandardOutput/StandardError → direct file append (legacy mode)
  - Configurable SyslogIdentifier based on `github_runner_syslog_identifier` variable

- **Professional Log Management** - Following enterprise logging patterns
  - RSyslog configuration follows tailscale/wireguard/nftables pattern from collection
  - Log filtering and routing based on program name
  - Centralized log management capability with RSyslog
  - Structured logging support for integration with log analysis tools

### Technical Implementation 🔧
- **Systemd Service Override** - Automatic configuration for RSyslog integration
  - Creates `/etc/systemd/system/[service].service.d/logging.conf` when RSyslog enabled
  - Sets StandardOutput=syslog, StandardError=syslog, SyslogIdentifier=[identifier]
  - Automatic cleanup when RSyslog is disabled
  - Proper daemon-reload and service restart handling

- **Enhanced Tasks Structure** - Updated logging.yml with comprehensive flow
  - Log directory and file creation with dynamic ownership
  - RSyslog package installation and configuration
  - Systemd service override management
  - Logrotate configuration with approach-specific service restart
  - Enhanced status reporting with RSyslog configuration details

### Documentation Updates 📚
- **Updated README.md** - Comprehensive RSyslog documentation
  - Added "RSyslog Integration (Professional Approach)" section to variables
  - Updated "Check Logs" section with RSyslog verification commands
  - Enhanced file structure documentation with RSyslog template
  - Updated usage examples with professional logging configuration

### Backward Compatibility ✅
- **No Breaking Changes** - Full backward compatibility maintained
  - RSyslog enabled by default but gracefully handles missing packages
  - Legacy direct file logging preserved when RSyslog disabled
  - Existing configurations continue to work without modification
  - Automatic migration to professional logging approach

### Quality Assurance 🧪
- **Enhanced Error Handling** - Robust configuration with fallback mechanisms
  - Conditional RSyslog configuration based on availability
  - Automatic cleanup when features are disabled
  - Proper service restart coordination between systemd and rsyslog
  - Enhanced debug output for troubleshooting

This major enhancement brings enterprise-grade logging capabilities to the GitHub Runner role, following the established patterns from other roles in the collection while maintaining full backward compatibility.

## [1.0.4] - 2025-06-29

### Fixed 🔧
- **Logrotate Size Option** - Fixed "dict object has no attribute 'size'" error in logrotate template
  - Made the `size` option conditional in logrotate template (`{% if github_runner_logrotate_options.size is defined %}`)
  - Commented out the default `size: "100M"` option in `defaults/main.yml` to allow users to opt-in
  - Users can now exclude the size-based rotation entirely or add it back if needed
  - Resolves issue where logrotate would fail when size attribute was not properly defined

### Changed 📝
- **Logrotate Configuration** - Size-based log rotation is now optional
  - Default behavior: logs rotate based on frequency (daily) and count (30) only
  - To enable size-based rotation: add `size: "100M"` to your `github_runner_logrotate_options` configuration
  - This change provides more flexibility for different log management preferences

## [1.0.3] - 2025-06-29

### Added ✅
- **Dedicated Logging Support** - Added comprehensive logging functionality to redirect GitHub Runner logs to dedicated files
  - New logging task file (`logging.yml`) for log directory and file management
  - Configurable log directory path with default `/var/log/github-runner`
  - Separate log files for standard output (`runner.log`) and errors (`runner-error.log`)
  - Automatic log directory and file creation with proper ownership and permissions
  - Integration with systemd service to redirect output from journal to files when enabled
  - Conditional logging configuration - defaults to systemd journal logging for backward compatibility

- **Logrotate Integration** - Comprehensive log rotation management to prevent disk space issues
  - Automatic logrotate configuration generation (`/etc/logrotate.d/github-runner`)
  - Configurable rotation frequency (daily, weekly, monthly) with daily default
  - Configurable retention count (default 30 rotations)
  - Maximum file size rotation trigger (default 100M)
  - Compression support with delay compression option
  - Proper file permissions and ownership after rotation
  - Service reload integration for log file reopening

### Configuration Variables 📊
- **Logging Control**:
  - `github_runner_logging_enabled` - Enable/disable dedicated logging (default: `false`)
  - `github_runner_log_dir` - Log directory path (default: `/var/log/github-runner`)
  - `github_runner_log_file` - Main log file name (default: `runner.log`)
  - `github_runner_log_error_file` - Error log file name (default: `runner-error.log`)
  - `github_runner_log_dir_mode` - Log directory permissions (default: `0755`)
  - `github_runner_log_file_mode` - Log file permissions (default: `0644`)

- **Logrotate Configuration** (`github_runner_logrotate_options` dictionary):
  - `enabled` - Enable logrotate configuration (default: `true`)
  - `frequency` - Rotation frequency (default: `daily`)
  - `rotate_count` - Number of rotated files to keep (default: `30`)
  - `size` - Maximum file size before rotation (default: `100M`)
  - `compress` - Compress rotated logs (default: `true`)
  - `delaycompress` - Delay compression by one cycle (default: `true`)
  - `missingok` - Don't error if log file is missing (default: `true`)
  - `notifempty` - Only rotate if log file is not empty (default: `true`)
  - `copytruncate` - Use copytruncate instead of moving files (default: `false`)
  - `dateext` - Use date extension for rotated files (default: `false`)
  - `dateformat` - Date format for rotated files (default: `.%Y-%m-%d`)
  - `olddir` - Directory to move old log files to (default: `""`)
  - `create_mode` - File permissions for newly created logs (default: `0644`)
  - `create_owner` - Owner for newly created log files (default: `github_runner_user`)
  - `create_group` - Group for newly created log files (default: `github_runner_user_group`)

### Enhanced Features 🚀
- **Task Organization** - Added new `logging` tag and task group for selective execution
- **Role Action Support** - Extended `github_runner_role_action` to include `logging` option
- **Template System** - New logrotate template (`logrotate/github-runner.j2`) for log management
- **Service Integration** - Updated systemd service template to support both journal and file logging modes
- **Debug Output** - Enhanced debug mode with logging configuration status display
- **Backward Compatibility** - Logging is disabled by default to maintain existing behavior

### Technical Implementation 🔧
- **Conditional Output Redirection** - Systemd service uses `append:` directive for log files when enabled
- **Permission Management** - Proper file and directory ownership with configurable permissions
- **Service Reload** - Logrotate integration with service reload for log file reopening
- **Error Handling** - Comprehensive error handling in logrotate configuration
- **Security** - Log rotation runs under GitHub Runner user context for security

### Usage Examples 💡
```yaml
# Enable dedicated logging
github_runner_logging_enabled: true
github_runner_log_dir: "/var/log/github-runner"

# Custom logrotate settings
github_runner_logrotate_options:
  enabled: true
  frequency: "weekly"
  rotate_count: 52
  size: "500M"
  compress: true
  dateext: true
```

This enhancement addresses syslog noise by providing dedicated log file management while maintaining full backward compatibility with existing systemd journal logging.

### Changed 📝
- **Logrotate Variable Structure** - Restructured logrotate configuration variables into a single `github_runner_logrotate_options` dictionary
  - Follows consistent pattern from other roles in the collection (tailscale, chrony)
  - Improved template structure with proper jinja2 headers and conditional blocks
  - Enhanced configurability with additional options like `missingok`, `notifempty`, `dateext`, etc.
  - **Breaking Change**: Individual logrotate variables consolidated into structured format

### Migration Guide 🔄
**Old format:**
```yaml
github_runner_logrotate_enabled: true
github_runner_logrotate_frequency: "daily"
github_runner_logrotate_rotate_count: 30
github_runner_logrotate_max_size: "100M"
```

**New format:**
```yaml
github_runner_logrotate_options:
  enabled: true
  frequency: "daily"
  rotate_count: 30
  size: "100M"
```

## [1.0.2] - 2025-06-25

### Fixed 🔧
- **Documentation Table Structure** - Removed unused "CI" column from README.md header table
  - Cleaned up repository status table by removing empty CI column that had no badge content
  - Updated table headers and separators to maintain proper markdown formatting
  - Improved documentation readability and consistency

## [1.0.1] - 2025-06-25

### Fixed 🔧
- **SSH Key Generation Directory Creation** - Fixed issue where SSH directory was not created before generating SSH keys
  - Reordered SSH tasks to set up verified paths before directory creation
  - Fixed `when` conditions syntax using proper Ansible YAML folded scalar format (`>`)
  - Added parentheses for clarity in complex boolean conditions
- **SSH Key File Permissions** - Fixed "file is not readable" error when reading generated SSH public key
  - Added `become: true` to SSH public key reading task for proper file access permissions
- **SSH Configuration Validation** - Improved SSH key configuration detection in user summary
  - Updated SSH key status logic to properly handle all SSH configuration methods (import, file, generation)
  - Enhanced debug output to show both original and verified SSH directory paths

### Changed 📝
- **SSH Task Organization** - Improved task flow and conditional execution for SSH key management
  - Consolidated SSH path verification into shared setup tasks
  - Simplified conditional logic across all SSH-related tasks
  - Enhanced debugging output with verified vs original path comparison

### Technical Details 🔍
- **Error Resolution**: Fixed "The directory /home/github-runner/.ssh does not exist" by ensuring proper task execution order
- **Permission Fix**: Resolved SSH public key read permissions by adding privilege escalation to file reading tasks
- **Syntax Improvement**: Converted multi-line `when` conditions from list format to folded scalar format for better reliability

## [1.0.0] - 2025-06-25

### Added ✅
- **GitHub Runner Installation** - Automatic download and installation of latest GitHub Actions Runner
- **Dual Mode Support** - Complete support for both repository-level and organization-level runners
- **User Management System** - Dedicated system user creation with configurable sudo privileges
- **Service Integration** - Full systemd service configuration with restart policies and resource limits
- **Security Hardening** - Comprehensive security restrictions and sandboxing features
- **Version Management** - Support for latest version auto-detection and specific version pinning
- **Dependency Management** - Automatic installation of required and optional package dependencies
- **Network Configuration** - Proxy support and SSL verification controls
- **Environment Customization** - Custom environment variables and hook scripts support
- **Health Verification** - Built-in verification and connectivity testing suite

### Features 🚀
- **Automatic Latest Version Detection** - Fetches and installs the latest GitHub Runner release automatically
- **Architecture Support** - Full support for x64 and arm64 architectures with auto-detection
- **Flexible Authentication** - Support for both Personal Access Tokens and direct registration tokens
- **Resource Management** - Configurable memory limits, CPU quotas, and process restrictions
- **Comprehensive Logging** - Detailed logging with systemd journal integration and debug capabilities
- **SSH Key Management** - Optional SSH key deployment for runner user authentication

- **File Permission Controls** - Secure file ownership and permission management
- **Cleanup Automation** - Automatic cleanup of temporary files and installation artifacts
- **Configuration Validation** - Extensive variable validation and system compatibility checks

### Security Enhancements 🛡️
- **Systemd Security** - Implementation of comprehensive systemd security features:
  - NoNewPrivileges, PrivateTmp, ProtectSystem, ProtectHome
  - Device access control and capability restrictions
  - System call filtering and namespace restrictions
- **User Isolation** - Dedicated system user with minimal required privileges
- **File System Protection** - Read-only system areas with specific read-write path allowances
- **Resource Limits** - Memory, CPU, and task limits to prevent resource exhaustion
- **Sudo Configuration** - Optional sudo privileges with detailed logging and environment preservation
- **SSL Verification** - Configurable SSL certificate validation for secure communications
- **Telemetry Controls** - Options to disable telemetry and analytics collection

### Installation and Configuration 📦
- **Prerequisites Management** - Automatic system requirements validation and dependency installation
- **Network Connectivity** - GitHub API and releases connectivity verification
- **Disk Space Validation** - Minimum disk space and memory requirements checking
- **Package Installation** - Support for both required and optional package dependencies across distributions
- **Service Configuration** - Complete systemd service file generation with environment variables
- **Override Support** - Systemd service override files for resource limits and security settings

### Operating System Support 🐧
- **Ubuntu Support** - Full support for Ubuntu 22.04 (Jammy) and 24.04 (Noble)
- **Debian Support** - Complete compatibility with Debian 12 (Bookworm)
- **Enterprise Linux** - Support for Rocky Linux 9 and Oracle Linux 9
- **Package Management** - Automatic package manager detection and usage (apt, yum, dnf)
- **Distribution Variables** - OS-specific variable loading and configuration

### Task Organization and Flow 🔄
- **Modular Task Structure** - Organized task files for each major component:
  - `main.yml` - Main orchestration and flow control
  - `assert.yml` - Comprehensive variable validation (50+ validation checks)
  - `prerequisites.yml` - System requirements and dependency management
  - `user.yml` - User and group creation with SSH and sudo configuration
  - `install.yml` - GitHub Runner download, extraction, and installation
  - `configure.yml` - Runner registration and configuration with GitHub
  - `service.yml` - Systemd service creation and management
  - `verify.yml` - Installation verification and health checks

### Template System 📄
- **Systemd Service Template** - Complete service configuration with security hardening
- **Service Override Template** - Resource limits and additional security restrictions
- **Environment Template** - Runner environment variables and configuration
- **Sudoers Template** - Secure sudo configuration with logging and environment preservation

### Variable System and Customization ⚙️
- **Comprehensive Defaults** - Production-ready default configuration values
- **Flexible Customization** - 50+ configurable variables for complete customization
- **Validation Framework** - Extensive variable validation with clear error messages
- **Documentation** - Comprehensive variable documentation with examples and use cases

### Debugging and Troubleshooting 🔍
- **Debug Mode** - Optional debug mode with detailed output and verification
- **Status Reporting** - Comprehensive status summaries throughout execution
- **Error Handling** - Clear error messages with suggested resolutions
- **Log Integration** - Service log monitoring and display capabilities
- **Health Checks** - Multi-level health verification and reporting

### Handler System 🔄
- **Service Management** - Handlers for systemd daemon reload and service restart
- **State Management** - Proper service state management with start/stop/restart capabilities
- **Dependency Handling** - Proper handler dependency and execution order

### Documentation and Examples 📚
- **Comprehensive README** - Complete documentation with architecture diagrams and examples
- **Configuration Examples** - Multiple real-world configuration examples
- **Troubleshooting Guide** - Detailed troubleshooting procedures and solutions
- **Security Documentation** - Security feature documentation and best practices
- **API Documentation** - Complete variable documentation with types and defaults

### Code Quality and Standards 🏆
- **Emoji Integration** - Consistent emoji usage following repository standards (🧪 🔍 ✅ ❌ 📊 🚀 etc.)
- **English-Only Content** - All content in English following repository policies
- **Professional Naming** - Consistent and professional task naming conventions
- **Error Handling** - Comprehensive error handling with meaningful messages
- **Performance Optimization** - Efficient task execution with proper conditionals and checks

### Testing and Validation Framework 🧪
- **System Validation** - Operating system and architecture compatibility checks
- **Network Testing** - GitHub API and connectivity validation
- **Installation Verification** - Complete installation file and permission verification
- **Service Testing** - Service status and health verification
- **Configuration Validation** - Runner configuration and registration verification
- **Resource Monitoring** - System resource usage and availability checks

### Ansible Integration 🔧
- **Role Metadata** - Complete Galaxy metadata with platform and tag information
- **Variable Loading** - OS-specific variable loading with fallback mechanisms
- **Fact Gathering** - Proper Ansible fact usage and validation
- **Module Usage** - Extensive use of Ansible built-in modules for reliability
- **Tag Support** - Comprehensive tagging for selective task execution

### Enterprise Features 🏢
- **GitHub Enterprise Support** - Full compatibility with GitHub Enterprise Server
- **Organization Management** - Complete organization-level runner support
- **Group Management** - Runner group assignment and management
- **Token Management** - Support for both PAT and registration token authentication
- **Proxy Support** - HTTP proxy configuration for enterprise environments

This initial release provides a complete, production-ready solution for GitHub Actions Runner deployment with comprehensive security, flexibility, and reliability features.
