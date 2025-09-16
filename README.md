# postgres-exporter Installation Guide

postgres-exporter is a free and open-source Prometheus exporter for PostgreSQL metrics. Postgres Exporter connects to PostgreSQL databases and exports metrics for Prometheus monitoring, essential for database observability

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
  - CPU: 1 core minimum
  - RAM: 128MB minimum
  - Storage: 50MB for installation
  - Network: PostgreSQL and HTTP access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9187 (default postgres-exporter port)
  - PostgreSQL connection
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

# Install postgres-exporter
sudo dnf install -y prometheus-postgres-exporter

# Enable and start service
sudo systemctl enable --now postgres_exporter

# Configure firewall
sudo firewall-cmd --permanent --add-port=9187/tcp
sudo firewall-cmd --reload

# Verify installation
postgres_exporter --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install postgres-exporter
sudo apt install -y prometheus-postgres-exporter

# Enable and start service
sudo systemctl enable --now postgres_exporter

# Configure firewall
sudo ufw allow 9187

# Verify installation
postgres_exporter --version
```

### Arch Linux

```bash
# Install postgres-exporter
sudo pacman -S prometheus-postgres-exporter

# Enable and start service
sudo systemctl enable --now postgres_exporter

# Verify installation
postgres_exporter --version
```

### Alpine Linux

```bash
# Install postgres-exporter
apk add --no-cache prometheus-postgres-exporter

# Enable and start service
rc-update add postgres_exporter default
rc-service postgres_exporter start

# Verify installation
postgres_exporter --version
```

### openSUSE/SLES

```bash
# Install postgres-exporter
sudo zypper install -y prometheus-postgres-exporter

# Enable and start service
sudo systemctl enable --now postgres_exporter

# Configure firewall
sudo firewall-cmd --permanent --add-port=9187/tcp
sudo firewall-cmd --reload

# Verify installation
postgres_exporter --version
```

### macOS

```bash
# Using Homebrew
brew install prometheus-postgres-exporter

# Start service
brew services start prometheus-postgres-exporter

# Verify installation
postgres_exporter --version
```

### FreeBSD

```bash
# Using pkg
pkg install prometheus-postgres-exporter

# Enable in rc.conf
echo 'postgres_exporter_enable="YES"' >> /etc/rc.conf

# Start service
service postgres_exporter start

# Verify installation
postgres_exporter --version
```

### Windows

```bash
# Using Chocolatey
choco install prometheus-postgres-exporter

# Or using Scoop
scoop install prometheus-postgres-exporter

# Verify installation
postgres_exporter --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/prometheus-postgres-exporter

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
postgres_exporter --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable postgres_exporter

# Start service
sudo systemctl start postgres_exporter

# Stop service
sudo systemctl stop postgres_exporter

# Restart service
sudo systemctl restart postgres_exporter

# Check status
sudo systemctl status postgres_exporter

# View logs
sudo journalctl -u postgres_exporter -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add postgres_exporter default

# Start service
rc-service postgres_exporter start

# Stop service
rc-service postgres_exporter stop

# Restart service
rc-service postgres_exporter restart

# Check status
rc-service postgres_exporter status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'postgres_exporter_enable="YES"' >> /etc/rc.conf

# Start service
service postgres_exporter start

# Stop service
service postgres_exporter stop

# Restart service
service postgres_exporter restart

# Check status
service postgres_exporter status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start prometheus-postgres-exporter
brew services stop prometheus-postgres-exporter
brew services restart prometheus-postgres-exporter

# Check status
brew services list | grep prometheus-postgres-exporter
```

### Windows Service Manager

```powershell
# Start service
net start postgres_exporter

# Stop service
net stop postgres_exporter

# Using PowerShell
Start-Service postgres_exporter
Stop-Service postgres_exporter
Restart-Service postgres_exporter

# Check status
Get-Service postgres_exporter
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream prometheus-postgres-exporter_backend {
    server 127.0.0.1:9187;
}

server {
    listen 80;
    server_name prometheus-postgres-exporter.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name prometheus-postgres-exporter.example.com;

    ssl_certificate /etc/ssl/certs/prometheus-postgres-exporter.example.com.crt;
    ssl_certificate_key /etc/ssl/private/prometheus-postgres-exporter.example.com.key;

    location / {
        proxy_pass http://prometheus-postgres-exporter_backend;
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
    ServerName prometheus-postgres-exporter.example.com
    Redirect permanent / https://prometheus-postgres-exporter.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName prometheus-postgres-exporter.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/prometheus-postgres-exporter.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/prometheus-postgres-exporter.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9187/
    ProxyPassReverse / http://127.0.0.1:9187/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend prometheus-postgres-exporter_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/prometheus-postgres-exporter.pem
    redirect scheme https if !{ ssl_fc }
    default_backend prometheus-postgres-exporter_backend

backend prometheus-postgres-exporter_backend
    balance roundrobin
    server prometheus-postgres-exporter1 127.0.0.1:9187 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R prometheus-postgres-exporter:prometheus-postgres-exporter /etc/prometheus-postgres-exporter
sudo chmod 750 /etc/prometheus-postgres-exporter

# Configure firewall
sudo firewall-cmd --permanent --add-port=9187/tcp
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
sudo systemctl status postgres_exporter

# View logs
sudo journalctl -u postgres_exporter -f

# Monitor resource usage
top -p $(pgrep prometheus-postgres-exporter)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/prometheus-postgres-exporter"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/prometheus-postgres-exporter-backup-$DATE.tar.gz" /etc/prometheus-postgres-exporter /var/lib/prometheus-postgres-exporter

echo "Backup completed: $BACKUP_DIR/prometheus-postgres-exporter-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop postgres_exporter

# Restore from backup
tar -xzf /backup/prometheus-postgres-exporter/prometheus-postgres-exporter-backup-*.tar.gz -C /

# Start service
sudo systemctl start postgres_exporter
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u postgres_exporter -n 100
sudo tail -f /var/log/prometheus-postgres-exporter/prometheus-postgres-exporter.log

# Check configuration
postgres_exporter --version

# Check permissions
ls -la /etc/prometheus-postgres-exporter
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9187

# Test connectivity
telnet localhost 9187

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep prometheus-postgres-exporter)

# Check disk I/O
iotop -p $(pgrep prometheus-postgres-exporter)

# Check connections
ss -an | grep 9187
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  prometheus-postgres-exporter:
    image: prometheus-postgres-exporter:latest
    ports:
      - "9187:9187"
    volumes:
      - ./config:/etc/prometheus-postgres-exporter
      - ./data:/var/lib/prometheus-postgres-exporter
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update prometheus-postgres-exporter

# Debian/Ubuntu
sudo apt update && sudo apt upgrade prometheus-postgres-exporter

# Arch Linux
sudo pacman -Syu prometheus-postgres-exporter

# Alpine Linux
apk update && apk upgrade prometheus-postgres-exporter

# openSUSE
sudo zypper update prometheus-postgres-exporter

# FreeBSD
pkg update && pkg upgrade prometheus-postgres-exporter

# Always backup before updates
tar -czf /backup/prometheus-postgres-exporter-pre-update-$(date +%Y%m%d).tar.gz /etc/prometheus-postgres-exporter

# Restart after updates
sudo systemctl restart postgres_exporter
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/prometheus-postgres-exporter

# Clean old logs
find /var/log/prometheus-postgres-exporter -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/prometheus-postgres-exporter
```

## Additional Resources

- Official Documentation: https://docs.prometheus-postgres-exporter.org/
- GitHub Repository: https://github.com/prometheus-postgres-exporter/prometheus-postgres-exporter
- Community Forum: https://forum.prometheus-postgres-exporter.org/
- Best Practices Guide: https://docs.prometheus-postgres-exporter.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
