# Bug-detector
KevinHarryOx Bug Detector is a comprehensive web vulnerability scanning tool designed for security professionals and ethical hackers. It performs automated checks for SQL Injection, XSS, exposed directories, security headers misconfigurations, open ports, and information disclosure vulnerabilities.
# 🐛 KevinHarryOx Bug Detector

**Advanced Web Vulnerability Scanner for Authorized Security Testing**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Kali Linux](https://img.shields.io/badge/Kali%20Linux-Compatible-blue)](https://www.kali.org/)
[![Bash](https://img.shields.io/badge/Shell-Bash-green)](https://www.gnu.org/software/bash/)

## ⚠️ Legal Disclaimer

**This tool is for EDUCATIONAL and AUTHORIZED testing purposes ONLY.**
- Only scan websites you OWN or have EXPLICIT WRITTEN PERMISSION to test
- Unauthorized scanning violates laws including CFAA, GDPR, and Computer Misuse Act
- The author assumes NO liability for misuse of this tool
- ALWAYS obtain proper authorization before security testing

## 🎯 Features

### 🔍 Comprehensive Vulnerability Scanning
- **SQL Injection (SQLi)** - Error-based and time-based detection
- **Cross-Site Scripting (XSS)** - Reflected XSS with payload testing
- **Directory Enumeration** - Find exposed directories and listing vulnerabilities
- **Security Headers** - Check for HSTS, CSP, X-Frame-Options, X-Content-Type-Options
- **Port Scanning** - Detect open ports (80,443,22,3306,8080, etc.)
- **File Exposure** - Find sensitive files (.env, config.php, .htaccess, backups)
- **Information Disclosure** - Email addresses, server info, developer comments
- **SSL/TLS Analysis** - Certificate information and security issues

### 📊 Output Features
- 🎨 **Color-coded output** - Easy-to-read vulnerability reports
- 📈 **Severity levels** - HIGH, MEDIUM, LOW classification
- 📝 **Auto-generated reports** - Save scan results to timestamped files
- 🔔 **Real-time alerts** - Immediate notification of critical findings

## 🛠️ Requirements

- **Kali Linux** (Recommended) or any Debian-based Linux distribution
- **Bash 4.0+**
- **Root/sudo privileges** (for port scanning and some features)

### Automated Dependencies
The script automatically installs required tools:
- `curl` - HTTP requests
- `nmap` - Port scanning
- `openssl` - SSL/TLS checks

## 📦 Installation

### Quick Install
```bash
# Clone the repository
git clone https://github.com/kevinharryox/bug-detector.git
cd bug-detector

# Make executable
chmod +x bug-detector.sh

# Run
./bug-detector.sh

Quick Commands:
# Clone and run in one line
git clone https://github.com/kevinharryox/bug-detector.git && cd bug-detector && chmod +x bug-detector.sh && sudo ./bug-detector.sh

# Create alias for quick access
alias bugscan='sudo /path/to/bug-detector.sh'

# Run with specific target (modified version)
echo "https://target.com" | ./bug-detector.sh

## Tags for GitHub Repository

```yaml
topics:
  - security-scanner
  - vulnerability-scanner
  - web-security
  - penetration-testing
  - ethical-hacking
  - sql-injection
  - xss-detection
  - kali-linux
  - bug-bounty
  - cybersecurity
  bug-detector/
├── bug-detector.sh          # Main script
├── README.md                # Documentation
├── LICENSE                  # MIT License
├── CHANGELOG.md            # Version history
├── CONTRIBUTING.md         # Contribution guidelines
├── examples/               # Example outputs
│   └── sample_report.txt
└── docs/                   # Additional documentation
    ├── installation.md
    └── troubleshooting.md
