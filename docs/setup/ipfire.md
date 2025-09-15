# IPFire Configuration: Segmentation Plane Security

The NanoPi R2S segmentation plane transforms abstract security policy into **mathematically enforceable network behavior** through IPFire's zone-based architecture. This configuration addresses the hardware constraint of **dual Ethernet ports only** by implementing VLAN-based zone separation on the internal interface.

## Hardware Architecture and Port Assignment

### Physical Interface Mapping (NanoPi R2S Dual-Port Reality)

**Critical hardware constraint**: The NanoPi R2S has **only two Ethernet ports**, requiring VLAN trunking for multi-zone support[30].

**Port assignment**:
- **Upper port (eth0)**: WAN/RED zone - Internet connectivity
- **Lower port (eth1)**: Combined GREEN/ORANGE zones via VLAN trunking

**Network topology requirement**:
```
Internet → [eth0/RED] NanoPi R2S [eth1] → VLAN-capable switch
                                              ├── Untagged: Management devices (GREEN)
                                              └── VLAN 100: From NanoPi R1S (ORANGE)
```

### VLAN Trunking Configuration

**Lower port VLAN assignment**:
- **Native/Untagged**: GREEN zone (192.168.1.0/24) - Management network
- **VLAN 100**: ORANGE zone (192.168.100.0/24) - IoT devices from access plane
- **VLAN 200**: BLUE zone (192.168.200.0/24) - Services plane

## Initial System Access and Setup

### Serial Console Access
```
# NanoPi R2S uses standard baud rate (not 1.5M like R1S)
screen /dev/ttyUSB0 115200
# or
picocom -b 115200 /dev/ttyUSB0
```

**Debug header pinout** (3-pin, 2.54mm spacing):
```
[GND] [RX] [TX]
  1     2    3
```

### Initial Boot and Interface Recognition

**Network setup wizard configuration**:
1. **Network configuration type**: Select `GREEN + RED + ORANGE`
2. **Interface assignment**:
   - **RED zone**: Upper port (eth0) - WAN interface
   - **GREEN zone**: Lower port (eth1) - Management interface  
   - **ORANGE zone**: VLAN interface (eth1.100) - IoT zone
3. **IP addressing**:
   - **RED**: DHCP or static (upstream dependent)
   - **GREEN**: 192.168.1.1/24
   - **ORANGE**: 192.168.100.1/24

### Post-Installation Security Hardening

**Immediate credential security**:
```
# Change default passwords during setup
# Root password: Strong unique password for console access
# Admin password: Strong unique password for web interface access
# Enable two-factor authentication if available
```

**System updates and essential packages**:
```
# Update system to latest version
pakfire update
pakfire upgrade

# Install essential security packages
pakfire install squid3 suricata snort clamav

# Verify system status
cat /opt/pakfire/etc/pakfire.conf
pakfire list --installed | grep -E "(suricata|snort|clamav)"
```

## Zone-Based Security Architecture Implementation

### Network Zone Configuration

**Zone definitions and security boundaries**:

**RED Zone (Internet/Untrusted)**:
- **Interface**: eth0 (upper port)
- **Security level**: Zero trust - all inbound denied by default
- **Purpose**: Internet connectivity with complete upstream isolation

**GREEN Zone (Management/Trusted)**:
- **Interface**: eth1 (lower port, untagged)
- **IP range**: 192.168.1.0/24
- **Security level**: Full internal access and management privileges
- **Purpose**: Administrative access, monitoring systems, trusted devices

**ORANGE Zone (IoT/Restricted)**:
- **Interface**: eth1.100 (VLAN 100 on lower port)
- **IP range**: 192.168.100.0/24  
- **Security level**: Minimal privileges, strict egress control
- **Purpose**: IoT device containment and controlled external access

### VLAN Interface Configuration

**Command-line VLAN setup**:
```
# Create VLAN interface for ORANGE zone
echo "ORANGE_DEV=eth1.100" >> /var/ipfire/ethernet/settings
echo "ORANGE_ADDRESS=192.168.100.1" >> /var/ipfire/ethernet/settings
echo "ORANGE_NETADDRESS=192.168.100.0" >> /var/ipfire/ethernet/settings
echo "ORANGE_NETMASK=255.255.255.0" >> /var/ipfire/ethernet/settings
echo "ORANGE_ENABLED=on" >> /var/ipfire/ethernet/settings

# Optional: Services plane BLUE zone
echo "BLUE_DEV=eth1.200" >> /var/ipfire/ethernet/settings
echo "BLUE_ADDRESS=192.168.200.1" >> /var/ipfire/ethernet/settings
echo "BLUE_NETADDRESS=192.168.200.0" >> /var/ipfire/ethernet/settings
echo "BLUE_NETMASK=255.255.255.0" >> /var/ipfire/ethernet/settings
echo "BLUE_ENABLED=on" >> /var/ipfire/ethernet/settings

# Apply network configuration
/etc/init.d/network restart
```

**Web interface VLAN configuration**:
1. Navigate to **Network → Zone Configuration**
2. Click **ORANGE Configuration**
3. Set interface parameters:
   - **Interface**: `eth1.100`
   - **IP Address**: `192.168.100.1`
   - **Subnet Mask**: `255.255.255.0`
   - **Enable Interface**: ✓ Checked

### DHCP Configuration by Zone

**Zone-specific DHCP pools**:
```
# GREEN zone DHCP (management devices)
# Navigate to: Network → DHCP Server → GREEN
# Start IP: 192.168.1.100
# End IP: 192.168.1.200
# Default gateway: 192.168.1.1
# DNS servers: 192.168.1.1, 192.168.200.100
# Lease time: 24 hours

# ORANGE zone DHCP (IoT devices)  
# Navigate to: Network → DHCP Server → ORANGE
# Start IP: 192.168.100.10
# End IP: 192.168.100.100
# Default gateway: 192.168.100.1
# DNS servers: 192.168.200.100 (Services plane only)
# Lease time: 12 hours
# Additional options: NTP server, domain name
```

**DHCP security hardening**:
```
# Add to DHCP configuration files:
# /var/ipfire/dhcp/dhcpd.conf

# ORANGE zone security options
subnet 192.168.100.0 netmask 255.255.255.0 {
    range 192.168.100.10 192.168.100.100;
    option routers 192.168.100.1;
    option domain-name-servers 192.168.200.100;
    option ntp-servers 192.168.200.100;
    default-lease-time 43200;
    max-lease-time 86400;
    
    # Security options
    option dhcp-parameter-request-list 1,3,6,15,119,252;
    deny unknown-clients;  # Only serve known MAC addresses
}
```

## Firewall Rules: Default-Deny Implementation

### Core Security Policy Framework

**Fundamental firewall philosophy**:
- **Default policy**: DENY ALL (explicit permit required for every flow)
- **Minimum privilege**: Grant only essential access for device functionality
- **Audit everything**: Log all denied and permitted traffic
- **Time-based controls**: Implement temporal restrictions where appropriate

### Essential Inter-Zone Rules

**Rule Set 1: IoT to Services Communication**
```
# Rule: IoT DNS Resolution
Source: ORANGE network (192.168.100.0/24)
Destination: Services plane (192.168.200.100)
Service: DNS (TCP/UDP 53)
Action: ACCEPT
Logging: Enable (all connections)
Time restriction: None
Rate limit: 100 queries/minute per source

# Rule: IoT NTP Synchronization  
Source: ORANGE network (192.168.100.0/24)
Destination: Services plane (192.168.200.100)
Service: NTP (UDP 123)
Action: ACCEPT
Logging: Enable
Time restriction: None
Rate limit: 10 requests/minute per source
```

**Rule Set 2: IoT Internet Access (Controlled)**
```
# Rule: IoT HTTP/HTTPS (Essential)
Source: ORANGE network (192.168.100.0/24)
Destination: RED network (Internet)
Service: HTTP/HTTPS (TCP 80,443)
Action: ACCEPT
Logging: Enable (connections and data volume)
Time restriction: Business hours (optional)
Rate limit: 10 Mbps per source IP
Additional: Content filtering via proxy

# Rule: IoT Cloud Services (Specific)
Source: ORANGE network (192.168.100.0/24)  
Destination: Approved cloud endpoints (whitelist)
Service: HTTPS (TCP 443)
Action: ACCEPT
Logging: Enable (full packet headers)
Time restriction: None
Additional: Deep packet inspection
```

**Rule Set 3: Management Access**
```
# Rule: Admin to all zones
Source: GREEN network (192.168.1.0/24)
Destination: Any zone
Service: SSH (TCP 22), HTTPS (TCP 443), SNMP (UDP 161)
Action: ACCEPT
Logging: Enable
Source IP restriction: Admin workstations only
Additional: Certificate-based authentication preferred

# Rule: Services plane monitoring
Source: Services plane (192.168.200.100)
Destination: Firewall interfaces
Service: SNMP (UDP 161), Syslog (UDP 514)
Action: ACCEPT
Logging: Enable
Additional: SNMP v3 with encryption
```

**Rule Set 4: Explicit Deny Rules (Logging)**
```
# Rule: Inter-IoT device communication
Source: ORANGE network (192.168.100.0/24)
Destination: ORANGE network (192.168.100.0/24)
Service: Any
Action: DROP
Logging: Enable (security events)
Alert: Generate security alert

# Rule: IoT to management network
Source: ORANGE network (192.168.100.0/24)
Destination: GREEN network (192.168.1.0/24)
Service: Any  
Action: DROP
Logging: Enable (security events)
Alert: Generate high-priority security alert
```

### Web Interface Rule Configuration

**Firewall rule creation process**:
1. Navigate to **Firewall → Firewall Rules**
2. Click **New Rule**
3. Configure rule parameters:
   - **Used**: ✓ Enable rule
   - **Action**: ACCEPT/DROP/REJECT
   - **Source**: Select zone or custom network
   - **Destination**: Select zone, network, or specific hosts
   - **Protocol**: TCP/UDP/ICMP/GRE/ESP
   - **Source port**: Usually "Any" unless specific requirement
   - **Destination port**: Specific service ports or port ranges
4. **Advanced Options**:
   - **Log**: Enable for security-relevant rules
   - **Time constraints**: Business hours, maintenance windows
   - **Rate limiting**: Connections per second/minute
   - **Remark**: Detailed description of rule purpose

**Rule ordering and optimization**:
```
# Rules are processed top-to-bottom
# Order by frequency: Most common rules first
# Group related rules together
# Place DENY rules after specific ALLOW rules
# Use "Insert Rule" to maintain logical ordering
```

## Advanced Security Configuration

### Intrusion Detection and Prevention (IDS/IPS)

**Suricata IDS/IPS configuration**:
```
# Navigate to: Services → Intrusion Detection System

# Enable IDS/IPS: Yes
# IPS mode: Enabled (block malicious traffic)
# Ruleset selection:
# - Emerging Threats Open (free, updated daily)
# - Snort Community Rules
# - Custom local rules

# Performance tuning for ARM hardware:
# Detection threads: 2 (match CPU cores)  
# Memory allocation: Conservative (512MB max)
# Rule optimization: Enable fast pattern matcher
```

**Custom IoT-specific detection rules**:
```
# Create: /etc/suricata/rules/jason-iot.rules

# Detect IoT device communication violations
alert tcp $ORANGE_NET any -> $GREEN_NET any (msg:"IoT device accessing management network"; sid:1000001; rev:1; priority:1;)

# Detect unusual IoT protocols  
alert tcp $ORANGE_NET any -> $EXTERNAL_NET ![8080][8443] (msg:"IoT device unusual port access"; sid:1000002; rev:1; priority:2;)

# Detect potential botnet communication
alert tcp $ORANGE_NET any -> $EXTERNAL_NET any (msg:"IoT device high frequency connections"; threshold:type both, track by_src, count 100, seconds 60; sid:1000003; rev:1; priority:1;)

# Detect scanning behavior
alert icmp $ORANGE_NET any -> any any (msg:"IoT device network scanning"; threshold:type both, track by_src, count 50, seconds 30; sid:1000004; rev:1; priority:2;)
```

### DNS Security Implementation

**DNS over HTTPS/TLS configuration**:
```
# Navigate to: Network → Domain Name System

# Forwarders configuration:
# Primary: 192.168.200.100 (Services plane Unbound)
# Secondary: 9.9.9.9 (Quad9 with malware blocking)
# Tertiary: 1.1.1.1 (Cloudflare)

# Security options:
# Enable DNS over TLS: Yes
# DNSSEC validation: Strict
# Query logging: Enable for security analysis
# Cache poisoning protection: Enable
```

**DNS filtering and sinkholing**:
```
# Custom DNS blacklist for IoT security
# Create: /etc/unbound/custom-blacklist.conf

# Block known IoT malware domains
local-zone: "malware-c2-domain.com" refuse
local-zone: "iot-botnet-control.net" refuse  
local-zone: "suspicious-firmware-update.org" refuse

# Block privacy-violating telemetry
local-zone: "device-telemetry-collector.com" refuse
local-zone: "analytics-harvester.net" refuse

# Redirect to local sinkhole for analysis
local-zone: "research-target.example" redirect  
local-data: "research-target.example A 192.168.200.250"
```

### Web Proxy Integration

**Squid proxy for HTTPS inspection**:
```
# Navigate to: Services → Web Proxy

# Transparent proxy: Enable for ORANGE zone
# SSL certificate inspection: Enable (with custom CA)
# Content filtering: Enable malware/phishing protection
# Cache settings: Minimal (privacy-focused)
# Access control: ORANGE zone only

# Upstream proxy configuration:
# Parent proxy: Services plane (if configured)
# Direct internet: Fallback only
```

**SSL/TLS inspection setup**:
```
# Generate inspection CA certificate
# Navigate to: Services → Web Proxy → SSL Certificate

# Create custom CA for SSL inspection
# Deploy CA certificate to managed devices
# Enable HTTPS inspection for security analysis
# Log certificate errors and anomalies
```

## Traffic Shaping and Quality of Service

### Bandwidth Management by Zone

**QoS policy implementation**:
```
# Navigate to: Network → Quality of Service

# Total bandwidth allocation (adjust for WAN speed):
# GREEN zone: 60% (management and critical services)
# ORANGE zone: 30% (IoT devices, rate-limited)  
# System/RED: 10% (firewall operations, updates)

# Priority classifications:
# High: SSH, HTTPS management, DNS, NTP
# Medium: HTTP/HTTPS from GREEN zone
# Low: IoT HTTP/HTTPS, bulk transfers
# Lowest: P2P, streaming (if allowed)
```

**Traffic shaping rules**:
```
# Rule 1: Prioritize management traffic
Source: GREEN zone
Traffic type: SSH, SNMP, HTTPS management
Bandwidth: Guaranteed 10 Mbps, burst to 50 Mbps
Priority: High

# Rule 2: Limit IoT bandwidth per device
Source: ORANGE zone  
Traffic type: All
Bandwidth: 1 Mbps per IP, total 20 Mbps
Priority: Low
Burst allowed: No

# Rule 3: Emergency management access
Source: Any zone
Traffic type: SSH to firewall
Bandwidth: Guaranteed 1 Mbps
Priority: Highest (cannot be throttled)
```

## System Hardening and Attack Surface Reduction

### Service Minimization

**Disable unnecessary services**:
```
# Navigate to: System → Services

# Disable unused services:
# - IPFire Dynamic DNS (unless required)
# - Wake-on-LAN (security risk)
# - UPNP (major security risk)  
# - Samba/SMB (not needed for firewall)
# - FTP server (use SFTP instead)

# Essential services only:
# - SSH (hardened configuration)
# - Web interface (restricted access)
# - NTP client (time synchronization)
# - Syslog (centralized logging)
```

**Package management security**:
```
# Remove unused addons
pakfire list --installed | grep -v "essential\|core\|base"
pakfire remove [unused-addon]

# Minimize attack surface
pakfire list --available | grep -E "(game|media|desktop)" 
# Avoid installing non-security packages
```

### SSH Hardening Configuration

**Secure SSH daemon setup**:
```
# Edit /etc/ssh/sshd_config
Protocol 2
Port 2222                          # Non-standard port
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
MaxAuthTries 3
MaxStartups 3:30:10
ClientAliveInterval 300
ClientAliveCountMax 2
UseDNS no
X11Forwarding no
AllowTcpForwarding no
GatewayPorts no
PermitTunnel no

# Restrict users and groups
AllowUsers admin
AllowGroups admin

# Enable logging
SyslogFacility AUTHPRIV
LogLevel VERBOSE

# Restart SSH service
systemctl restart sshd
```

### Web Interface Security Hardening

**Apache security configuration**:
```
# Edit /etc/httpd/conf/httpd.conf

# Hide server information
ServerTokens Prod  
ServerSignature Off

# Security headers
Header always set X-Frame-Options "SAMEORIGIN"
Header always set X-Content-Type-Options "nosniff"  
Header always set X-XSS-Protection "1; mode=block"
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
Header always set Content-Security-Policy "default-src 'self'"

# Disable dangerous methods
<Location />
    <LimitExcept GET POST HEAD>
        Deny from all
    </LimitExcept>
</Location>

# Restart Apache
systemctl restart httpd
```

**Web interface access control**:
```
# Navigate to: System → Access

# Restrict access by source IP:
# Allowed networks: GREEN zone only (192.168.1.0/24)
# Block all other networks

# Enable HTTPS only:
# Force SSL: Yes
# HTTP redirect to HTTPS: Yes
# Strong cipher suites only: Yes

# Session security:
# Session timeout: 30 minutes
# Concurrent sessions: 2 per user
# Lock account after failed attempts: 5
```

## Logging and Monitoring Configuration

### Centralized Logging Setup

**Syslog forwarding to services plane**:
```
# Edit /etc/rsyslog.conf

# Forward all logs to services plane
*.* @@192.168.200.100:514

# Local log retention (backup)
*.info /var/log/messages
authpriv.* /var/log/secure
mail.* /var/log/maillog  
cron.* /var/log/cron

# Restart rsyslog
systemctl restart rsyslog
```

**Firewall-specific logging**:
```
# Enhanced iptables logging
# Edit /var/ipfire/optionsfw/settings

# Enable comprehensive logging
LOG_MARTIANS=on
DROPFW_DEFAULT_NETWORKS=on
DROPFW_LOCALIZED_ATTACKS=on
LOG_USERSPACE=on

# Custom log format for analysis
IPTABLES_LOG_FORMAT="IPFire[%t]: IN=%i OUT=%o SRC=%s DST=%d PROTO=%p SPT=%{tcp:sport,udp:sport} DPT=%{tcp:dport,udp:dport}"

# Apply settings
/etc/init.d/firewall restart
```

### SNMP Monitoring Configuration

**SNMP v3 setup for security**:
```
# Navigate to: Services → SNMP

# SNMP version: v3 only (v1/v2 disabled for security)
# User authentication: SHA-256
# Privacy protocol: AES-256  
# Community strings: Disabled
# Allowed hosts: 192.168.200.100 (Services plane only)

# MIB access restrictions:
# System information: Read-only
# Interface statistics: Read-only  
# Firewall rules: No access
# Configuration: No access
```

**Custom SNMP monitoring objects**:
```
# Edit /etc/snmp/snmpd.conf

# Firewall-specific OIDs
extend .1.3.6.1.4.1.2021.8.1 zone_traffic /usr/local/bin/zone_stats.sh
extend .1.3.6.1.4.1.2021.8.2 rule_hits /usr/local/bin/rule_stats.sh
extend .1.3.6.1.4.1.2021.8.3 threat_level /usr/local/bin/threat_stats.sh

# Create monitoring scripts
# /usr/local/bin/zone_stats.sh - Monitor traffic by zone
# /usr/local/bin/rule_stats.sh - Track firewall rule hits  
# /usr/local/bin/threat_stats.sh - IDS threat level summary
```

## Performance Optimization and Tuning

### Hardware-Specific Optimizations

**ARM Cortex-A53 tuning for NanoPi R2S**:
```
# Edit /etc/sysctl.conf

# Network performance optimization
net.core.netdev_max_backlog = 5000
net.core.rmem_default = 262144
net.core.rmem_max = 16777216  
net.core.wmem_default = 262144
net.core.wmem_max = 16777216

# Connection tracking optimization  
net.netfilter.nf_conntrack_max = 32768
net.netfilter.nf_conntrack_tcp_timeout_established = 7440
net.netfilter.nf_conntrack_tcp_timeout_close_wait = 60
net.netfilter.nf_conntrack_udp_timeout = 30

# Memory management for limited RAM
vm.swappiness = 1
vm.dirty_background_ratio = 5
vm.dirty_ratio = 10

# Apply settings
sysctl -p
```

**Firewall rule optimization**:
```
# Monitor rule performance
iptables -L FORWARDFW -n -v --line-numbers

# Optimize rule order:
# 1. Most frequently matched rules first
# 2. Specific rules before general rules  
# 3. ACCEPT rules before DROP rules
# 4. Source-based grouping for efficiency

# Remove unused rules periodically
# Monitor with: iptables -L -v -n | grep "0     0"
```

### Resource Monitoring and Alerting

**System resource monitoring**:
```
# CPU and memory monitoring
# Create: /usr/local/bin/system_monitor.sh

#!/bin/bash
# System resource monitoring for Jason firewall

CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | sed 's/%us,//')
MEM_USAGE=$(free | grep Mem | awk '{printf("%.2f"), $3/$2 * 100.0}')
DISK_USAGE=$(df -h / | awk 'NR==2 {print $5}' | sed 's/%//')

# Alert thresholds
if (( $(echo "$CPU_USAGE > 80" | bc -l) )); then
    logger "WARNING: High CPU usage: $CPU_USAGE%"
fi

if (( $(echo "$MEM_USAGE > 90" | bc -l) )); then
    logger "WARNING: High memory usage: $MEM_USAGE%"  
fi

if [ "$DISK_USAGE" -gt 85 ]; then
    logger "WARNING: High disk usage: $DISK_USAGE%"
fi

# Network interface monitoring
for interface in eth0 eth1 eth1.100; do
    if ! ip link show "$interface" | grep -q "UP"; then
        logger "ERROR: Interface $interface is down"
    fi
done
```

## Backup and Recovery Procedures

### Configuration Backup Strategy

**Automated backup configuration**:
```
# Navigate to: System → Backup

# Backup content selection:
# ✓ IPFire settings and configuration
# ✓ Firewall rules and policies  
# ✓ Network configuration
# ✓ SSL certificates and keys
# ✓ User accounts and authentication
# ✓ Addon configurations
# ✓ Custom scripts and modifications

# Backup schedule: Daily at 2:00 AM
# Backup location: External storage + services plane
# Retention: 30 days local, 90 days remote
# Encryption: Yes (AES-256)
```

**Manual backup procedures**:
```
# Critical configuration files backup
tar -czf /tmp/ipfire-config-$(date +%Y%m%d).tar.gz \
    /var/ipfire/ethernet/settings \
    /var/ipfire/optionsfw/settings \
    /etc/sysconfig/firewall \
    /var/ipfire/firewall/ \
    /etc/ssh/sshd_config \
    /etc/httpd/conf/ \
    /etc/snmp/snmpd.conf

# Copy to services plane for storage
scp /tmp/ipfire-config-*.tar.gz admin@192.168.200.100:/backup/ipfire/
```

### Disaster Recovery Planning

**Recovery procedures documentation**:
```
# Emergency recovery steps:
# 1. Reflash IPFire image to new SD card
# 2. Complete initial setup wizard
# 3. Restore configuration from backup
# 4. Verify all zones and interfaces
# 5. Test firewall rules functionality  
# 6. Restore certificates and keys
# 7. Validate monitoring and logging
# 8. Perform security verification

# Recovery time objective: 2 hours
# Recovery point objective: 24 hours  
# Test recovery quarterly
```

## Security Validation and Testing

### Firewall Rule Testing Procedures

**Systematic rule validation**:
```
# Test 1: Inter-zone isolation
# From ORANGE zone device (192.168.100.x):
nmap -sT 192.168.1.1          # Should be blocked
ping 192.168.1.100            # Should be blocked  
ssh admin@192.168.1.1         # Should be blocked

# Test 2: Allowed services
# From ORANGE zone device:
nslookup google.com 192.168.200.100    # Should work
curl http://www.google.com              # Should work (if allowed)
ntpdate -q 192.168.200.100             # Should work

# Test 3: Management access
# From GREEN zone device (192.168.1.x):
ssh admin@192.168.100.1        # Should work
https://192.168.100.1:444       # Should work (web interface)
snmpwalk -v3 192.168.100.1      # Should work (if configured)
```

**Security penetration testing**:
```
# Test 4: VLAN hopping attempts
# Verify VLAN isolation integrity
tcpdump -i eth1 -n vlan        # Monitor VLAN traffic separation

# Test 5: Firewall bypass attempts  
# Test for common firewall evasion techniques
# Fragment attacks, connection state manipulation
# Protocol tunneling attempts

# Test 6: DoS resistance
# SYN flood testing (controlled)
# Connection exhaustion testing
# Bandwidth exhaustion testing
```

### Performance Baseline Testing

**Network throughput measurement**:
```
# Baseline throughput testing
# GREEN to RED (management internet access)
iperf3 -c [external-server] -t 60 -i 10

# ORANGE to RED (IoT internet access)  
iperf3 -c [external-server] -t 60 -i 10 -B 192.168.100.x

# Inter-zone blocked traffic (should fail)
iperf3 -c 192.168.1.100 -B 192.168.100.x

# Document baseline performance for monitoring
```

**Firewall processing overhead**:
```
# Measure processing delay
hping3 -S -p 80 -i u100 [target]     # 10,000 pps test
# Monitor CPU usage during test
top -p $(pgrep iptables)

# Connection tracking overhead
ss -tuln | wc -l                     # Active connections
cat /proc/net/nf_conntrack | wc -l   # Tracked connections
```

## Maintenance and Operational Procedures

### Regular Security Updates

**Update management strategy**:
```
# Weekly security update check
pakfire update
pakfire list --upgrades

# Critical security updates: Apply immediately
# Feature updates: Test in lab environment first
# Kernel updates: Require reboot planning

# Update verification procedure:
# 1. Backup configuration before update
# 2. Apply updates during maintenance window
# 3. Verify all zones functional after reboot
# 4. Test critical firewall rules
# 5. Monitor logs for anomalies
```

### Log Analysis and Security Monitoring

**Daily security review procedures**:
```
# Review firewall denies
grep "DENY\|DROP\|REJECT" /var/log/messages | tail -100

# Check for authentication failures  
grep "authentication failure" /var/log/secure

# Monitor for unusual traffic patterns
grep "unusual\|anomaly\|suspicious" /var/log/messages

# IDS/IPS alert review
grep -E "(ALERT|WARNING|CRITICAL)" /var/log/suricata/fast.log

# Weekly security report generation
# Monthly trend analysis
# Quarterly security posture review
```

### Capacity Planning and Scaling

**Performance monitoring and thresholds**:
```
# Weekly capacity review:
# - CPU utilization trends
# - Memory consumption patterns  
# - Network throughput utilization
# - Connection tracking table usage
# - Disk space consumption

# Scaling triggers:
# - CPU >70% sustained
# - Memory >85% sustained  
# - Connection tracking >80% of maximum
# - Network utilization >80% of capacity

# Scaling options:
# - Hardware upgrade to higher-spec SBC
# - Load balancing across multiple firewalls
# - Rule optimization for performance
# - Traffic engineering and QoS adjustment
```

This configuration creates **immutable security boundaries** through zone-based architecture. The configuration emphasizes **defense in depth**, **comprehensive logging**, and **systematic hardening** to create a ready and deployable platform that can withstand both automated attacks and sophisticated targeted intrusions while providing the visibility necessary for operational awareness.