# The Service Layer

The Raspberry Pi hosts Jason's **security services** - DNS resolution, threat detection, and honeypots. This setup focuses on simplicity and clarity while maintaining security.

## Quick Setup

### System Preparation
```
# Update and install Docker
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io docker-compose-plugin

# Create service user
sudo useradd -r -m jason
sudo usermod -aG docker jason

# Create directory structure
sudo mkdir -p /opt/jason/{config,data,logs}
sudo chown -R jason:jason /opt/jason
```

### Service Stack
Create `/opt/jason/docker-compose.yml`:

```
version: '3.8'

services:
  # DNS Resolver with DNSSEC
  unbound:
    image: mvance/unbound:latest
    container_name: unbound
    ports:
      - "192.168.200.100:53:53/udp"
      - "192.168.200.100:53:53/tcp"
    volumes:
      - ./config/unbound.conf:/opt/unbound/etc/unbound/unbound.conf:ro
      - ./logs/unbound:/opt/unbound/logs
    restart: unless-stopped
    mem_limit: 256m

  # DNS Sinkhole
  technitium:
    image: technitium/dns-server:latest
    container_name: technitium
    ports:
      - "192.168.200.100:5380:5380"
    volumes:
      - ./data/technitium:/etc/dns
    environment:
      - DNS_SERVER_ADMIN_PASSWORD=ChangeThisPassword123!
    restart: unless-stopped
    mem_limit: 512m

  # Threat Detection
  crowdsec:
    image: crowdsecurity/crowdsec:latest
    container_name: crowdsec
    ports:
      - "127.0.0.1:8080:8080"
    volumes:
      - ./config/crowdsec/acquis.yaml:/etc/crowdsec/acquis.yaml:ro
      - ./logs:/logs:ro
      - /var/log:/var/log:ro
    environment:
      - COLLECTIONS=crowdsecurity/linux crowdsecurity/iptables
    restart: unless-stopped
    mem_limit: 512m

  # SSH Honeypot
  cowrie:
    image: cowrie/cowrie:latest
    container_name: cowrie
    ports:
      - "192.168.200.100:2222:2222"
    volumes:
      - ./data/cowrie:/cowrie/cowrie-git/var
      - ./logs/cowrie:/cowrie/cowrie-git/var/log
    restart: unless-stopped
    mem_limit: 256m

  # HTTP Honeypot
  dionaea:
    image: dinotools/dionaea:latest
    container_name: dionaea
    ports:
      - "192.168.200.100:8080:80"
      - "192.168.200.100:8443:443"
    volumes:
      - ./logs/dionaea:/opt/dionaea/var/log
    restart: unless-stopped
    mem_limit: 256m

  # Log Collection
  rsyslog:
    image: rsyslog/syslog_appliance_alpine:latest
    container_name: rsyslog
    ports:
      - "192.168.200.100:514:514/udp"
    volumes:
      - ./logs/collected:/logs
    restart: unless-stopped
    mem_limit: 256m

  # Monitoring Dashboard
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "192.168.200.100:3000:3000"
    volumes:
      - ./data/grafana:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=ChangeThisPassword123!
    restart: unless-stopped
    mem_limit: 512m
```

## Configuration Files

### Unbound DNS (`/opt/jason/config/unbound.conf`)
```
server:
    # Basic settings
    interface: 0.0.0.0@53
    port: 53
    do-ip4: yes
    do-ip6: no
    
    # Security
    hide-identity: yes
    hide-version: yes
    harden-dnssec-stripped: yes
    
    # DNSSEC validation
    auto-trust-anchor-file: "/opt/unbound/var/root.key"
    
    # Performance (Pi-optimized)
    num-threads: 2
    rrset-cache-size: 64m
    msg-cache-size: 32m
    
    # Privacy
    qname-minimisation: yes
    
    # Access control
    access-control: 0.0.0.0/0 refuse
    access-control: 127.0.0.0/8 allow
    access-control: 192.168.100.0/24 allow    # IoT devices
    access-control: 192.168.1.0/24 allow      # Management
    
    # Logging
    logfile: "/opt/unbound/logs/unbound.log"
    log-queries: yes
    verbosity: 1
```

### CrowdSec Log Sources (`/opt/jason/config/crowdsec/acquis.yaml`)
```
# System logs
filenames:
  - /var/log/auth.log
  - /var/log/syslog
labels:
  type: syslog

# Honeypot logs  
filenames:
  - /logs/cowrie/*
labels:
  type: cowrie

# DNS logs
filenames:
  - /logs/unbound/*
labels:
  type: unbound
```

## Deployment

### Start Services
```
# Switch to service user
sudo su - jason
cd /opt/jason

# Create required directories
mkdir -p {config,data,logs}/{unbound,technitium,crowdsec,cowrie,dionaea,grafana}

# Start the stack
docker compose up -d

# Check status
docker compose ps
docker compose logs --tail=20
```

### Initial Configuration

**1. Configure Technitium Sinkhole:**
- Open `https://192.168.200.100:5380`
- Login with admin/ChangeThisPassword123!
- Go to **Zones → Add Zone**
- Create sinkhole zone for malicious domains

**2. Configure Grafana Dashboard:**
- Open `https://192.168.200.100:3000`
- Login with admin/ChangeThisPassword123!
- Add Prometheus data source (if needed)
- Import security monitoring dashboards

**3. Test DNS Resolution:**
```
# Test from IoT device
nslookup google.com 192.168.200.100
dig @192.168.200.100 +dnssec cloudflare.com
```

**4. Test Honeypots:**
```
# SSH honeypot
ssh test@192.168.200.100 -p 2222

# HTTP honeypot  
curl http://192.168.200.100:8080
```

## Monitoring and Maintenance

### Health Checks
```
# Check service health
docker compose ps
docker stats

# View logs
docker compose logs unbound --tail=50
docker compose logs crowdsec --tail=50

# Monitor honeypot activity
tail -f logs/cowrie/cowrie.log
tail -f logs/dionaea/dionaea.log
```

### Regular Tasks
```
# Weekly updates (create as cron job)
docker compose pull
docker compose up -d

# Log cleanup
find logs/ -name "*.log" -mtime +30 -delete

# Security scan
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image cowrie/cowrie:latest
```

### Backup Configuration
```
# Backup script
tar -czf backup-$(date +%Y%m%d).tar.gz \
  docker-compose.yml config/ data/

# Copy to safe location
scp backup-*.tar.gz admin@backup-server:/backups/jason/
```

This setup gives you **production-ready security services**. Each service has a clear purpose, and the configuration is easy to understand and modify. The stack provides DNS security, threat detection, deception capabilities, and monitoring - everything needed for service layer.