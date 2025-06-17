# FMC Ansible Automation Project

A comprehensive Ansible automation suite for Cisco Firepower Management Center (FMC) that provides security hardening, rule management, packet tracing, and connection testing capabilities.

## Features

- **Security Hardening**: Implements H-Cloud Security Hardening Benchmark requirements
- **Rule Management**: Time-based rule expiration and cleanup
- **Packet Tracing**: Network troubleshooting and path analysis
- **Connection Testing**: FMC connectivity validation
- **Compliance Reporting**: Detailed security compliance reports
- **DevNet Sandbox Ready**: Pre-configured for Cisco DevNet sandbox environment

## Requirements

- **Ansible**: 2.18+
- **Python**: 3.6+
- **Collections**: 
  - `cisco.fmcansible` (>=1.0.2)
  - `ansible.posix` (>=2.0.0)
  - `community.general` (>=10.7.0)
  - `ansible.utils` (>=6.0.0)

## Installation

1. **Clone the repository**:
   ```bash
   git clone https://github.com/LoneExile/fmc-ansible.git
   cd fmc-ansible
   ```

2. **Install required collections**:
   ```bash
   ansible-galaxy install -r requirements.yml
   ```

3. **Install Python dependencies** (if any):
   ```bash
   pip install -r requirements.txt
   ```

4. **Configure inventory**:
   - Edit `inventory/hosts` with your FMC details
   - Update credentials (consider using Ansible Vault for production)

## How to Run

### Quick Start (DevNet Sandbox)

The project is pre-configured for Cisco DevNet FMC sandbox:

```bash
# Run complete automation suite
ansible-playbook site.yml

# Run specific components with tags
ansible-playbook site.yml --tags connection
ansible-playbook site.yml --tags hardening
ansible-playbook site.yml --tags rules
ansible-playbook site.yml --tags packet
```

### Custom Environment

1. **Update inventory** (`inventory/hosts`):
   ```ini
   [fmc_servers]
   your-fmc ansible_host=your-fmc-ip

   [fmc_servers:vars]
   ansible_connection=httpapi
   ansible_network_os=cisco.fmcansible.fmc
   ansible_user=your-username
   ansible_password=your-password
   fmc_host={{ ansible_host }}
   ```

2. **Run with custom variables**:
   ```bash
   ansible-playbook site.yml -e "fmc_host=your-fmc-ip" -e "fmc_username=admin" -e "fmc_password=password"
   ```

### Advanced Usage

**Security Hardening Only**:
```bash
ansible-playbook site.yml --tags hardening -e "apply_hardening=true"
```

**Packet Tracing**:
```bash
ansible-playbook site.yml --tags packet -e "run_packet_tracer=true" -e "source_ip=192.168.1.10" -e "dest_ip=8.8.8.8"
```

**Rule Management**:
```bash
ansible-playbook site.yml --tags rules -e "rule_auto_disable=true" -e "rule_expiry_check_enabled=true"
```

## Project Structure

```
fmc-ansible/
├── site.yml                    # Main entry point
├── ansible.cfg                 # Ansible configuration
├── requirements.yml             # Collection dependencies
├── inventory/
│   └── hosts                   # Inventory file
├── playbooks/
│   ├── roles/main.yml          # Role orchestration playbook
│   └── test.yml                # Test playbook
├── roles/                      # Ansible roles
│   ├── fmc_common/             # Common functionality
│   ├── fmc_connection/         # Connection testing
│   ├── fmc_hardening/          # Security hardening
│   ├── fmc_packet_tracer/      # Packet tracing
│   └── fmc_rule_management/    # Rule management
├── docs/                       # Documentation
└── outputs/                    # Generated reports and logs
```

## Roles Overview

### 1. `fmc_common`
**Purpose**: Shared functionality across all roles

**What it does**:
- Sets up timestamp variables
- Creates output directories (configs, traces, reports, logs)
- Validates FMC connection parameters
- Sets default values for FMC protocol and domain UUID
- Provides common error handling

**Key Variables**:
- `config_output_dir`: Configuration output directory
- `trace_output_dir`: Packet trace output directory
- `report_output_dir`: Report output directory
- `log_output_dir`: Log output directory

### 2. `fmc_connection`
**Purpose**: Tests and validates FMC connectivity

**What it does**:
- Tests FMC server version access
- Validates domain access permissions
- Checks access policy permissions
- Provides comprehensive connection diagnostics
- Sets connection status variables for other roles

**Key Features**:
- Server version retrieval
- Domain enumeration
- Access policy validation
- Connection health summary
- Failure diagnostics

**Output**: Connection test results and status variables

### 3. `fmc_hardening`
**Purpose**: Implements H-Cloud Security Hardening Benchmark

**What it does**:
- **Hostname Validation**: Checks naming convention compliance
- **Warning Banner**: Configures legal disclaimer banners
- **Access Control Lists**: Restricts management access to approved IPs
- **Session Timeouts**: Configures browser and shell timeouts (15 minutes)
- **Password Policy**: Enforces strong password requirements (15+ chars, 2+ categories)
- **Health Monitoring**: Sets up system monitoring and audit logging
- **Compliance Reporting**: Generates detailed compliance reports

**Key Variables**:
- `apply_hardening`: Apply configurations (default: false, validation only)
- `validate_compliance`: Fail on critical compliance issues
- `browser_session_timeout`: Browser timeout in minutes (default: 15)
- `password_min_length`: Minimum password length (default: 15)

**Security Controls**:
- ✅ 3.1.1 Hostname Configuration
- ✅ 3.1.2 Warning Banner
- ✅ 3.3 Management Access Control List
- ✅ Session Timeout Configuration
- ✅ Password Policy Configuration
- ✅ Health Monitoring
- ✅ Compliance Reporting

**Output**: 
- JSON compliance reports
- Configuration recommendations
- Security audit logs

### 4. `fmc_packet_tracer`
**Purpose**: Network troubleshooting and path analysis

**What it does**:
- Executes packet traces through FMC-managed devices
- Analyzes network paths and policy decisions
- Supports multiple protocols (TCP, UDP, ICMP)
- Provides detailed trace results and logging
- Handles interface selection and validation

**Key Variables**:
- `source_ip`: Source IP address (required)
- `dest_ip`: Destination IP address (required)
- `source_port`: Source port (default: 80)
- `dest_port`: Destination port (default: 80)
- `protocol`: Protocol (default: TCP)
- `device_id`: Target device ID (auto-selected if not specified)
- `interface_id`: Specific interface ID (required)
- `interface_name`: Specific interface name (required)
- `interface_type`: Specific interface type (required)

**Features**:
- Automatic device selection
- Interface object enumeration
- Protocol support (TCP/UDP/ICMP)
- Detailed trace logging
- JSON and text output formats

**Output**:
- Packet trace results (JSON)
- Detailed trace logs
- Network path analysis
- Policy decision information

### 5. `fmc_rule_management`
**Purpose**: Time-based rule lifecycle management

**What it does**:
- Scans access policies for expired rules
- Identifies rules with expiry metadata or date-based naming
- Automatically disables expired rules (optional)
- Automatically deletes expired rules (optional)
- Creates rule backups before modifications
- Generates rule management reports

**Key Variables**:
- `rule_expiry_check_enabled`: Enable expiry checking (default: true)
- `rule_auto_disable`: Auto-disable expired rules (default: false)
- `rule_auto_delete`: Auto-delete expired rules (default: false)
- `rule_backup_before_delete`: Backup rules before deletion (default: true)

**Expiry Detection Methods**:
1. **Metadata-based**: Rules with `metadata.expiry_date` field
2. **Name-based**: Rules with YYYY-MM-DD format in name

**Features**:
- Multi-policy rule scanning
- Flexible expiry date detection
- Safe rule backup before changes
- Comprehensive activity logging
- Detailed management reports

**Output**:
- Rule management reports (JSON)
- Rule backups (JSON)
- Activity logs
- Expiry analysis

## Output Files

All outputs are organized in the `outputs/` directory:

```
outputs/
├── configs/                    # Configuration files
│   ├── fmc_compliance_report_*.json
│   └── rules_backup_*.json
├── packet_traces/              # Packet trace results
│   ├── packet_trace_*.json
│   └── packet_trace_detailed_*.log
├── reports/                    # Management reports
│   └── rule_management_*.json
└── logs/                       # Activity logs
    ├── hardening_process.log
    ├── compliance_audit.log
    ├── packet_tracer.log
    └── rule_management.log
```

## Configuration Variables

### Global Variables
```yaml
# FMC Connection
fmc_host: "your-fmc-host"
fmc_username: "admin"
fmc_password: "password"
fmc_protocol: "https"
fmc_port: 443
domain_uuid: "auto-detected"

# Feature Toggles
extract_hardening_config: true
manage_rules: true
run_packet_tracer: false
debug_mode: false

# Output Directories
config_output_dir: "./outputs/configs"
trace_output_dir: "./outputs/packet_traces"
report_output_dir: "./outputs/reports"
log_output_dir: "./outputs/logs"
```

### Role-Specific Variables

**FMC Hardening**:
```yaml
apply_hardening: false              # Apply configurations
validate_compliance: true          # Fail on critical issues
compliance_report_enabled: true    # Generate reports
browser_session_timeout: 15        # Minutes
password_min_length: 15           # Characters
```

**Packet Tracer**:
```yaml
source_ip: "192.168.1.10"         # Required
dest_ip: "8.8.8.8"                # Required
protocol: "TCP"                    # TCP/UDP/ICMP
source_port: 80                    # Default port
dest_port: 80                      # Default port
interface_id: ""                   # Specify interface by ID
interface_name: ""                 # Specify interface by name
interface_type: ""                 # Specify interface type (e.g., "physical", "logical")
```

**Rule Management**:
```yaml
rule_expiry_check_enabled: true    # Enable expiry checking
rule_auto_disable: false           # Auto-disable expired rules
rule_auto_delete: false            # Auto-delete expired rules
rule_backup_before_delete: true    # Backup before deletion
```

## Tags

Use tags to run specific components:

- `always`: Always executed tasks
- `common`: Common setup tasks
- `connection`: Connection testing
- `hardening`: Security hardening
- `config`: Configuration extraction
- `rules`: Rule management
- `management`: Rule management
- `packet`: Packet tracing
- `tracer`: Packet tracing

## Security Considerations

1. **Credentials**: Use Ansible Vault for production credentials
2. **Network Access**: Ensure proper network connectivity to FMC
3. **Permissions**: FMC user needs administrative privileges
4. **Compliance**: Review hardening configurations before applying
5. **Backups**: Always backup configurations before changes

## Troubleshooting

### Common Issues

1. **Connection Failures**:
   ```bash
   # Test connection only
   ansible-playbook site.yml --tags connection -v
   ```

2. **Authentication Issues**:
   - Verify credentials in inventory
   - Check FMC user permissions
   - Validate network connectivity

3. **API Timeouts**:
   - Increase timeout values in `ansible.cfg`
   - Check FMC system load

4. **Missing Dependencies**:
   ```bash
   ansible-galaxy install -r requirements.yml --force
   ```

### Debug Mode

Enable verbose output:
```bash
ansible-playbook site.yml -e "debug_mode=true" -vvv
```

## Examples

### Basic Security Assessment
```bash
# Run hardening validation only
ansible-playbook site.yml --tags hardening
```

### Apply Security Hardening
```bash
# Apply hardening configurations
ansible-playbook site.yml --tags hardening -e "apply_hardening=true"
```

### Network Troubleshooting
```bash
# Trace packet path
ansible-playbook site.yml --tags packet \
  -e "run_packet_tracer=true" \
  -e "source_ip=192.168.1.10" \
  -e "dest_ip=8.8.8.8" \
  -e "protocol=TCP" \
  -e "dest_port=443 " \

```

### Rule Cleanup
```bash
# Check for expired rules
ansible-playbook site.yml --tags rules

# Auto-disable expired rules
ansible-playbook site.yml --tags rules -e "rule_auto_disable=true"
```

## Support

- **DevNet Sandbox**: https://devnetsandbox.cisco.com/DevNet
- **FMC API Documentation**: Check `docs/` directory
- [FMC Ansible Collection](https://github.com/CiscoDevNet/FMCAnsible)

## TODO

- [x] Refactor to use `cisco.fmcansible` collection
- [x] Implement Security Controls FMC plan
- [x] Add comprehensive role documentation
<!-- - [ ] Add unit tests for roles -->
<!-- - [ ] Implement CI/CD pipeline -->
<!-- - [ ] Add more packet tracer scenarios -->
<!-- - [ ] Enhance compliance reporting -->

