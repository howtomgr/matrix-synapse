# matrix-synapse Installation Guide

matrix-synapse is a free and open-source Matrix homeserver. Synapse provides Matrix protocol homeserver for decentralized communication

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
  - CPU: 2+ cores
  - RAM: 2GB minimum
  - Storage: 5GB for data
  - Network: HTTP/HTTPS Matrix
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8008 (default matrix-synapse port)
  - Federation on 8448
- **Dependencies**:
  - See official documentation for specific requirements
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

# Install matrix-synapse
sudo dnf install -y matrix-synapse

# Enable and start service
sudo systemctl enable --now matrix-synapse

# Configure firewall
sudo firewall-cmd --permanent --add-port=8008/tcp
sudo firewall-cmd --reload

# Verify installation
matrix-synapse --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install matrix-synapse
sudo apt install -y matrix-synapse

# Enable and start service
sudo systemctl enable --now matrix-synapse

# Configure firewall
sudo ufw allow 8008

# Verify installation
matrix-synapse --version
```

### Arch Linux

```bash
# Install matrix-synapse
sudo pacman -S matrix-synapse

# Enable and start service
sudo systemctl enable --now matrix-synapse

# Verify installation
matrix-synapse --version
```

### Alpine Linux

```bash
# Install matrix-synapse
apk add --no-cache matrix-synapse

# Enable and start service
rc-update add matrix-synapse default
rc-service matrix-synapse start

# Verify installation
matrix-synapse --version
```

### openSUSE/SLES

```bash
# Install matrix-synapse
sudo zypper install -y matrix-synapse

# Enable and start service
sudo systemctl enable --now matrix-synapse

# Configure firewall
sudo firewall-cmd --permanent --add-port=8008/tcp
sudo firewall-cmd --reload

# Verify installation
matrix-synapse --version
```

### macOS

```bash
# Using Homebrew
brew install matrix-synapse

# Start service
brew services start matrix-synapse

# Verify installation
matrix-synapse --version
```

### FreeBSD

```bash
# Using pkg
pkg install matrix-synapse

# Enable in rc.conf
echo 'matrix-synapse_enable="YES"' >> /etc/rc.conf

# Start service
service matrix-synapse start

# Verify installation
matrix-synapse --version
```

### Windows

```bash
# Using Chocolatey
choco install matrix-synapse

# Or using Scoop
scoop install matrix-synapse

# Verify installation
matrix-synapse --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/matrix-synapse

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
matrix-synapse --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable matrix-synapse

# Start service
sudo systemctl start matrix-synapse

# Stop service
sudo systemctl stop matrix-synapse

# Restart service
sudo systemctl restart matrix-synapse

# Check status
sudo systemctl status matrix-synapse

# View logs
sudo journalctl -u matrix-synapse -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add matrix-synapse default

# Start service
rc-service matrix-synapse start

# Stop service
rc-service matrix-synapse stop

# Restart service
rc-service matrix-synapse restart

# Check status
rc-service matrix-synapse status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'matrix-synapse_enable="YES"' >> /etc/rc.conf

# Start service
service matrix-synapse start

# Stop service
service matrix-synapse stop

# Restart service
service matrix-synapse restart

# Check status
service matrix-synapse status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start matrix-synapse
brew services stop matrix-synapse
brew services restart matrix-synapse

# Check status
brew services list | grep matrix-synapse
```

### Windows Service Manager

```powershell
# Start service
net start matrix-synapse

# Stop service
net stop matrix-synapse

# Using PowerShell
Start-Service matrix-synapse
Stop-Service matrix-synapse
Restart-Service matrix-synapse

# Check status
Get-Service matrix-synapse
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream matrix-synapse_backend {
    server 127.0.0.1:8008;
}

server {
    listen 80;
    server_name matrix-synapse.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name matrix-synapse.example.com;

    ssl_certificate /etc/ssl/certs/matrix-synapse.example.com.crt;
    ssl_certificate_key /etc/ssl/private/matrix-synapse.example.com.key;

    location / {
        proxy_pass http://matrix-synapse_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName matrix-synapse.example.com
    Redirect permanent / https://matrix-synapse.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName matrix-synapse.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/matrix-synapse.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/matrix-synapse.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8008/
    ProxyPassReverse / http://127.0.0.1:8008/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend matrix-synapse_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/matrix-synapse.pem
    redirect scheme https if !{ ssl_fc }
    default_backend matrix-synapse_backend

backend matrix-synapse_backend
    balance roundrobin
    server matrix-synapse1 127.0.0.1:8008 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R matrix-synapse:matrix-synapse /etc/matrix-synapse
sudo chmod 750 /etc/matrix-synapse

# Configure firewall
sudo firewall-cmd --permanent --add-port=8008/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status matrix-synapse

# View logs
sudo journalctl -u matrix-synapse -f

# Monitor resource usage
top -p $(pgrep matrix-synapse)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/matrix-synapse"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/matrix-synapse-backup-$DATE.tar.gz" /etc/matrix-synapse /var/lib/matrix-synapse

echo "Backup completed: $BACKUP_DIR/matrix-synapse-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop matrix-synapse

# Restore from backup
tar -xzf /backup/matrix-synapse/matrix-synapse-backup-*.tar.gz -C /

# Start service
sudo systemctl start matrix-synapse
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u matrix-synapse -n 100
sudo tail -f /var/log/matrix-synapse/matrix-synapse.log

# Check configuration
matrix-synapse --version

# Check permissions
ls -la /etc/matrix-synapse
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8008

# Test connectivity
telnet localhost 8008

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep matrix-synapse)

# Check disk I/O
iotop -p $(pgrep matrix-synapse)

# Check connections
ss -an | grep 8008
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  matrix-synapse:
    image: matrix-synapse:latest
    ports:
      - "8008:8008"
    volumes:
      - ./config:/etc/matrix-synapse
      - ./data:/var/lib/matrix-synapse
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update matrix-synapse

# Debian/Ubuntu
sudo apt update && sudo apt upgrade matrix-synapse

# Arch Linux
sudo pacman -Syu matrix-synapse

# Alpine Linux
apk update && apk upgrade matrix-synapse

# openSUSE
sudo zypper update matrix-synapse

# FreeBSD
pkg update && pkg upgrade matrix-synapse

# Always backup before updates
tar -czf /backup/matrix-synapse-pre-update-$(date +%Y%m%d).tar.gz /etc/matrix-synapse

# Restart after updates
sudo systemctl restart matrix-synapse
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/matrix-synapse

# Clean old logs
find /var/log/matrix-synapse -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/matrix-synapse
```

## Additional Resources

- Official Documentation: https://docs.matrix-synapse.org/
- GitHub Repository: https://github.com/matrix-synapse/matrix-synapse
- Community Forum: https://forum.matrix-synapse.org/
- Best Practices Guide: https://docs.matrix-synapse.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
