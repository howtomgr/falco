# falco Installation Guide

falco is a free and open-source runtime security. Falco provides cloud-native runtime security

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
  - Storage: 10GB for logs
  - Network: gRPC/HTTP
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8765 (default falco port)
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

# Install falco
sudo dnf install -y falco

# Enable and start service
sudo systemctl enable --now falco

# Configure firewall
sudo firewall-cmd --permanent --add-port=8765/tcp
sudo firewall-cmd --reload

# Verify installation
falco --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install falco
sudo apt install -y falco

# Enable and start service
sudo systemctl enable --now falco

# Configure firewall
sudo ufw allow 8765

# Verify installation
falco --version
```

### Arch Linux

```bash
# Install falco
sudo pacman -S falco

# Enable and start service
sudo systemctl enable --now falco

# Verify installation
falco --version
```

### Alpine Linux

```bash
# Install falco
apk add --no-cache falco

# Enable and start service
rc-update add falco default
rc-service falco start

# Verify installation
falco --version
```

### openSUSE/SLES

```bash
# Install falco
sudo zypper install -y falco

# Enable and start service
sudo systemctl enable --now falco

# Configure firewall
sudo firewall-cmd --permanent --add-port=8765/tcp
sudo firewall-cmd --reload

# Verify installation
falco --version
```

### macOS

```bash
# Using Homebrew
brew install falco

# Start service
brew services start falco

# Verify installation
falco --version
```

### FreeBSD

```bash
# Using pkg
pkg install falco

# Enable in rc.conf
echo 'falco_enable="YES"' >> /etc/rc.conf

# Start service
service falco start

# Verify installation
falco --version
```

### Windows

```bash
# Using Chocolatey
choco install falco

# Or using Scoop
scoop install falco

# Verify installation
falco --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/falco

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
falco --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable falco

# Start service
sudo systemctl start falco

# Stop service
sudo systemctl stop falco

# Restart service
sudo systemctl restart falco

# Check status
sudo systemctl status falco

# View logs
sudo journalctl -u falco -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add falco default

# Start service
rc-service falco start

# Stop service
rc-service falco stop

# Restart service
rc-service falco restart

# Check status
rc-service falco status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'falco_enable="YES"' >> /etc/rc.conf

# Start service
service falco start

# Stop service
service falco stop

# Restart service
service falco restart

# Check status
service falco status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start falco
brew services stop falco
brew services restart falco

# Check status
brew services list | grep falco
```

### Windows Service Manager

```powershell
# Start service
net start falco

# Stop service
net stop falco

# Using PowerShell
Start-Service falco
Stop-Service falco
Restart-Service falco

# Check status
Get-Service falco
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream falco_backend {
    server 127.0.0.1:8765;
}

server {
    listen 80;
    server_name falco.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name falco.example.com;

    ssl_certificate /etc/ssl/certs/falco.example.com.crt;
    ssl_certificate_key /etc/ssl/private/falco.example.com.key;

    location / {
        proxy_pass http://falco_backend;
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
    ServerName falco.example.com
    Redirect permanent / https://falco.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName falco.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/falco.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/falco.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8765/
    ProxyPassReverse / http://127.0.0.1:8765/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend falco_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/falco.pem
    redirect scheme https if !{ ssl_fc }
    default_backend falco_backend

backend falco_backend
    balance roundrobin
    server falco1 127.0.0.1:8765 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R falco:falco /etc/falco
sudo chmod 750 /etc/falco

# Configure firewall
sudo firewall-cmd --permanent --add-port=8765/tcp
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
sudo systemctl status falco

# View logs
sudo journalctl -u falco -f

# Monitor resource usage
top -p $(pgrep falco)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/falco"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/falco-backup-$DATE.tar.gz" /etc/falco /var/lib/falco

echo "Backup completed: $BACKUP_DIR/falco-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop falco

# Restore from backup
tar -xzf /backup/falco/falco-backup-*.tar.gz -C /

# Start service
sudo systemctl start falco
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u falco -n 100
sudo tail -f /var/log/falco/falco.log

# Check configuration
falco --version

# Check permissions
ls -la /etc/falco
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8765

# Test connectivity
telnet localhost 8765

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep falco)

# Check disk I/O
iotop -p $(pgrep falco)

# Check connections
ss -an | grep 8765
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  falco:
    image: falco:latest
    ports:
      - "8765:8765"
    volumes:
      - ./config:/etc/falco
      - ./data:/var/lib/falco
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update falco

# Debian/Ubuntu
sudo apt update && sudo apt upgrade falco

# Arch Linux
sudo pacman -Syu falco

# Alpine Linux
apk update && apk upgrade falco

# openSUSE
sudo zypper update falco

# FreeBSD
pkg update && pkg upgrade falco

# Always backup before updates
tar -czf /backup/falco-pre-update-$(date +%Y%m%d).tar.gz /etc/falco

# Restart after updates
sudo systemctl restart falco
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/falco

# Clean old logs
find /var/log/falco -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/falco
```

## Additional Resources

- Official Documentation: https://docs.falco.org/
- GitHub Repository: https://github.com/falco/falco
- Community Forum: https://forum.falco.org/
- Best Practices Guide: https://docs.falco.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
