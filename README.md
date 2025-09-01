
# FMC/FTD Ansible Automation Suite - Comprehensive Summary

## Project Overview

This Ansible automation suite provides a comprehensive **read-only assessment and validation toolkit** for Cisco Firepower Management Center (FMC) and Firepower Threat Defense (FTD) deployments. The suite is specifically designed for security teams to perform compliance checking, security assessment, and troubleshooting operations without any risk of configuration changes in production environments.

**Key Characteristics:**
- **Safety First**: All operations are strictly read-only (GET requests only)
- **Production Safe**: No configuration changes, deployments, or policy modifications
- **Comprehensive Assessment**: Covers security hardening, rule analysis, connectivity testing, and packet tracing
- **Modular Architecture**: 8 specialized roles with conditional execution
- **Detailed Reporting**: Generates markdown reports for compliance and audit purposes

## Architecture Overview

### Project Structure
```
fmc-ansible/
├── site.yml                           # Main playbook entry point
├── playbooks/
│   └── roles/
│       └── main.yml                   # Primary role orchestration playbook
├── roles/                             # 8 specialized assessment roles
│   ├── fmc_common/                    # Common setup and validation
│   ├── fmc_connection/                # FMC API connection testing
│   ├── ftd_discovery/                 # FTD device discovery
│   ├── fmc_hardening/                 # FMC security hardening assessment
│   ├── ftd_hardening/                 # FTD security hardening validation
│   ├── fmc_rule_management/           # Time-based FMC rule analysis
│   ├── ftd_rule_management/           # Device-level FTD rule management
│   └── fmc_packet_tracer/            # Network packet tracing utilities
├── inventory/                         # Ansible inventory configurations
└── reports/                          # Generated assessment reports
```

### Technology Stack
- **Ansible**: 2.9+ with cisco.fmcansible collection
- **Target Systems**: Cisco FMC 6.6+ and FTD devices
- **API Integration**: FMC REST API for all operations
- **Reporting**: Markdown format with structured findings

## Detailed Role Documentation

### 1. fmc_common Role
**File**: `roles/fmc_common/tasks/main.yml` (143 lines)

**Purpose**: Provides foundational setup, validation, and prerequisite checks for all other roles.

**Key Functions:**
- Validates Ansible environment and required collections
- Establishes common variables and configuration baselines
- Performs initial system readiness checks
- Sets up logging and reporting directory structure
- Validates inventory configuration and host accessibility

**Dependencies**: None (always executed first)

**Tags**: `['always', 'common']`

### 2. fmc_connection Role
**File**: `roles/fmc_connection/tasks/main.yml` (173 lines)

**Purpose**: Tests and validates FMC API connectivity and authentication.

**Key Functions:**
- Tests FMC API endpoint availability and responsiveness
- Validates authentication credentials and permissions
- Checks API version compatibility
- Verifies SSL/TLS certificate status
- Tests API rate limiting and timeout configurations
- Generates connectivity status reports

**Safety Features:**
- Only uses GET requests for API testing
- No authentication token persistence
- Graceful handling of connection failures

**Conditional Execution**: `fail_on_connection_error: true`

**Tags**: `['connection', 'test']`

### 3. ftd_discovery Role
**File**: `roles/ftd_discovery/tasks/main.yml` (237 lines)

**Purpose**: Discovers and validates FTD device connectivity and registration status.

**Key Functions:**
- Discovers all registered FTD devices in the FMC
- Tests management connectivity to each FTD device
- Validates device registration status and health
- Checks device version compatibility
- Assesses high availability status for HA pairs
- Generates device inventory and connectivity matrix

**Key Variables Set:**
- `ftd_devices_found`: Boolean flag for subsequent role conditions
- `ftd_device_list`: Inventory of discovered devices
- `ftd_connectivity_status`: Per-device connectivity results

**Conditional Execution**: `discover_ftd_devices | default(true)`

**Tags**: `['ftd', 'discovery', 'test']`

### 4. fmc_hardening Role
**File**: `roles/fmc_hardening/tasks/main.yml` (397 lines)

**Purpose**: Comprehensive security hardening assessment for FMC platform.

**Key Functions:**
- **Password Policy Validation**: Checks complexity requirements, expiration policies, lockout settings
- **Authentication Assessment**: Validates multi-factor authentication, session timeouts, login banners
- **Access Control Review**: Analyzes user roles, permissions, and administrative access controls
- **Logging Configuration**: Verifies audit logging, syslog configuration, and log retention policies
- **Network Security**: Assesses management interface security, HTTPS configuration, certificate validity
- **System Hardening**: Checks system services, unnecessary features, and security patches

**Assessment Areas:**
- User account security and password policies
- Administrative access controls and role-based permissions
- Audit and security logging configuration
- Network interface and protocol security
- Certificate management and PKI configuration
- System services and feature hardening

**Conditional Execution**: `extract_hardening_config | default(true)`

**Tags**: `['fmc', 'hardening', 'config']`

### 5. ftd_hardening Role
**File**: `roles/ftd_hardening/tasks/main.yml` (682 lines)

**Purpose**: Comprehensive security hardening validation for FTD devices.

**Key Functions:**
- **Interface Security Assessment**: Validates interface configurations, security zones, and access controls
- **Security Policy Review**: Analyzes security policies, intrusion prevention settings, and malware protection
- **Network Segmentation**: Validates VLAN configurations, routing security, and network isolation
- **Threat Intelligence**: Checks threat intelligence feeds, reputation services, and security intelligence
- **VPN Configuration**: Assesses VPN settings, certificate management, and encryption standards
- **High Availability**: Validates HA configuration, failover mechanisms, and synchronization status

**Advanced Security Checks:**
- Intrusion Prevention System (IPS) policy validation
- Advanced Malware Protection (AMP) configuration
- URL filtering and web security policies
- Application visibility and control settings
- File policy and malware detection rules
- Network analysis and behavioral monitoring

**Dependencies**: Requires `ftd_devices_found: true`

**Conditional Execution**:
- `ftd_hardening_enabled | default(true)`
- `ftd_devices_found | default(false)`

**Tags**: `['ftd', 'hardening', 'security']`

### 6. fmc_rule_management Role
**File**: `roles/fmc_rule_management/tasks/main.yml` (550 lines)

**Purpose**: Time-based analysis and management of FMC access control rules.

**Key Functions:**
- **Rule Inventory**: Catalogs all access control policies and rules
- **Rule Analysis**: Analyzes rule usage, hit counts, and effectiveness
- **Time-Based Assessment**: Identifies rules with expiration dates or time-based conditions
- **Rule Optimization**: Identifies redundant, shadowed, or ineffective rules
- **Policy Validation**: Validates rule ordering, precedence, and logical consistency
- **Compliance Checking**: Ensures rules meet organizational security standards

**Analysis Categories:**
- Rule utilization and hit count analysis
- Expired or soon-to-expire time-based rules
- Overly permissive or broad rules (Any-Any rules)
- Shadowed rules that never match traffic
- Duplicate or redundant rule detection
- Rule ordering and precedence issues

**Conditional Execution**:
- `manage_rules | default(true)`
- `rule_expiry_check_enabled | default(true)`

**Tags**: `['rules', 'management', 'fmc']`

### 7. ftd_rule_management Role
**File**: `roles/ftd_rule_management/tasks/main.yml` (1,100 lines)

**Purpose**: Device-level FTD rule analysis and management - the most comprehensive role.

**Key Functions:**
- **Device-Specific Rule Analysis**: Analyzes rules applied to specific FTD devices
- **Rule Performance Assessment**: Evaluates rule performance and resource utilization
- **Traffic Pattern Analysis**: Correlates rules with actual traffic patterns and flows
- **Rule Effectiveness Metrics**: Measures rule hit rates, block rates, and allow rates
- **Policy Deployment Status**: Validates rule deployment status across device groups
- **Rule Conflict Detection**: Identifies conflicting or contradictory rules

**Advanced Features:**
- Per-device rule performance metrics
- Rule-to-traffic correlation analysis
- Device group policy consistency validation
- Rule modification impact assessment
- Custom rule validation frameworks
- Automated rule optimization recommendations

**Comprehensive Coverage:**
- Access control rule analysis
- NAT rule evaluation and optimization
- VPN rule assessment and validation
- Intrusion prevention rule analysis
- Quality of Service (QoS) rule review
- Application control rule validation

**Dependencies**:
- Requires `ftd_devices_found: true`
- Depends on successful FTD discovery

**Conditional Execution**:
- `manage_ftd_rules | default(true)`
- `ftd_devices_found | default(false)`
- `ftd_rule_management_enabled | default(true)`

**Tags**: `['ftd', 'rules', 'management']`

### 8. fmc_packet_tracer Role
**File**: `roles/fmc_packet_tracer/tasks/main.yml` (639 lines)

**Purpose**: Network packet tracing and flow analysis for troubleshooting.

**Key Functions:**
- **Packet Flow Simulation**: Simulates packet flows without generating actual traffic
- **Policy Match Analysis**: Identifies which policies and rules match specific traffic
- **Path Validation**: Validates expected packet paths through FTD devices
- **Connectivity Troubleshooting**: Diagnoses connectivity issues and traffic blocking
- **Rule Hit Analysis**: Determines which rules are triggered by specific traffic patterns
- **Network Topology Mapping**: Maps traffic flows through network topology

**Troubleshooting Capabilities:**
- Source-to-destination connectivity testing
- Policy match and rule hit identification
- NAT translation validation
- VPN tunnel path analysis
- Interface-specific traffic flow analysis
- Security zone traversal validation

**Required Parameters**:
- `source_ip`: Source IP address for packet trace
- `dest_ip`: Destination IP address for packet trace
- `interface_id`: Source interface identifier
- `interface_name`: Source interface name
- `interface_type`: Interface type specification

**Conditional Execution**:
- `run_packet_tracer | default(false)` (disabled by default)
- All required packet trace parameters must be defined

**Tags**: `['packet', 'tracer']`

## Execution Workflow

### Sequential Execution Order
The roles execute in a carefully orchestrated sequence with dependency management:

1. **fmc_common** - Always executes first for foundational setup
2. **fmc_connection** - Tests FMC API connectivity
3. **ftd_discovery** - Discovers FTD devices (sets `ftd_devices_found`)
4. **fmc_hardening** - Assesses FMC security hardening
5. **ftd_hardening** - Validates FTD security (requires FTD devices)
6. **fmc_rule_management** - Analyzes FMC-level rules
7. **ftd_rule_management** - Analyzes device-level rules (requires FTD devices)
8. **fmc_packet_tracer** - Performs packet tracing (optional, requires parameters)

### Conditional Execution Logic
```yaml
# FTD-dependent roles only execute when devices are found
when: ftd_devices_found | default(false) | bool

# User-controllable role execution
when: manage_rules | default(true) | bool

# Parameter-dependent execution
when:
  - run_packet_tracer | default(false) | bool
  - source_ip is defined
  - dest_ip is defined
```

### Tag-Based Execution
Users can execute specific subsets of roles using Ansible tags:

```bash
# Execute only connection testing
ansible-playbook site.yml --tags "connection,test"

# Execute only hardening assessments
ansible-playbook site.yml --tags "hardening"

# Execute only rule management
ansible-playbook site.yml --tags "rules,management"

# Execute FTD-specific tasks only
ansible-playbook site.yml --tags "ftd"
```

## Safety and Security Features

### Read-Only Operations
- **API Calls**: Only GET requests are used
- **No Configuration Changes**: No POST, PUT, or DELETE operations
- **No Policy Deployments**: No activation or deployment of policies
- **No Rule Modifications**: No creation, modification, or deletion of rules
- **No Object Changes**: No modification of network objects or groups

### Production Safety
- **Check Mode Support**: All tasks support `--check` mode for simulation
- **Graceful Error Handling**: Continues assessment even if individual checks fail
- **Connection Failure Tolerance**: Gracefully handles API timeouts and connectivity issues
- **Idempotent Operations**: Can be run multiple times safely without side effects

### Security Best Practices
- **Credential Protection**: Supports Ansible Vault for credential encryption
- **Minimal Privileges**: Only requires read-only API access
- **Audit Trail**: Comprehensive logging of all operations
- **Certificate Validation**: Validates SSL/TLS certificates for secure connections

## Configuration Requirements

### Inventory Configuration
```yaml
[fmc_servers]
fmc-production ansible_host=10.10.10.10
fmc-sandbox ansible_host=10.12.201.212

[fmc_servers:vars]
fmc_username=admin
fmc_password={{ vault_fmc_password }}
fmc_protocol=https
fmc_port=443
```

### Required Variables
```yaml
# Directory configurations
report_output_dir: "./reports"
log_output_dir: "./logs"

# Feature toggles
discover_ftd_devices: true
extract_hardening_config: true
ftd_hardening_enabled: true
manage_rules: true
manage_ftd_rules: true
rule_expiry_check_enabled: true

# Optional packet tracer parameters
run_packet_tracer: false
# source_ip: "192.168.1.10"
# dest_ip: "10.0.0.100"
# interface_id: "005056ba-cc55-0370-0000-171798699134"
# interface_name: "inside"
# interface_type: "SecurityZone"
```

## Output and Reporting

### Report Generation
All roles contribute to comprehensive markdown reports stored in the configured output directory:

```
reports/
├── fmc_assessment_report_YYYY-MM-DD_HH-MM-SS.md
├── ftd_assessment_report_YYYY-MM-DD_HH-MM-SS.md
├── rule_management_report_YYYY-MM-DD_HH-MM-SS.md
└── packet_tracer_report_YYYY-MM-DD_HH-MM-SS.md
```

### Report Structure
Each report follows a standardized format:
- **Executive Summary**: High-level findings and compliance status
- **Detailed Findings**: Categorized assessment results with severity levels
- **Recommendations**: Prioritized action items for remediation
- **Technical Details**: Raw data, configurations, and diagnostic information
- **Appendix**: Supporting documentation and reference materials

### Finding Severity Levels
- **Critical**: Security vulnerabilities requiring immediate attention
- **High**: Significant security or compliance issues
- **Medium**: Important configuration improvements
- **Low**: Minor optimization opportunities
- **Info**: Informational findings and baseline documentation

## Prerequisites and Dependencies

### System Requirements
- **Ansible**: Version 2.9 or higher
- **Python**: Version 3.6 or higher
- **Collections**: cisco.fmcansible, ansible.builtin
- **Network Access**: HTTPS connectivity to FMC on port 443

### FMC Requirements
- **FMC Version**: 6.6 or higher
- **API Access**: REST API must be enabled
- **User Account**: Read-only user account with appropriate permissions
- **Network Connectivity**: Management network access to FMC and FTD devices

### Permissions Required
- **FMC API Access**: Read-only access to all configuration areas
- **Device Management**: Read access to device inventory and status
- **Policy Management**: Read access to policies and rules
- **System Information**: Read access to system health and status

## Usage Examples

### Basic Execution
```bash
# Run complete assessment suite
ansible-playbook site.yml

# Run with specific inventory
ansible-playbook site.yml -i production_inventory.ini

# Run in check mode (simulation only)
ansible-playbook site.yml --check
```

### Selective Execution
```bash
# Connection testing only
ansible-playbook site.yml --tags "connection,test"

# Hardening assessment only
ansible-playbook site.yml --tags "hardening"

# Rule management only
ansible-playbook site.yml --tags "rules,management"

# FTD-specific tasks
ansible-playbook site.yml --tags "ftd"
```

### Variable Overrides
```bash
# Disable FTD discovery
ansible-playbook site.yml -e "discover_ftd_devices=false"

# Enable packet tracer with parameters
ansible-playbook site.yml -e "run_packet_tracer=true" \
  -e "source_ip=192.168.1.10" \
  -e "dest_ip=10.0.0.100" \
  -e "interface_name=inside"

# Custom output directories
ansible-playbook site.yml -e "report_output_dir=/tmp/reports"
```

## Troubleshooting and Maintenance

### Common Issues
1. **Connection Failures**: Check network connectivity and firewall rules
2. **Authentication Errors**: Verify credentials and user permissions
3. **API Rate Limiting**: Adjust task delays and retry mechanisms
4. **Certificate Issues**: Validate SSL certificates or disable verification

### Logging and Debugging
- Enable verbose output: `ansible-playbook site.yml -vvv`
- Check generated logs in the configured log directory
- Review individual role task outputs for specific failures
- Use check mode to validate configuration before execution

### Performance Optimization
- Run roles selectively using tags for faster execution
- Disable unnecessary roles using variable overrides
- Parallelize execution across multiple FMC instances
- Optimize API call frequency and batch sizes

## Contributing and Extensions

### Adding New Assessments
1. Create new tasks within existing roles
2. Follow read-only operation principles
3. Implement proper error handling and logging
4. Add appropriate tags and conditional execution
5. Update documentation and examples

### Custom Validation Rules
1. Extend existing validation frameworks
2. Add custom compliance checks
3. Implement organization-specific security standards
4. Create custom report templates
5. Add new severity levels or categories

## Summary

This FMC/FTD Ansible automation suite provides a comprehensive, production-safe toolkit for security assessment and compliance validation. With 8 specialized roles covering over 3,900 lines of automation logic, it offers thorough assessment capabilities while maintaining strict read-only operations to ensure zero risk in production environments.

The modular architecture allows for flexible execution, comprehensive reporting, and easy extension to meet specific organizational requirements. Whether used for routine compliance checking, security auditing, or troubleshooting network connectivity issues, this suite provides the foundation for automated network security assessment at enterprise scale.

**Key Benefits:**
- **Zero Risk**: Strictly read-only operations ensure production safety
- **Comprehensive Coverage**: From basic connectivity to advanced security hardening
- **Flexible Execution**: Tag-based and conditional execution for specific use cases
- **Detailed Reporting**: Structured findings with actionable recommendations
- **Enterprise Ready**: Supports large-scale deployments with multiple FMC instances

