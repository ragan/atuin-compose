# Atuin Server - Docker Compose Setup

This directory contains the setup for self-hosting Atuin sync server using Docker Compose.

## Quick Start

1. Create `.env` file with database credentials
2. Create `docker-compose.yml` with service definitions
3. Run `docker compose up -d`
4. Configure clients to sync with your server

## Files

### `.env` - Environment Variables

```bash
ATUIN_DB_NAME=atuin
ATUIN_DB_USERNAME=atuin
# Choose your own secure password. Stick to [A-Za-z0-9.~_-]
ATUIN_DB_PASSWORD=really-insecure
```

### `docker-compose.yml` - Service Definitions

```yaml
services:
  atuin:
    restart: always
    image: ghcr.io/atuinsh/atuin:<LATEST TAGGED RELEASE>
    command: start
    volumes:
      - "./config:/config"
    ports:
      - 8888:8888
    environment:
      ATUIN_HOST: "0.0.0.0"
      ATUIN_OPEN_REGISTRATION: "true"
      ATUIN_DB_URI: postgres://${ATUIN_DB_USERNAME}:${ATUIN_DB_PASSWORD}@db/${ATUIN_DB_NAME}
      RUST_LOG: info,atuin_server=debug
    depends_on:
      - db
  db:
    image: postgres:18
    restart: unless-stopped
    volumes:
      - "./database:/var/lib/postgresql/data/"
    environment:
      POSTGRES_USER: ${ATUIN_DB_USERNAME}
      POSTGRES_PASSWORD: ${ATUIN_DB_PASSWORD}
      POSTGRES_DB: ${ATUIN_DB_NAME}
```

## Setup Instructions

### 1. Prepare Directories

```bash
mkdir config
chown 1000:1000 config
```

### 2. Start Services

```bash
docker compose up -d
```

### 3. Configure Clients

On each client machine, update `~/.config/atuin/config.toml`:

```toml
sync_address = "https://your-server.com:8888"
```

### 4. Register/Login

**First machine:**
```bash
atuin register -u <USERNAME> -e <EMAIL>
atuin key  # Save this key!
atuin sync
```

**Additional machines:**
```bash
atuin login -u <USERNAME>
# Enter password + key from first machine
atuin sync
```

## What Happens If Sync Server Is Down?

### Local Operations Continue Normally

- ✅ Commands are still recorded to local SQLite database
- ✅ History search works with local data
- ✅ No interruption to shell functionality

### Sync Behavior

- Auto-sync fails silently (based on 30s network timeout)
- When server is back online, sync resumes automatically
- Run `atuin sync` manually to trigger sync when server is up
- All local data is preserved and will sync when connection restores

### Important Notes

- Atuin is designed to be fault-tolerant
- The sync server is only for backup and cross-machine sync
- Each machine works independently with its local database
- **Never lose your encryption key** (`atuin key`) - you can't recover synced history without it

## Optional: Systemd Service

Create `/etc/systemd/system/atuin.service`:

```ini
[Unit]
Description=Docker Compose Atuin Service
Requires=docker.service
After=docker.service

[Service]
# Where the docker-compose file is located
WorkingDirectory=/path/to/atuin-compose
ExecStart=/usr/bin/docker compose up
ExecStop=/usr/bin/docker compose down
TimeoutStartSec=0
Restart=on-failure
StartLimitBurst=3

[Install]
WantedBy=multi-user.target
```

Start and enable:
```bash
systemctl enable --now atuin
systemctl status atuin
```

## Optional: Database Backups

Add to `docker-compose.yml`:

```yaml
backup:
  container_name: atuin_db_dumper
  image: prodrigestivill/postgres-backup-local
  env_file:
    - .env
  environment:
    POSTGRES_HOST: postgresql
    POSTGRES_DB: ${ATUIN_DB_NAME}
    POSTGRES_USER: ${ATUIN_DB_USERNAME}
    POSTGRES_PASSWORD: ${ATUIN_DB_PASSWORD}
    SCHEDULE: "@daily"
    BACKUP_DIR: /db_dumps
  volumes:
    - ./db_dumps:/db_dumps
  depends_on:
    - postgresql
```

## Configuration Reference

### Server Config Options

- `ATUIN_HOST`: The host to listen on (default: 127.0.0.1)
- `ATUIN_PORT`: TCP port to listen on (default: 8888)
- `ATUIN_OPEN_REGISTRATION`: Accept new user registrations (default: false)
- `ATUIN_DB_URI`: PostgreSQL URI or SQLite path

### Client Config Options

- `sync_address`: Server URL (default: https://api.atuin.sh)
- `sync_frequency`: How often to auto-sync (default: 1h)
- `auto_sync`: Enable/disable auto-sync (default: true)
- `db_path`: Local database path (default: ~/.local/share/atuin/history.db)

### Useful Commands

```bash
# View your encryption key
atuin key

# Manual sync
atuin sync

# Force full sync
atuin sync -f

# Delete your account (deletes server data, local data remains)
atuin account delete

# Logout
atuin logout

# Check client info
atuin info
```

## Important Security Notes

- All history is end-to-end encrypted
- Server operators cannot see your data
- Never share your encryption key
- Use strong passwords for database
- Use reverse proxy (nginx/caddy) for HTTPS in production
- Backup your database regularly
- Store encryption key in password manager

## Troubleshooting

### Sync Fails
- Check server is running: `docker compose ps`
- Check server logs: `docker compose logs atuin`
- Verify sync_address in client config
- Check network connectivity

### Database Issues
- Check database is running: `docker compose ps db`
- Check database logs: `docker compose logs db`
- Verify database credentials in .env

### Permissions Issues
- Ensure config directory is owned by UID 1000
- Check database volume permissions

## Resources

- [Atuin Documentation](https://docs.atuin.sh)
- [GitHub Repository](https://github.com/atuinsh/atuin)
- [Releases](https://github.com/atuinsh/atuin/releases)
- [Community Forum](https://forum.atuin.sh)
- [Discord](https://discord.gg/jR3tfchVvW)
