# GitLab CE Docker Compose Setup

An optimized GitLab Community Edition deployment using Docker Compose with enhanced logging, Git LFS support, and production-ready configurations.

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Initial Setup](#initial-setup)
- [Firewall Configuration](#firewall-configuration)
- [Maintenance](#maintenance)
- [Troubleshooting](#troubleshooting)
- [Security Considerations](#security-considerations)

## 🎯 Overview

This project provides a production-ready GitLab CE setup using Docker Compose with optimized configurations for:

- **Log Management**: Automated log rotation to prevent disk space issues
- **Large File Storage**: Git LFS support for files up to 5GB
- **Persistent Data**: Named volumes for configuration, logs, and data
- **Custom SSH Port**: SSH access on port 2222 to avoid conflicts

## ✨ Features

- **GitLab CE Latest**: Always runs the latest stable GitLab Community Edition
- **Optimized Logging**:
  - Log rotation every 200MB
  - Keeps last 10 log files
  - Daily rotation schedule
  - 7-day retention period
- **Git LFS Support**:
  - Maximum file size: 5GB
  - Nginx configured for large uploads
- **Persistent Storage**: Separate volumes for config, logs, and data
- **Custom SSH Port**: Port 2222 for Git SSH operations
- **Auto-restart**: Container automatically restarts on failure

## 📦 Prerequisites

- **Docker**: Version 20.10 or higher
- **Docker Compose**: Version 2.0 or higher
- **System Requirements**:
  - Minimum 4GB RAM (8GB recommended)
  - 10GB free disk space (more for repositories)
  - Linux-based OS (Ubuntu, Debian, CentOS, etc.)

### Verify Installation

```bash
docker --version
docker compose version
```

## 🚀 Quick Start

### 1. Clone or Download

```bash
git clone <your-repo-url>
cd gitlab-server
```

### 2. Configure Domain/IP

Edit `docker-compose.yml` and replace `<ip>` with your server's IP address or domain name:

```yaml
external_url 'http://192.168.1.100' # Example
```

Also update the hostname:

```yaml
hostname: "gitlab.example.com" # Your domain or IP
```

### 3. Start GitLab

```bash
docker compose up -d
```

### 4. Monitor Startup

GitLab takes 2-5 minutes to fully initialize. Monitor the logs:

```bash
docker compose logs -f gitlab
```

Wait for the message: **"GitLab is operational"**

### 5. Access GitLab

Open your browser and navigate to:

```
http://<your-ip-or-domain>
```

## ⚙️ Configuration

### Environment Variables

The `docker-compose.yml` includes the following optimizations:

| Configuration           | Value               | Description                       |
| ----------------------- | ------------------- | --------------------------------- |
| `external_url`          | `http://<ip>`       | Your GitLab instance URL          |
| `gitlab_shell_ssh_port` | `2222`              | SSH port for Git operations       |
| `svlogd_size`           | `209715200` (200MB) | Max log file size before rotation |
| `svlogd_num`            | `10`                | Number of rotated logs to keep    |
| `logrotate_frequency`   | `daily`             | Log rotation schedule             |
| `logrotate_rotate`      | `7`                 | Days to keep rotated logs         |
| `lfs_enabled`           | `true`              | Enable Git LFS                    |
| `lfs_max_file_size`     | `5120` (5GB)        | Maximum LFS file size             |
| `client_max_body_size`  | `5g`                | Nginx upload limit                |

### Ports

| Port   | Service | Description                                     |
| ------ | ------- | ----------------------------------------------- |
| `80`   | HTTP    | Web interface                                   |
| `443`  | HTTPS   | Secure web interface (configure SSL separately) |
| `2222` | SSH     | Git SSH operations                              |

### Volumes

| Volume          | Mount Point       | Purpose                    |
| --------------- | ----------------- | -------------------------- |
| `gitlab_config` | `/etc/gitlab`     | GitLab configuration files |
| `gitlab_logs`   | `/var/log/gitlab` | Application logs           |
| `gitlab_data`   | `/var/lib/gitlab` | Repositories and database  |

## 🔐 Initial Setup

### 1. Retrieve Root Password

After GitLab starts, get the initial root password:

```bash
sudo docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```

**Note**: This file is automatically deleted 24 hours after installation for security.

### 2. First Login

- **Username**: `root`
- **Password**: (from the command above)
- **URL**: `http://<your-ip-or-domain>`

### 3. Change Root Password

1. Log in as root
2. Go to **User Settings** → **Password**
3. Set a strong new password
4. Save changes

### 4. Create Users/Projects

You can now create users, groups, and projects through the web interface.

## 🔥 Firewall Configuration

If you have UFW (Uncomplicated Firewall) enabled, allow the necessary ports:

```bash
# Allow HTTP
sudo ufw allow 80/tcp

# Allow HTTPS (for future SSL setup)
sudo ufw allow 443/tcp

# Allow GitLab SSH on port 2222
sudo ufw allow 2222/tcp

# Enable the firewall
sudo ufw enable

# Check status
sudo ufw status
```

### For Other Firewalls

**firewalld** (CentOS/RHEL):

```bash
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --permanent --add-port=2222/tcp
sudo firewall-cmd --reload
```

**iptables**:

```bash
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 2222 -j ACCEPT
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```

## 🛠️ Maintenance

### View Logs

```bash
# All logs
docker compose logs -f

# Last 100 lines
docker compose logs --tail=100

# Specific service
docker compose logs -f gitlab
```

### Restart GitLab

```bash
docker compose restart
```

### Stop GitLab

```bash
docker compose down
```

### Update GitLab

```bash
# Pull latest image
docker compose pull

# Recreate container with new image
docker compose up -d
```

### Backup

```bash
# Create backup
sudo docker exec -it gitlab gitlab-backup create

# Backups are stored in the gitlab_data volume
# at /var/lib/gitlab/backups/
```

### Restore from Backup

```bash
# Copy backup file to container
sudo docker cp <backup-file>.tar gitlab:/var/opt/gitlab/backups/

# Stop services
sudo docker exec -it gitlab gitlab-ctl stop puma
sudo docker exec -it gitlab gitlab-ctl stop sidekiq

# Restore
sudo docker exec -it gitlab gitlab-backup restore BACKUP=<timestamp>

# Restart
docker compose restart
```

## 🔍 Troubleshooting

### GitLab Not Starting

```bash
# Check container status
docker compose ps

# View detailed logs
docker compose logs gitlab

# Check system resources
docker stats gitlab
```

### Cannot Access Web Interface

1. Verify container is running: `docker compose ps`
2. Check firewall rules: `sudo ufw status`
3. Verify external_url in config matches your access URL
4. Check logs: `docker compose logs gitlab`

### SSH Clone Not Working

Ensure you're using port 2222:

```bash
# Correct format
git clone ssh://git@<your-domain>:2222/username/project.git

# Or configure SSH config
cat ~/.ssh/config
Host gitlab.example.com
    Port 2222
```

### Disk Space Issues

Check Docker volume usage:

```bash
docker system df -v
```

Clean up old logs manually:

```bash
sudo docker exec -it gitlab find /var/log/gitlab -name "*.log.*" -mtime +7 -delete
```

### Performance Issues

Increase container resources in `docker-compose.yml`:

```yaml
services:
  gitlab:
    deploy:
      resources:
        limits:
          memory: 8G
        reservations:
          memory: 4G
```

## 🔒 Security Considerations

### 1. Enable HTTPS

For production, configure SSL/TLS:

```yaml
external_url 'https://<your-domain>'
```

Mount SSL certificates:

```yaml
volumes:
  - "./ssl:/etc/gitlab/ssl"
```

### 2. Change Default Ports

Consider using non-standard ports to reduce automated attacks.

### 3. Regular Updates

Keep GitLab updated:

```bash
docker compose pull && docker compose up -d
```

### 4. Strong Passwords

Enforce strong password policies in **Admin Area** → **Settings** → **General** → **Sign-up restrictions**.

### 5. Two-Factor Authentication

Enable 2FA for all users, especially administrators.

### 6. Backup Strategy

Automate regular backups:

```bash
# Add to crontab
0 2 * * * docker exec gitlab gitlab-backup create CRON=1
```

### 7. Network Security

- Use a reverse proxy (Nginx, Traefik) for SSL termination
- Implement rate limiting
- Use a VPN for administrative access

## 📚 Additional Resources

- [GitLab Documentation](https://docs.gitlab.com/)
- [GitLab Docker Images](https://docs.gitlab.com/ee/install/docker.html)
- [GitLab Backup/Restore](https://docs.gitlab.com/ee/raketasks/backup_restore.html)
- [GitLab Configuration Options](https://docs.gitlab.com/omnibus/settings/configuration.html)

## 📝 License

This setup configuration is provided as-is for educational and production use.

---

**Note**: Replace `<ip>` and `<your-domain>` with your actual server IP address or domain name throughout this documentation.
