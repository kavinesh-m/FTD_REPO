# FMC Ansible Automation Project

https://devnetsandbox.cisco.com/DevNet

## TODO

- [ ] refactor to use `ansible.cisco.fmcansible`
- [ ] Implemente Security Controls FMC plan
    - 3.1.1 Hostname Configuration
        - Validates hostname format against H-Cloud naming convention
        - Provides compliance reporting and manual configuration guidance

    -  3.1.2 Warning Banner
        - Configures standard login banner with legal disclaimer
        - Supports both validation and automatic configuration

    - 3.3 Management Access Control List
        - Restricts SSH/HTTPS access to H-Cloud approved IP ranges only
        - Comprehensive IP allowlist validation and configuration

    - Session Timeout Configuration
        - Browser session timeout: 15 minutes
        - Shell timeout: 15 minutes

    - Password Policy Configuration
        - Minimum 15 characters
        - Requires 2+ character categories
        - Local user account analysis

    - Health Monitoring
        - System health monitoring setup
        - Audit logging configuration
        - Service status validation

    - Compliance Reporting
        - Comprehensive JSON and HTML reports
        - Compliance percentage calculation
        - Critical failure identification
        - Detailed recommendations

