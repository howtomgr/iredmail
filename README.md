# iRedMail Installation Guide

iRedMail is a free and open-source Mail Server Stack. Full-featured mail server solution

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 2 cores minimum (4+ cores recommended)
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 1GB for installation
  - Network: 443 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 443 (default iredmail port)
- **Dependencies**:
  - postfix, dovecot, nginx
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install iredmail
sudo dnf install -y iredmail postfix, dovecot, nginx

# Enable and start service
sudo systemctl enable --now iredmail

# Configure firewall
sudo firewall-cmd --permanent --add-service=iredmail
sudo firewall-cmd --reload

# Verify installation
iredmail --version || systemctl status iredmail
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install iredmail
sudo apt install -y iredmail postfix, dovecot, nginx

# Enable and start service
sudo systemctl enable --now iredmail

# Configure firewall
sudo ufw allow 443

# Verify installation
iredmail --version || systemctl status iredmail
```

### Arch Linux

```bash
# Install iredmail
sudo pacman -S iredmail

# Enable and start service
sudo systemctl enable --now iredmail

# Verify installation
iredmail --version || systemctl status iredmail
```

### Alpine Linux

```bash
# Install iredmail
apk add --no-cache iredmail

# Enable and start service
rc-update add iredmail default
rc-service iredmail start

# Verify installation
iredmail --version || rc-service iredmail status
```

### openSUSE/SLES

```bash
# Install iredmail
sudo zypper install -y iredmail postfix, dovecot, nginx

# Enable and start service
sudo systemctl enable --now iredmail

# Configure firewall
sudo firewall-cmd --permanent --add-service=iredmail
sudo firewall-cmd --reload

# Verify installation
iredmail --version || systemctl status iredmail
```

### macOS

```bash
# Using Homebrew
brew install iredmail

# Start service
brew services start iredmail

# Verify installation
iredmail --version
```

### FreeBSD

```bash
# Using pkg
pkg install iredmail

# Enable in rc.conf
echo 'iredmail_enable="YES"' >> /etc/rc.conf

# Start service
service iredmail start

# Verify installation
iredmail --version || service iredmail status
```

### Windows

```powershell
# Using Chocolatey
choco install iredmail

# Or using Scoop
scoop install iredmail

# Verify installation
iredmail --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /opt/iredmail

# Set up basic configuration
sudo tee /opt/iredmail/iredmail.conf << 'EOF'
# iRedMail Configuration
process_limit = 1024
EOF

# Test configuration
sudo iredmail -t || sudo iredmail configtest

# Reload service
sudo systemctl reload iredmail
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R iredmail:iredmail /opt/iredmail
sudo chmod 750 /opt/iredmail

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable iredmail

# Start service
sudo systemctl start iredmail

# Stop service
sudo systemctl stop iredmail

# Restart service
sudo systemctl restart iredmail

# Reload configuration
sudo systemctl reload iredmail

# Check status
sudo systemctl status iredmail

# View logs
sudo journalctl -u iredmail -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add iredmail default

# Start service
rc-service iredmail start

# Stop service
rc-service iredmail stop

# Restart service
rc-service iredmail restart

# Check status
rc-service iredmail status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'iredmail_enable="YES"' >> /etc/rc.conf

# Start service
service iredmail start

# Stop service
service iredmail stop

# Restart service
service iredmail restart

# Check status
service iredmail status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start iredmail
brew services stop iredmail
brew services restart iredmail

# Check status
brew services list | grep iredmail
```

### Windows Service Manager

```powershell
# Start service
net start iredmail

# Stop service
net stop iredmail

# Using PowerShell
Start-Service iredmail
Stop-Service iredmail
Restart-Service iredmail

# Check status
Get-Service iredmail
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /opt/iredmail/iredmail.conf << 'EOF'
process_limit = 1024
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart iredmail
```

### Clustering and High Availability

```bash
# Configure clustering (if supported)
# See official documentation for cluster setup

# Basic load balancing setup example
# Configure multiple instances on different ports
```

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream iredmail_backend {
    server 127.0.0.1:443;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name iredmail.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name iredmail.example.com;

    ssl_certificate /etc/ssl/certs/iredmail.example.com.crt;
    ssl_certificate_key /etc/ssl/private/iredmail.example.com.key;

    location / {
        proxy_pass http://iredmail_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support (if needed)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName iredmail.example.com
    Redirect permanent / https://iredmail.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName iredmail.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/iredmail.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/iredmail.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:443/
    ProxyPassReverse / http://127.0.0.1:443/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:443/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend iredmail_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/iredmail.pem
    redirect scheme https if !{ ssl_fc }
    default_backend iredmail_backend

backend iredmail_backend
    balance roundrobin
    option httpchk GET /health
    server iredmail1 127.0.0.1:443 check
    server iredmail2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R iredmail:iredmail /opt/iredmail
sudo chmod 750 /opt/iredmail

# Configure firewall
sudo firewall-cmd --permanent --add-service=iredmail
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/iredmail.conf << 'EOF'
[iredmail]
enabled = true
port = 443
filter = iredmail
logpath = /var/log/iredmail/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/iredmail.key \
    -out /etc/ssl/certs/iredmail.crt

# Configure SSL in iredmail
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE iredmail_db;
CREATE USER iredmail_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE iredmail_db TO iredmail_user;
EOF

# Configure iredmail to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE iredmail_db;
CREATE USER 'iredmail_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON iredmail_db.* TO 'iredmail_user'@'localhost';
FLUSH PRIVILEGES;
EOF
```

## Performance Optimization

### System Tuning

```bash
# Kernel parameters
sudo tee -a /etc/sysctl.conf << EOF
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
net.core.netdev_max_backlog = 5000
vm.swappiness = 10
EOF

sudo sysctl -p

# iRedMail specific tuning
process_limit = 1024
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
iredmail soft nofile 65535
iredmail hard nofile 65535
iredmail soft nproc 32768
iredmail hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'iredmail'
    static_configs:
      - targets: ['localhost:443']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet iredmail; then
    echo "iRedMail is running"
    exit 0
else
    echo "iRedMail is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/iredmail << 'EOF'
/var/log/iredmail/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 iredmail iredmail
    postrotate
        systemctl reload iredmail > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# iRedMail backup script
BACKUP_DIR="/backup/iredmail"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop iredmail

# Backup configuration
tar -czf "$BACKUP_DIR/iredmail-config-$DATE.tar.gz" /opt/iredmail

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/iredmail-data-$DATE.tar.gz" /var/lib/iredmail

# Start service
systemctl start iredmail

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop iredmail

# Restore configuration
sudo tar -xzf /backup/iredmail/iredmail-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/iredmail/iredmail-data-*.tar.gz -C /

# Set permissions
sudo chown -R iredmail:iredmail /opt/iredmail
sudo chown -R iredmail:iredmail /var/lib/iredmail

# Start service
sudo systemctl start iredmail
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u iredmail -n 100
sudo tail -f /var/log/iredmail/*.log

# Check configuration
sudo iredmail -t || sudo iredmail configtest

# Check permissions
ls -la /opt/iredmail
ls -la /var/lib/iredmail
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 443
sudo netstat -tlnp | grep 443

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 443
nc -zv localhost 443
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep dovecot)
htop -p $(pgrep dovecot)

# Check connections
ss -ant | grep :443 | wc -l

# Monitor I/O
iotop -p $(pgrep dovecot)
```

### Debug Mode

```bash
# Run in debug mode
sudo iredmail -d
# or
sudo iredmail debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  iredmail:
    image: iredmail:latest
    container_name: iredmail
    ports:
      - "443:443"
    volumes:
      - ./config:/opt/iredmail
      - ./data:/var/lib/iredmail
    environment:
      - iredmail_CONFIG=/opt/iredmail/iredmail.conf
    restart: unless-stopped
    networks:
      - iredmail_net

networks:
  iredmail_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iredmail
spec:
  replicas: 3
  selector:
    matchLabels:
      app: iredmail
  template:
    metadata:
      labels:
        app: iredmail
    spec:
      containers:
      - name: iredmail
        image: iredmail:latest
        ports:
        - containerPort: 443
        volumeMounts:
        - name: config
          mountPath: /opt/iredmail
      volumes:
      - name: config
        configMap:
          name: iredmail-config
---
apiVersion: v1
kind: Service
metadata:
  name: iredmail
spec:
  selector:
    app: iredmail
  ports:
  - port: 443
    targetPort: 443
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure iRedMail
  hosts: all
  become: yes
  tasks:
    - name: Install iredmail
      package:
        name: iredmail
        state: present
    
    - name: Configure iredmail
      template:
        src: iredmail.conf.j2
        dest: /opt/iredmail/iredmail.conf
        owner: iredmail
        group: iredmail
        mode: '0640'
      notify: restart iredmail
    
    - name: Start and enable iredmail
      systemd:
        name: iredmail
        state: started
        enabled: yes
  
  handlers:
    - name: restart iredmail
      systemd:
        name: iredmail
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update iredmail

# Debian/Ubuntu
sudo apt update && sudo apt upgrade iredmail

# Arch Linux
sudo pacman -Syu iredmail

# Alpine Linux
apk update && apk upgrade iredmail

# openSUSE
sudo zypper update iredmail

# FreeBSD
pkg update && pkg upgrade iredmail

# Always backup before updates
tar -czf /backup/iredmail-pre-update-$(date +%Y%m%d).tar.gz /opt/iredmail

# Restart after updates
sudo systemctl restart iredmail
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/iredmail -name "*.log" -mtime +30 -delete

# Verify integrity
sudo iredmail --verify || sudo iredmail check

# Update databases (if applicable)
sudo iredmail-update-db

# Optimize performance
sudo iredmail-optimize

# Check for security updates
sudo iredmail --security-check
```

## Additional Resources

- Official Documentation: https://docs.iredmail.org/
- GitHub Repository: https://github.com/iredmail/iredmail
- Community Forum: https://forum.iredmail.org/
- Wiki: https://wiki.iredmail.org/
- Comparison vs mailcow, Mailu, Mail-in-a-Box, Zimbra: https://docs.iredmail.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
