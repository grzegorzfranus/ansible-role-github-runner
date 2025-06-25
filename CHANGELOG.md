# Changelog

All notable changes to this GitHub Runner Ansible role will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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

## [1.0.2] - 2025-06-25

### Fixed 🔧
- **Documentation Table Structure** - Removed unused "CI" column from README.md header table
  - Cleaned up repository status table by removing empty CI column that had no badge content
  - Updated table headers and separators to maintain proper markdown formatting
  - Improved documentation readability and consistency 