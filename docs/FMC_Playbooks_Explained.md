# FMC Ansible Playbooks Explained

## What is FMC (Firewall Management Center)?

**FMC (Firewall Management Center)** is Cisco's centralized management platform for their Firepower security devices. Think of it as the "control tower" for your network security infrastructure. It allows network administrators to:

- Configure firewall rules and policies
- Monitor network traffic and threats
- Manage multiple firewall devices from one location
- Generate security reports and analytics
- Troubleshoot network connectivity issues

## Overview of the Automation Suite

This Ansible project provides automated tools to manage and monitor your FMC system. Instead of manually clicking through the FMC web interface, these playbooks use API calls to perform tasks automatically, saving time and reducing human error.

### 🏗️ **Architecture Overview**
The automation suite follows a modular design with:
- **Master Orchestrator** (`main.yml`) - Coordinates all automation tasks
- **Individual Playbooks** - Specialized tools for specific functions
- **Centralized Configuration** - All settings managed in `group_vars/all.yml`
- **Organized Output Structure** - Separate directories for configs, reports, logs, and traces

**Note:** The main.yml playbook is designed to use modular task files (e.g., `test_connection_tasks.yml`, `hardening_extraction_tasks.yml`, etc.) for better organization. If these task files don't exist yet, you can run the individual playbooks directly or create the task files by extracting the relevant tasks from each individual playbook.

---

## 📋 Playbook Breakdown

### 1. **main.yml** - The Master Orchestrator
**What it does:** This is the "conductor" that runs all other playbooks in a coordinated sequence using a modular task-based approach.

**Think of it like:** A daily maintenance checklist that automatically runs through all your security tasks with intelligent conditional execution.

**Key Features:**
- **Smart Orchestration:** Uses include_tasks to modularize operations
- **Conditional Execution:** Only runs tasks when needed (configurable via variables)
- **Organized Output:** Creates structured directories for all reports and logs
- **Comprehensive Banner:** Displays clear status and task overview
- **Master Summary:** Generates consolidated HTML report of all activities
- **Error Handling:** Graceful handling of optional tasks and dependencies

**Execution Flow:**
1. **Setup Phase:** Creates output directories and displays execution banner
2. **Connection Test:** Validates FMC connectivity and authentication
3. **Configuration Backup:** Extracts hardening configs (if enabled)
4. **Rule Management:** Manages expired rules (if enabled and configured)
5. **Packet Tracing:** Runs network diagnostics (if parameters provided)
6. **Reporting:** Generates master summary report and final status

**Configuration Options:**
- `extract_hardening_config`: Enable/disable configuration extraction
- `manage_rules`: Enable/disable rule management
- `rule_expiry_check_enabled`: Control rule expiry checking
- `run_packet_tracer`: Enable/disable packet tracing
- `source_ip` & `dest_ip`: Required for packet tracing

**When to use:** Run this for complete daily/weekly automation of your FMC maintenance tasks.

---

### 2. **test_connection.yml** - The Health Check
**What it does:** Performs comprehensive connectivity and authentication testing to verify that your automation can successfully communicate with the FMC system.

**Think of it like:** A doctor performing a complete physical examination before treatment.

**Enhanced Testing Features:**
- **Multi-Stage Validation:** Progressive testing from basic connectivity to full API access
- **Detailed Status Reporting:** Clear PASS/FAIL status for each test component
- **Intelligent Error Handling:** Distinguishes between different types of failures
- **Comprehensive Summary:** Overall readiness assessment for automation

**What it tests:**
- **Basic Connectivity:** Can we reach the FMC server? (Expects 401/200 status)
- **Authentication:** Are our username/password correct? (Expects 204 status)
- **API Access:** Can we successfully make authenticated API calls?
- **Domain Access:** Can we access the specific security domain and policies?
- **Server Information:** Retrieves FMC version and build details

**Enhanced Output Features:**
- Clear status indicators for each test phase
- FMC version and build information display
- Domain UUID validation
- Access policy count verification
- Overall readiness assessment

**Safety Features:**
- Non-destructive testing (read-only operations)
- Graceful failure handling
- Detailed error reporting
- Automatic retry logic with configurable timeouts

**When to use:** 
- Before running any other automation
- When troubleshooting connection issues
- As part of regular health monitoring
- After FMC system updates or changes

---

### 3. **hardening_extraction.yml** - The Configuration Backup
**What it does:** Extracts and saves all important security configurations from your FMC with enhanced error handling and comprehensive data collection.

**Think of it like:** Creating a detailed backup of all your security settings and policies with intelligent fallback mechanisms.

**Enhanced Features:**
- **Comprehensive Data Collection:** Extracts multiple configuration types in a single run
- **Intelligent Error Handling:** Gracefully handles missing or inaccessible configuration types
- **Structured Output:** Organizes data with timestamps and metadata for easy tracking
- **Retry Logic:** Automatic retry with configurable timeouts for reliable extraction

**What it extracts:**
- **System Information:** FMC version, build details, and server information
- **Access Policies:** Rules that control what traffic is allowed/blocked
- **Network Policies:** Advanced threat detection settings (with fallback to prefilter policies)
- **Device Information:** Details about connected firewalls and their status
- **Network Objects:** Defined IP addresses, subnets, and ranges
- **Port Objects:** Defined services and port configurations

**Enhanced Output Features:**
- **Timestamped JSON Files:** Complete configuration backup with extraction metadata
- **Detailed Logging:** Comprehensive extraction summary with counts and status
- **Structured Data:** Organized JSON format for easy parsing and analysis
- **Error Resilience:** Continues extraction even if some components fail

**Safety Features:**
- **Read-only Operations:** No modifications made to FMC configuration
- **Comprehensive Logging:** Detailed activity logs for audit trails
- **Graceful Degradation:** Handles missing permissions or unavailable features
- **Data Validation:** Ensures extracted data integrity

**When to use:**
- Before making any changes to FMC
- For compliance documentation and audit preparation
- Regular backup schedules (daily/weekly)
- Before system upgrades or maintenance
- Emergency configuration preservation

---

### 4. **rule_management.yml** - The Automatic Housekeeper
**What it does:** Automatically finds and manages expired firewall rules to keep your security policies clean and current with enhanced processing and comprehensive tracking.

**Think of it like:** A smart assistant that removes outdated calendar appointments automatically with detailed audit trails.

**Enhanced Features:**
- **Comprehensive Rule Discovery:** Scans all access policies and their rules systematically
- **Dual Expiry Detection:** Uses both metadata and name-based expiry identification
- **Intelligent Processing:** Handles large rule sets efficiently with proper error handling
- **Detailed Tracking:** Maintains comprehensive logs of all rule operations

**How it identifies expired rules:**
1. **Metadata Method:** Rules with built-in expiry dates in metadata.expiry_date field
2. **Name-based Method:** Rules with dates in their names (e.g., "Temp_Access_2024-01-15")

**What it can do:**
- **Find:** Locate all expired rules across all policies with detailed analysis
- **Backup:** Save copies of rules before making changes with timestamps
- **Disable:** Turn off expired rules (safer option) with status tracking
- **Delete:** Permanently remove expired rules (more aggressive) with confirmation

**Enhanced Safety Features:**
- **Automatic Backups:** Always creates timestamped backups before making changes
- **Configurable Actions:** Choose between disable vs. delete operations
- **Detailed Logging:** Comprehensive activity logs with rule counts and status
- **Error Resilience:** Continues processing even if individual rule operations fail
- **Status Tracking:** Monitors success/failure of each rule modification

**Processing Intelligence:**
- **Batch Processing:** Efficiently handles multiple policies and rules
- **Conditional Execution:** Only processes rules when expiry checking is enabled
- **Smart Filtering:** Identifies expired rules using multiple detection methods
- **Comprehensive Reporting:** Generates detailed JSON reports with full rule details

**When to use:**
- Daily/weekly cleanup of temporary rules
- Compliance maintenance and audit preparation
- Before security audits to ensure clean rule sets
- When firewall performance is degrading due to too many rules
- Regular housekeeping to maintain optimal policy efficiency

---

### 5. **packet_tracer.yml** - The Network Detective
**What it does:** Simulates network traffic to troubleshoot connectivity issues and understand how traffic flows through your firewall with enhanced error handling and multiple API endpoint support.

**Think of it like:** A GPS route planner that shows exactly how data travels from point A to point B through your network security.

**Enhanced Features:**
- **Multiple API Endpoint Support:** Tries different packet tracer API endpoints for maximum compatibility
- **Intelligent Device Selection:** Automatically selects available devices for tracing
- **Comprehensive Parameter Validation:** Ensures all required parameters are provided
- **Detailed Error Handling:** Gracefully handles API limitations and unavailable features
- **Structured Output:** Organized JSON reports with complete trace metadata

**What you provide:**
- **Source IP:** Where the traffic starts (e.g., 192.168.1.100)
- **Destination IP:** Where the traffic is going (e.g., 8.8.8.8)
- **Ports:** What service/application (e.g., port 80 for web, 443 for HTTPS)
- **Protocol:** Type of traffic (TCP, UDP, ICMP)

**What it tells you:**
- **Path:** Exact route traffic takes through your firewall
- **Rules Applied:** Which security rules affect this traffic
- **Allow/Block Decision:** Whether traffic would be permitted
- **Performance Impact:** How long processing takes
- **Troubleshooting Info:** Why traffic might be blocked

**Enhanced Output Features:**
- **Detailed JSON Report:** Complete trace results with metadata and timestamps
- **Human-readable Log:** Comprehensive trace execution details
- **Error Resilience:** Handles DevNet sandbox limitations and API unavailability
- **Device Information:** Shows which firewall device performed the trace

**API Endpoint Intelligence:**
- **Primary Endpoint:** `/api/fmc_troubleshoot/v1/domain/{domain}/packettracer`
- **Alternative Endpoint:** `/api/fmc_config/v1/domain/{domain}/object/packettracer`
- **Third Endpoint:** `/api/firepower/device-management/v1/packet-tracer`
- **Fallback Handling:** Graceful degradation when packet tracer is unavailable

**When to use:**
- Troubleshooting "why can't I access this website/service?"
- Validating new firewall rules work correctly
- Understanding traffic flow for compliance
- Performance analysis of security policies
- Network connectivity diagnostics

---

## 🔄 How They Work Together

### Typical Automation Flow:
1. **Test Connection** → Ensure everything is working
2. **Extract Hardening Config** → Backup current state
3. **Manage Rules** → Clean up expired policies
4. **Packet Tracer** → Verify critical paths still work
5. **Generate Reports** → Document everything that happened

### Real-World Example:
```
Daily 2 AM Automation:
├── Check FMC is accessible ✓
├── Backup all current configurations ✓
├── Find and disable 3 expired temporary rules ✓
├── Test that critical business applications still work ✓
└── Email summary report to security team ✓
```

---

## 🎯 Benefits for Non-Technical Users

### **Consistency**
- Same checks performed every time
- No human errors or forgotten steps
- Standardized reporting format

### **Efficiency** 
- Tasks that take hours manually complete in minutes
- Can run during off-hours
- Frees up security team for strategic work

### **Compliance**
- Automatic documentation of all changes
- Audit trail of security policy maintenance
- Regular backup of configurations

### **Proactive Management**
- Catches issues before they become problems
- Maintains clean, efficient security policies
- Provides data for capacity planning

---

## 🚨 Safety Features

### **Backup First**
- Always saves current state before making changes
- Can restore if something goes wrong

### **Configurable Actions**
- Can run in "report only" mode
- Choose between disable vs. delete for rules
- Adjustable timeouts and retry logic

### **Detailed Logging**
- Every action is recorded with timestamps
- Easy to track what happened when
- Helps with troubleshooting

### **Validation**
- Tests connections before making changes
- Verifies credentials and permissions
- Confirms API access is working

---

## 📊 Understanding the Reports

### **Connection Test Report**
- Green checkmarks = Everything working
- Red X's = Need attention
- Shows exactly what's broken

### **Configuration Backup Report**
- Lists how many policies, rules, objects were saved
- Confirms backup was successful
- Shows file locations

### **Rule Management Report**
- Number of rules checked
- How many were expired
- What actions were taken
- Details of each rule affected

### **Packet Tracer Report**
- Visual representation of traffic flow
- Shows allow/block decisions
- Identifies bottlenecks or issues

### **Master Summary Report**
- Combined overview of all activities
- Executive summary format
- Perfect for management reporting

---

## 🎯 When to Use Each Playbook

| Scenario | Recommended Playbook | Frequency |
|----------|---------------------|-----------|
| Daily health check | `test_connection.yml` | Daily |
| Before system changes | `hardening_extraction.yml` | Before changes |
| Regular maintenance | `rule_management.yml` | Weekly |
| Troubleshooting connectivity | `packet_tracer.yml` | As needed |
| Complete automation | `main.yml` | Daily/Weekly |
| Emergency backup | `hardening_extraction.yml` | Immediately |

---

## 💡 Getting Started Tips

1. **Start Small:** Begin with `test_connection.yml` to ensure everything works
2. **Backup First:** Always run `hardening_extraction.yml` before making changes
3. **Test Mode:** Use report-only mode for `rule_management.yml` initially
4. **Monitor Logs:** Check the output files to understand what's happening
5. **Schedule Wisely:** Run automation during low-traffic periods

This automation suite transforms complex, time-consuming FMC management tasks into simple, reliable, automated processes that improve security while reducing administrative overhead.

