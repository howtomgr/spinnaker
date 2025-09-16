# spinnaker Installation Guide

spinnaker is a free and open-source multi-cloud continuous delivery platform. Originally from Netflix, Spinnaker provides advanced deployment strategies across multiple cloud providers

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
  - CPU: 4+ cores recommended
  - RAM: 16GB minimum
  - Storage: 10GB for installation
  - Network: Cloud provider APIs
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9000 (default spinnaker port)
  - Port 8084 for gate API
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

# Install spinnaker
sudo dnf install -y spinnaker

# Enable and start service
sudo systemctl enable --now spinnaker

# Configure firewall
sudo firewall-cmd --permanent --add-port=9000/tcp
sudo firewall-cmd --reload

# Verify installation
hal version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install spinnaker
sudo apt install -y spinnaker

# Enable and start service
sudo systemctl enable --now spinnaker

# Configure firewall
sudo ufw allow 9000

# Verify installation
hal version
```

### Arch Linux

```bash
# Install spinnaker
sudo pacman -S spinnaker

# Enable and start service
sudo systemctl enable --now spinnaker

# Verify installation
hal version
```

### Alpine Linux

```bash
# Install spinnaker
apk add --no-cache spinnaker

# Enable and start service
rc-update add spinnaker default
rc-service spinnaker start

# Verify installation
hal version
```

### openSUSE/SLES

```bash
# Install spinnaker
sudo zypper install -y spinnaker

# Enable and start service
sudo systemctl enable --now spinnaker

# Configure firewall
sudo firewall-cmd --permanent --add-port=9000/tcp
sudo firewall-cmd --reload

# Verify installation
hal version
```

### macOS

```bash
# Using Homebrew
brew install spinnaker

# Start service
brew services start spinnaker

# Verify installation
hal version
```

### FreeBSD

```bash
# Using pkg
pkg install spinnaker

# Enable in rc.conf
echo 'spinnaker_enable="YES"' >> /etc/rc.conf

# Start service
service spinnaker start

# Verify installation
hal version
```

### Windows

```bash
# Using Chocolatey
choco install spinnaker

# Or using Scoop
scoop install spinnaker

# Verify installation
hal version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/spinnaker

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
hal version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable spinnaker

# Start service
sudo systemctl start spinnaker

# Stop service
sudo systemctl stop spinnaker

# Restart service
sudo systemctl restart spinnaker

# Check status
sudo systemctl status spinnaker

# View logs
sudo journalctl -u spinnaker -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add spinnaker default

# Start service
rc-service spinnaker start

# Stop service
rc-service spinnaker stop

# Restart service
rc-service spinnaker restart

# Check status
rc-service spinnaker status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'spinnaker_enable="YES"' >> /etc/rc.conf

# Start service
service spinnaker start

# Stop service
service spinnaker stop

# Restart service
service spinnaker restart

# Check status
service spinnaker status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start spinnaker
brew services stop spinnaker
brew services restart spinnaker

# Check status
brew services list | grep spinnaker
```

### Windows Service Manager

```powershell
# Start service
net start spinnaker

# Stop service
net stop spinnaker

# Using PowerShell
Start-Service spinnaker
Stop-Service spinnaker
Restart-Service spinnaker

# Check status
Get-Service spinnaker
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream spinnaker_backend {
    server 127.0.0.1:9000;
}

server {
    listen 80;
    server_name spinnaker.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name spinnaker.example.com;

    ssl_certificate /etc/ssl/certs/spinnaker.example.com.crt;
    ssl_certificate_key /etc/ssl/private/spinnaker.example.com.key;

    location / {
        proxy_pass http://spinnaker_backend;
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
    ServerName spinnaker.example.com
    Redirect permanent / https://spinnaker.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName spinnaker.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/spinnaker.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/spinnaker.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9000/
    ProxyPassReverse / http://127.0.0.1:9000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend spinnaker_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/spinnaker.pem
    redirect scheme https if !{ ssl_fc }
    default_backend spinnaker_backend

backend spinnaker_backend
    balance roundrobin
    server spinnaker1 127.0.0.1:9000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R spinnaker:spinnaker /etc/spinnaker
sudo chmod 750 /etc/spinnaker

# Configure firewall
sudo firewall-cmd --permanent --add-port=9000/tcp
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
sudo systemctl status spinnaker

# View logs
sudo journalctl -u spinnaker -f

# Monitor resource usage
top -p $(pgrep spinnaker)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/spinnaker"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/spinnaker-backup-$DATE.tar.gz" /etc/spinnaker /var/lib/spinnaker

echo "Backup completed: $BACKUP_DIR/spinnaker-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop spinnaker

# Restore from backup
tar -xzf /backup/spinnaker/spinnaker-backup-*.tar.gz -C /

# Start service
sudo systemctl start spinnaker
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u spinnaker -n 100
sudo tail -f /var/log/spinnaker/spinnaker.log

# Check configuration
hal version

# Check permissions
ls -la /etc/spinnaker
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9000

# Test connectivity
telnet localhost 9000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep spinnaker)

# Check disk I/O
iotop -p $(pgrep spinnaker)

# Check connections
ss -an | grep 9000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  spinnaker:
    image: spinnaker:latest
    ports:
      - "9000:9000"
    volumes:
      - ./config:/etc/spinnaker
      - ./data:/var/lib/spinnaker
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update spinnaker

# Debian/Ubuntu
sudo apt update && sudo apt upgrade spinnaker

# Arch Linux
sudo pacman -Syu spinnaker

# Alpine Linux
apk update && apk upgrade spinnaker

# openSUSE
sudo zypper update spinnaker

# FreeBSD
pkg update && pkg upgrade spinnaker

# Always backup before updates
tar -czf /backup/spinnaker-pre-update-$(date +%Y%m%d).tar.gz /etc/spinnaker

# Restart after updates
sudo systemctl restart spinnaker
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/spinnaker

# Clean old logs
find /var/log/spinnaker -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/spinnaker
```

## Additional Resources

- Official Documentation: https://docs.spinnaker.org/
- GitHub Repository: https://github.com/spinnaker/spinnaker
- Community Forum: https://forum.spinnaker.org/
- Best Practices Guide: https://docs.spinnaker.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
