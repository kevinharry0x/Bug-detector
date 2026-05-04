#!/bin/bash

# ============================================
# KevinHarryOx Bug Detector v2.0
# Advanced Web Vulnerability Scanner
# For authorized security testing only
# ============================================

# Color definitions
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
PURPLE='\033[0;35m'
CYAN='\033[0;36m'
WHITE='\033[1;37m'
NC='\033[0m' # No Color

# Clear screen and display banner
clear
echo -e "${CYAN}================================================${NC}"
echo -e "${RED}          KevinHarryOx Bug Detector${NC}"
echo -e "${CYAN}================================================${NC}"
echo -e "${WHITE}Advanced Web Vulnerability Scanner${NC}"
echo -e "${CYAN}================================================${NC}\n"

# Function to check if required tools are installed
check_tools() {
    local tools=("curl" "nmap")
    local missing=()
    
    for tool in "${tools[@]}"; do
        if ! command -v "$tool" &> /dev/null; then
            missing+=("$tool")
        fi
    done
    
    if [ ${#missing[@]} -ne 0 ]; then
        echo -e "${YELLOW}Warning: The following tools are missing: ${missing[*]}${NC}"
        echo -e "${YELLOW}Installing missing tools...${NC}"
        sudo apt-get update -qq
        sudo apt-get install -y "${missing[@]}" 2>/dev/null
        echo -e "${GREEN}Installation complete.${NC}\n"
    fi
}

# Function to validate URL
validate_url() {
    if [[ $1 =~ ^https?://[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}(/.*)?$ ]]; then
        return 0
    else
        return 1
    fi
}

# Function to scan for SQL Injection
sql_injection_scan() {
    echo -e "${BLUE}[*] Running SQL Injection scan...${NC}"
    
    # Test payloads for SQL injection
    local payloads=("'" "\"" "1' OR '1'='1" "1' AND '1'='1" "' OR '1'='1'--" "admin'--")
    local found=0
    
    for payload in "${payloads[@]}"; do
        # Test different parameters
        test_url="${1}?id=${payload}"
        response=$(curl -s -o /dev/null -w "%{http_code}" --max-time 10 "$test_url" 2>/dev/null)
        
        # Check for SQL error patterns
        error_check=$(curl -s --max-time 10 "$test_url" 2>/dev/null | grep -iE "sql|syntax|mysql|oracle|postgres|database error" | wc -l)
        
        if [ "$error_check" -gt 0 ]; then
            echo -e "${RED}[-] SQL Injection vulnerability detected${NC}"
            echo -e "${YELLOW}[i] Type: SQL Injection${NC}"
            echo -e "${YELLOW}[i] Location: URL parameters (id parameter)${NC}"
            echo -e "${YELLOW}[i] Severity: HIGH${NC}"
            echo -e "${YELLOW}[i] Payload: $payload${NC}\n"
            found=1
            break
        fi
    done
    
    if [ $found -eq 0 ]; then
        echo -e "${GREEN}[+] No SQL Injection vulnerabilities detected${NC}\n"
    fi
}

# Function to scan for XSS vulnerabilities
xss_scan() {
    echo -e "${BLUE}[*] Running XSS scan...${NC}"
    
    # XSS payloads
    local payloads=("<script>alert('XSS')</script>" 
                    "<img src=x onerror=alert('XSS')>" 
                    "javascript:alert('XSS')" 
                    "'><script>alert('XSS')</script>" 
                    "\"><script>alert('XSS')</script>")
    local found=0
    
    for payload in "${payloads[@]}"; do
        test_url="${1}?search=${payload}"
        response=$(curl -s --max-time 10 "$test_url" 2>/dev/null)
        
        # Check if payload is reflected
        if echo "$response" | grep -q "$(echo "$payload" | sed 's/[][(){}]/\\&/g')"; then
            echo -e "${RED}[-] XSS vulnerability detected${NC}"
            echo -e "${YELLOW}[i] Type: Cross-Site Scripting (XSS)${NC}"
            echo -e "${YELLOW}[i] Location: Search parameter or input fields${NC}"
            echo -e "${YELLOW}[i] Severity: HIGH${NC}"
            echo -e "${YELLOW}[i] Payload: $payload${NC}\n"
            found=1
            break
        fi
    done
    
    if [ $found -eq 0 ]; then
        echo -e "${GREEN}[+] No XSS vulnerabilities detected${NC}\n"
    fi
}

# Function to scan for directory listing
directory_scan() {
    echo -e "${BLUE}[*] Running directory enumeration scan...${NC}"
    
    # Common directories to check
    local directories=("/admin" "/backup" "/temp" "/logs" "/uploads" "/images" "/css" "/js" "/config" "/include")
    local found=0
    
    base_url=$(echo "$1" | sed 's/\/$//')
    
    for dir in "${directories[@]}"; do
        url="$base_url$dir/"
        response=$(curl -s --max-time 10 "$url" 2>/dev/null)
        
        if echo "$response" | grep -qiE "index of|directory listing|parent directory|name.*last modified"; then
            severity="MEDIUM"
            if [[ $dir == *"admin"* || $dir == *"config"* ]]; then
                severity="HIGH"
            elif [[ $dir == *"backup"* || $dir == *"logs"* ]]; then
                severity="MEDIUM"
            else
                severity="LOW"
            fi
            
            echo -e "${RED}[-] Exposed directory: $dir/${NC}"
            echo -e "${YELLOW}[i] Type: Information Disclosure${NC}"
            echo -e "${YELLOW}[i] Location: $url${NC}"
            echo -e "${YELLOW}[i] Severity: $severity${NC}\n"
            found=1
        fi
    done
    
    if [ $found -eq 0 ]; then
        echo -e "${GREEN}[+] No exposed directories found${NC}\n"
    fi
}

# Function to scan for SSL/TLS issues
ssl_scan() {
    if [[ $1 == https://* ]]; then
        echo -e "${BLUE}[*] Running SSL/TLS scan...${NC}"
        
        # Extract the domain from the URL
        domain=$(echo "$1" | sed 's/https\?:\/\///' | sed 's/\/.*//')
        
        # Check SSL/TLS using openssl
        ssl_info=$(echo | openssl s_client -connect "$domain:443" -servername "$domain" 2>/dev/null | openssl x509 -text 2>/dev/null | grep -E "Signature Algorithm|Not Before|Not After")
        
        if [ -n "$ssl_info" ]; then
            echo -e "${GREEN}[+] SSL/TLS Certificate Information:${NC}"
            echo "$ssl_info" | while read line; do
                echo -e "${CYAN}    $line${NC}"
            done
            echo ""
        else
            echo -e "${YELLOW}[!] SSL/TLS connection issues detected${NC}\n"
        fi
    fi
}

# Function to scan for security headers
security_headers_scan() {
    echo -e "${BLUE}[*] Running security headers scan...${NC}"
    
    # Get response headers
    response_headers=$(curl -s -I --max-time 10 "$1" 2>/dev/null)
    
    # Check for common security headers
    if echo "$response_headers" | grep -qi "X-Frame-Options"; then
        echo -e "${GREEN}[+] X-Frame-Options header present${NC}"
    else
        echo -e "${RED}[-] Missing X-Frame-Options header (Clickjacking Protection)${NC}"
        echo -e "${YELLOW}[i] Type: Security Header Missing${NC}"
        echo -e "${YELLOW}[i] Location: HTTP Response Headers${NC}"
        echo -e "${YELLOW}[i] Severity: MEDIUM${NC}\n"
    fi
    
    if echo "$response_headers" | grep -qi "X-Content-Type-Options.*nosniff"; then
        echo -e "${GREEN}[+] X-Content-Type-Options header present${NC}"
    else
        echo -e "${RED}[-] Missing X-Content-Type-Options header (MIME Sniffing Protection)${NC}"
        echo -e "${YELLOW}[i] Type: Security Header Missing${NC}"
        echo -e "${YELLOW}[i] Location: HTTP Response Headers${NC}"
        echo -e "${YELLOW}[i] Severity: MEDIUM${NC}\n"
    fi
    
    if echo "$response_headers" | grep -qi "Strict-Transport-Security"; then
        echo -e "${GREEN}[+] HSTS header present${NC}"
    else
        if [[ $1 == https://* ]]; then
            echo -e "${RED}[-] Missing HSTS header (HTTPS Enforced)${NC}"
            echo -e "${YELLOW}[i] Type: Security Header Missing${NC}"
            echo -e "${YELLOW}[i] Location: HTTP Response Headers${NC}"
            echo -e "${YELLOW}[i] Severity: MEDIUM${NC}\n"
        fi
    fi
    
    if echo "$response_headers" | grep -qi "Content-Security-Policy"; then
        echo -e "${GREEN}[+] CSP header present${NC}"
    else
        echo -e "${YELLOW}[!] Missing CSP header (XSS Protection) - LOW${NC}\n"
    fi
    
    echo ""
}

# Function to scan for common files
common_files_scan() {
    echo -e "${BLUE}[*] Running common files scan...${NC}"
    
    # List of common sensitive files
    local common_files=(".env" "config.php" "database.yml" "web.config" ".htaccess" "robots.txt" "phpinfo.php" "test.php" "admin.php" "wp-config.php" "backup.sql" "dump.sql")
    
    base_url=$(echo "$1" | sed 's/\/$//')
    local found=0
    
    for file in "${common_files[@]}"; do
        url="$base_url/$file"
        status_code=$(curl -s -o /dev/null -w "%{http_code}" --max-time 5 "$url" 2>/dev/null)
        
        if [ "$status_code" != "404" ] && [ "$status_code" != "000" ]; then
            severity="LOW"
            if [[ $file == *".env"* || $file == *"config"* || $file == *"database"* || $file == *"sql"* ]]; then
                severity="HIGH"
            elif [[ $file == *"admin"* || $file == *"phpinfo"* || $file == *"wp-config"* ]]; then
                severity="MEDIUM"
            fi
            
            echo -e "${RED}[-] Exposed sensitive file: $file${NC}"
            echo -e "${YELLOW}[i] Type: Information Disclosure${NC}"
            echo -e "${YELLOW}[i] Location: $url${NC}"
            echo -e "${YELLOW}[i] HTTP Status: $status_code${NC}"
            echo -e "${YELLOW}[i] Severity: $severity${NC}\n"
            found=1
        fi
    done
    
    if [ $found -eq 0 ]; then
        echo -e "${GREEN}[+] No exposed sensitive files found${NC}\n"
    fi
}

# Function to check for open ports
port_scan() {
    echo -e "${BLUE}[*] Running port scan...${NC}"
    
    domain=$(echo "$1" | sed 's/https\?:\/\///' | sed 's/\/.*//')
    local common_ports=("80" "443" "22" "21" "25" "3306" "5432" "8080" "8443")
    local found=0
    
    for port in "${common_ports[@]}"; do
        timeout 2 bash -c "echo > /dev/tcp/$domain/$port" 2>/dev/null
        if [ $? -eq 0 ]; then
            echo -e "${YELLOW}[!] Port $port is open${NC}"
            echo -e "${YELLOW}[i] Type: Network Service${NC}"
            echo -e "${YELLOW}[i] Location: $domain:$port${NC}"
            echo -e "${YELLOW}[i] Severity: MEDIUM${NC}\n"
            found=1
        fi
    done
    
    if [ $found -eq 0 ]; then
        echo -e "${GREEN}[+] No unnecessary open ports detected${NC}\n"
    fi
}

# Function to check for information disclosure
info_disclosure_scan() {
    echo -e "${BLUE}[*] Running information disclosure scan...${NC}"
    
    response=$(curl -s --max-time 10 "$1" 2>/dev/null)
    
    # Check for email addresses
    emails=$(echo "$response" | grep -oE '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}' | sort -u | head -5)
    if [ -n "$emails" ]; then
        echo -e "${YELLOW}[!] Email addresses found in source code${NC}"
        echo -e "${YELLOW}[i] Type: Information Disclosure${NC}"
        echo -e "${YELLOW}[i] Location: HTML/JavaScript source${NC}"
        echo -e "${YELLOW}[i] Severity: LOW${NC}"
        echo -e "${CYAN}    Emails: $emails${NC}\n"
    fi
    
    # Check for comments with sensitive info
    if echo "$response" | grep -qiE "<!--.*TODO.*-->|<!--.*FIXME.*-->|<!--.*XXX.*-->"; then
        echo -e "${YELLOW}[!] Developer comments with TODOs found${NC}"
        echo -e "${YELLOW}[i] Type: Information Disclosure${NC}"
        echo -e "${YELLOW}[i] Location: HTML comments${NC}"
        echo -e "${YELLOW}[i] Severity: LOW${NC}\n"
    fi
    
    # Check for server information
    server_header=$(curl -s -I "$1" 2>/dev/null | grep -i "Server:")
    if [ -n "$server_header" ]; then
        echo -e "${YELLOW}[!] Server information disclosed${NC}"
        echo -e "${YELLOW}[i] Type: Information Disclosure${NC}"
        echo -e "${YELLOW}[i] Location: HTTP Server Header${NC}"
        echo -e "${YELLOW}[i] Severity: LOW${NC}"
        echo -e "${CYAN}    $server_header${NC}\n"
    fi
}

# Function to generate report
generate_report() {
    local url="$1"
    local report_file="bug_report_$(date +%Y%m%d_%H%M%S).txt"
    
    {
        echo "=========================================="
        echo "KevinHarryOx Bug Detector Report"
        echo "=========================================="
        echo "Target: $url"
        echo "Date: $(date)"
        echo "=========================================="
        echo ""
        echo "This report contains potential vulnerabilities found during scanning."
        echo "Only act on findings for websites you own or have permission to test."
        echo "=========================================="
    } > "$report_file"
    
    echo -e "${GREEN}[+] Report saved to: $report_file${NC}"
}

# Main function
main() {
    # Check if required tools are installed
    check_tools
    
    # Get URL from user
    echo -e "${WHITE}┌─[${GREEN}KevinHarryOx@BugDetector${WHITE}]"
    echo -e "└──╼ ${CYAN}Enter website URL to scan (e.g., https://example.com):${NC}"
    read -r url
    
    # Validate URL
    if ! validate_url "$url"; then
        echo -e "${RED}[!] Invalid URL format. Please include http:// or https://${NC}"
        exit 1
    fi
    
    echo -e "${GREEN}[+] Starting scan on: $url${NC}\n"
    echo -e "${CYAN}================================================${NC}\n"
    
    # Run all scans
    security_headers_scan "$url"
    sql_injection_scan "$url"
    xss_scan "$url"
    directory_scan "$url"
    common_files_scan "$url"
    info_disclosure_scan "$url"
    port_scan "$url"
    ssl_scan "$url"
    
    # Generate report
    generate_report "$url"
    
    echo -e "${CYAN}================================================${NC}"
    echo -e "${GREEN}[+] Scan completed!${NC}"
    echo -e "${RED}[!] Legal Notice: Only scan websites you own or have permission to test${NC}"
    echo -e "${CYAN}================================================${NC}\n"
}

# Run main function
main
