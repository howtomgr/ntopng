# ntopng Installation Guide

ntopng is a free and open-source network traffic probe. ntopng provides web-based traffic analysis and flow collection

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
  - Storage: 10GB for flows
  - Network: HTTP/NetFlow
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 3000 (default ntopng port)
  - None
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

# Install ntopng
sudo dnf install -y ntopng

# Enable and start service
sudo systemctl enable --now ntopng

# Configure firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload

# Verify installation
ntopng --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install ntopng
sudo apt install -y ntopng

# Enable and start service
sudo systemctl enable --now ntopng

# Configure firewall
sudo ufw allow 3000

# Verify installation
ntopng --version
```

### Arch Linux

```bash
# Install ntopng
sudo pacman -S ntopng

# Enable and start service
sudo systemctl enable --now ntopng

# Verify installation
ntopng --version
```

### Alpine Linux

```bash
# Install ntopng
apk add --no-cache ntopng

# Enable and start service
rc-update add ntopng default
rc-service ntopng start

# Verify installation
ntopng --version
```

### openSUSE/SLES

```bash
# Install ntopng
sudo zypper install -y ntopng

# Enable and start service
sudo systemctl enable --now ntopng

# Configure firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload

# Verify installation
ntopng --version
```

### macOS

```bash
# Using Homebrew
brew install ntopng

# Start service
brew services start ntopng

# Verify installation
ntopng --version
```

### FreeBSD

```bash
# Using pkg
pkg install ntopng

# Enable in rc.conf
echo 'ntopng_enable="YES"' >> /etc/rc.conf

# Start service
service ntopng start

# Verify installation
ntopng --version
```

### Windows

```bash
# Using Chocolatey
choco install ntopng

# Or using Scoop
scoop install ntopng

# Verify installation
ntopng --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/ntopng

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
ntopng --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable ntopng

# Start service
sudo systemctl start ntopng

# Stop service
sudo systemctl stop ntopng

# Restart service
sudo systemctl restart ntopng

# Check status
sudo systemctl status ntopng

# View logs
sudo journalctl -u ntopng -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add ntopng default

# Start service
rc-service ntopng start

# Stop service
rc-service ntopng stop

# Restart service
rc-service ntopng restart

# Check status
rc-service ntopng status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'ntopng_enable="YES"' >> /etc/rc.conf

# Start service
service ntopng start

# Stop service
service ntopng stop

# Restart service
service ntopng restart

# Check status
service ntopng status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start ntopng
brew services stop ntopng
brew services restart ntopng

# Check status
brew services list | grep ntopng
```

### Windows Service Manager

```powershell
# Start service
net start ntopng

# Stop service
net stop ntopng

# Using PowerShell
Start-Service ntopng
Stop-Service ntopng
Restart-Service ntopng

# Check status
Get-Service ntopng
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream ntopng_backend {
    server 127.0.0.1:3000;
}

server {
    listen 80;
    server_name ntopng.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name ntopng.example.com;

    ssl_certificate /etc/ssl/certs/ntopng.example.com.crt;
    ssl_certificate_key /etc/ssl/private/ntopng.example.com.key;

    location / {
        proxy_pass http://ntopng_backend;
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
    ServerName ntopng.example.com
    Redirect permanent / https://ntopng.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName ntopng.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/ntopng.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/ntopng.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:3000/
    ProxyPassReverse / http://127.0.0.1:3000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend ntopng_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/ntopng.pem
    redirect scheme https if !{ ssl_fc }
    default_backend ntopng_backend

backend ntopng_backend
    balance roundrobin
    server ntopng1 127.0.0.1:3000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R ntopng:ntopng /etc/ntopng
sudo chmod 750 /etc/ntopng

# Configure firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
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
sudo systemctl status ntopng

# View logs
sudo journalctl -u ntopng -f

# Monitor resource usage
top -p $(pgrep ntopng)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/ntopng"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/ntopng-backup-$DATE.tar.gz" /etc/ntopng /var/lib/ntopng

echo "Backup completed: $BACKUP_DIR/ntopng-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop ntopng

# Restore from backup
tar -xzf /backup/ntopng/ntopng-backup-*.tar.gz -C /

# Start service
sudo systemctl start ntopng
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u ntopng -n 100
sudo tail -f /var/log/ntopng/ntopng.log

# Check configuration
ntopng --version

# Check permissions
ls -la /etc/ntopng
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 3000

# Test connectivity
telnet localhost 3000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep ntopng)

# Check disk I/O
iotop -p $(pgrep ntopng)

# Check connections
ss -an | grep 3000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  ntopng:
    image: ntopng:latest
    ports:
      - "3000:3000"
    volumes:
      - ./config:/etc/ntopng
      - ./data:/var/lib/ntopng
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update ntopng

# Debian/Ubuntu
sudo apt update && sudo apt upgrade ntopng

# Arch Linux
sudo pacman -Syu ntopng

# Alpine Linux
apk update && apk upgrade ntopng

# openSUSE
sudo zypper update ntopng

# FreeBSD
pkg update && pkg upgrade ntopng

# Always backup before updates
tar -czf /backup/ntopng-pre-update-$(date +%Y%m%d).tar.gz /etc/ntopng

# Restart after updates
sudo systemctl restart ntopng
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/ntopng

# Clean old logs
find /var/log/ntopng -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/ntopng
```

## Additional Resources

- Official Documentation: https://docs.ntopng.org/
- GitHub Repository: https://github.com/ntopng/ntopng
- Community Forum: https://forum.ntopng.org/
- Best Practices Guide: https://docs.ntopng.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
