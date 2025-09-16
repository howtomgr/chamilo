# chamilo Installation Guide

chamilo is a free and open-source e-learning platform. Chamilo provides e-learning and collaboration platform

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
  - Storage: 20GB for courses
  - Network: HTTP/HTTPS
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default chamilo port)
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

# Install chamilo
sudo dnf install -y chamilo

# Enable and start service
sudo systemctl enable --now chamilo

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
chamilo --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install chamilo
sudo apt install -y chamilo

# Enable and start service
sudo systemctl enable --now chamilo

# Configure firewall
sudo ufw allow 80

# Verify installation
chamilo --version
```

### Arch Linux

```bash
# Install chamilo
sudo pacman -S chamilo

# Enable and start service
sudo systemctl enable --now chamilo

# Verify installation
chamilo --version
```

### Alpine Linux

```bash
# Install chamilo
apk add --no-cache chamilo

# Enable and start service
rc-update add chamilo default
rc-service chamilo start

# Verify installation
chamilo --version
```

### openSUSE/SLES

```bash
# Install chamilo
sudo zypper install -y chamilo

# Enable and start service
sudo systemctl enable --now chamilo

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
chamilo --version
```

### macOS

```bash
# Using Homebrew
brew install chamilo

# Start service
brew services start chamilo

# Verify installation
chamilo --version
```

### FreeBSD

```bash
# Using pkg
pkg install chamilo

# Enable in rc.conf
echo 'chamilo_enable="YES"' >> /etc/rc.conf

# Start service
service chamilo start

# Verify installation
chamilo --version
```

### Windows

```bash
# Using Chocolatey
choco install chamilo

# Or using Scoop
scoop install chamilo

# Verify installation
chamilo --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/chamilo

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
chamilo --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable chamilo

# Start service
sudo systemctl start chamilo

# Stop service
sudo systemctl stop chamilo

# Restart service
sudo systemctl restart chamilo

# Check status
sudo systemctl status chamilo

# View logs
sudo journalctl -u chamilo -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add chamilo default

# Start service
rc-service chamilo start

# Stop service
rc-service chamilo stop

# Restart service
rc-service chamilo restart

# Check status
rc-service chamilo status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'chamilo_enable="YES"' >> /etc/rc.conf

# Start service
service chamilo start

# Stop service
service chamilo stop

# Restart service
service chamilo restart

# Check status
service chamilo status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start chamilo
brew services stop chamilo
brew services restart chamilo

# Check status
brew services list | grep chamilo
```

### Windows Service Manager

```powershell
# Start service
net start chamilo

# Stop service
net stop chamilo

# Using PowerShell
Start-Service chamilo
Stop-Service chamilo
Restart-Service chamilo

# Check status
Get-Service chamilo
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream chamilo_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name chamilo.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name chamilo.example.com;

    ssl_certificate /etc/ssl/certs/chamilo.example.com.crt;
    ssl_certificate_key /etc/ssl/private/chamilo.example.com.key;

    location / {
        proxy_pass http://chamilo_backend;
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
    ServerName chamilo.example.com
    Redirect permanent / https://chamilo.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName chamilo.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/chamilo.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/chamilo.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend chamilo_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/chamilo.pem
    redirect scheme https if !{ ssl_fc }
    default_backend chamilo_backend

backend chamilo_backend
    balance roundrobin
    server chamilo1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R chamilo:chamilo /etc/chamilo
sudo chmod 750 /etc/chamilo

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
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
sudo systemctl status chamilo

# View logs
sudo journalctl -u chamilo -f

# Monitor resource usage
top -p $(pgrep chamilo)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/chamilo"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/chamilo-backup-$DATE.tar.gz" /etc/chamilo /var/lib/chamilo

echo "Backup completed: $BACKUP_DIR/chamilo-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop chamilo

# Restore from backup
tar -xzf /backup/chamilo/chamilo-backup-*.tar.gz -C /

# Start service
sudo systemctl start chamilo
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u chamilo -n 100
sudo tail -f /var/log/chamilo/chamilo.log

# Check configuration
chamilo --version

# Check permissions
ls -la /etc/chamilo
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80

# Test connectivity
telnet localhost 80

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep chamilo)

# Check disk I/O
iotop -p $(pgrep chamilo)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  chamilo:
    image: chamilo:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/chamilo
      - ./data:/var/lib/chamilo
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update chamilo

# Debian/Ubuntu
sudo apt update && sudo apt upgrade chamilo

# Arch Linux
sudo pacman -Syu chamilo

# Alpine Linux
apk update && apk upgrade chamilo

# openSUSE
sudo zypper update chamilo

# FreeBSD
pkg update && pkg upgrade chamilo

# Always backup before updates
tar -czf /backup/chamilo-pre-update-$(date +%Y%m%d).tar.gz /etc/chamilo

# Restart after updates
sudo systemctl restart chamilo
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/chamilo

# Clean old logs
find /var/log/chamilo -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/chamilo
```

## Additional Resources

- Official Documentation: https://docs.chamilo.org/
- GitHub Repository: https://github.com/chamilo/chamilo
- Community Forum: https://forum.chamilo.org/
- Best Practices Guide: https://docs.chamilo.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
