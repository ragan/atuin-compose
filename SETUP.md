# Atuin Server Setup Guide

This guide covers setting up a self-hosted Atuin sync server using Docker Compose with PostgreSQL.

## Prerequisites

- Docker and Docker Compose installed
- Git installed (optional, for cloning this repository)
- Network access to host the server

## Quick Setup

### 1. Clone or Download Repository

```bash
# If using git
git clone git@nas.hjkl.am:kpeek/atuin-compose.git
cd atuin-compose

# Or download and extract manually
```

### 2. Configure Environment

```bash
# Copy example environment file
cp .env.example .env

# Check current user ID (if needed)
id -u
id -g

# Edit .env if your UID/GID is not 1000:1000
nano .env
```

### 3. Start Services

```bash
docker compose up -d
```

This starts:
- PostgreSQL 16 database server
- Atuin sync server

### 4. Verify Services

```bash
# Check service status
docker compose ps

# Check logs
docker compose logs -f

# Test server (should return JSON response)
curl http://localhost:8888
```

## Configuration

### Environment Variables (.env)

```bash
# User/Group ID for container permissions
UID=1000
GID=1000

# PostgreSQL Configuration
POSTGRES_DB=atuin
POSTGRES_USER=atuin
POSTGRES_PASSWORD=CHANGE_THIS_STRONG_PASSWORD

# Atuin Server Configuration
ATUIN_OPEN_REGISTRATION=false
```

### Docker Compose Configuration

The docker-compose.yml file configures:

- **PostgreSQL Service**:
  - Image: postgres:16-alpine
  - Database: atuin
  - User: atuin
  - Password: Set via POSTGRES_PASSWORD environment variable
  - Health check enabled
  - Resource limits: 512MB memory, 1 CPU
  
- **Atuin Service**:
  - Image: ghcr.io/atuinsh/atuin:18.3.0
  - Host: 0.0.0.0
  - Port: 8888
  - Open registration: disabled by default (set via ATUIN_OPEN_REGISTRATION)
  - Health check enabled
  - Resource limits: 512MB memory, 1 CPU
  - Depends on PostgreSQL (waits for healthy status)

### Server Configuration (config/server.toml)

```toml
host = "0.0.0.0"
port = 8888
open_registration = false
```

Database connection is set via environment variable ATUIN_DB_URI, which uses credentials from .env file.

## Client Setup

### Install Atuin Client

```bash
# Linux/macOS
curl --proto='https' --tlsv1.2 -LsSf https://setup.atuin.sh | sh

# Or manually install
# Check https://github.com/atuinsh/atuin for other methods
```

### Register with Server

```bash
# Replace with your actual server URL, username, and email
# Use localhost:8888 for local access or your server's IP/domain for remote access
atuin register -u YOUR_USERNAME -e YOUR_EMAIL -s http://localhost:8888
```

### Import Existing History

```bash
# Import from shell history
atuin import auto

# Sync to server
atuin sync
```

### Configure Shell

Add the following to your shell configuration (e.g., ~/.bashrc, ~/.zshrc):

```bash
# For Bash
eval "$(atuin init bash)"

# For Zsh
eval "$(atuin init zsh)"

# For Fish
atuin init fish | source
```

## Maintenance

### View Logs

```bash
docker compose logs -f
```

### Stop Services

```bash
docker compose down
```

### Restart Services

```bash
docker compose restart
```

### Update Atuin

```bash
# Pull latest images
docker compose pull

# Restart services
docker compose up -d
```

### Backup Database

```bash
# Backup PostgreSQL database
docker exec atuin-compose-postgres-1 pg_dump -U atuin atuin > backup.sql

# Restore from backup
docker exec -i atuin-compose-postgres-1 psql -U atuin atuin < backup.sql
```

### Clean Up

```bash
# Stop and remove containers
docker compose down

# Remove volumes (WARNING: deletes all data)
docker compose down -v
```

## Security Considerations

- **Database Password**: REQUIRED - Set POSTGRES_PASSWORD to a strong password in .env before starting services
- **Open Registration**: Disabled by default. Enable only by setting ATUIN_OPEN_REGISTRATION=true in .env
- **TLS/SSL**: Consider adding reverse proxy with TLS (nginx, traefik, etc.) for external access
- **Firewall**: Ensure port 8888 is accessible only from trusted networks
- **Resource Limits**: Containers are configured with memory and CPU limits to prevent resource exhaustion
- **Health Checks**: Both services have health checks for better monitoring and reliability

## Troubleshooting

### Container Won't Start

```bash
# Check logs
docker compose logs

# Check permissions
ls -la config/

# Verify .env file exists
cat .env
```

### Database Connection Issues

```bash
# Check PostgreSQL is running
docker compose ps postgres

# Check PostgreSQL logs
docker compose logs postgres

# Test database connection
docker exec -it atuin-compose-postgres-1 psql -U atuin -d atuin
```

### Permission Errors

If you get permission errors, ensure:
1. UID/GID in .env matches your user
2. config/ directory is owned by your user
3. Docker has proper permissions

```bash
# Fix permissions if needed
sudo chown -R ${UID}:${GID} config/
chmod 755 config/
```

### Server Not Responding

```bash
# Check server is running
docker compose ps

# Test connectivity
curl -v http://localhost:8888

# Check firewall rules
sudo ufw status
```

## Advanced Configuration

### Disable Open Registration

Edit docker-compose.yml:

```yaml
environment:
  ATUIN_OPEN_REGISTRATION: "false"
```

### Use External PostgreSQL

Edit docker-compose.yml to remove PostgreSQL service and update:

```yaml
environment:
  ATUIN_DB_URI: postgres://user:password@external-host:5432/atuin
```

### Add TLS/SSL

Add a reverse proxy (nginx, traefik) with SSL certificates.

## Network Access

To access the server from other machines:

1. **Local Network**: Use the server's local IP (e.g., http://192.168.1.100:8888)
2. **DNS**: Add DNS entry for your domain
3. **Public Access**: Use a reverse proxy with TLS and proper security measures

## Systemd Service (Optional)

For automatic startup, use the provided atuin.service file:

```bash
# Copy service file
sudo cp atuin.service /etc/systemd/system/

# Enable and start
sudo systemctl enable atuin
sudo systemctl start atuin
```

Note: The systemd service assumes docker compose is already running. You may need to adjust the ExecStart path.

## Support

- Atuin Documentation: https://docs.atuin.sh
- Atuin GitHub: https://github.com/atuinsh/atuin
- Atuin Forum: https://forum.atuin.sh
