# Atuin Server - Docker Compose Setup

This directory contains the setup for self-hosting Atuin sync server using Docker Compose with SQLite database.

## Quick Start

1. Create `.env` file with user/group IDs
2. Run `docker compose up -d`
3. Configure clients to sync with your server

## Files

### `.env` - Environment Variables

```bash
# User/Group ID for container permissions
UID=1000
GID=1000
```

### `docker-compose.yml` - Service Definitions

```yaml
services:
  atuin:
    restart: always
    image: ghcr.io/atuinsh/atuin:18.3.0
    command: server start
    user: "${UID:-1000}:${GID:-1000}"
    volumes:
      - "./config:/config"
    ports:
      - 8888:8888
    env_file:
      - .env
    environment:
      ATUIN_HOST: "0.0.0.0"
      ATUIN_OPEN_REGISTRATION: "true"
      ATUIN_DB_URI: sqlite:///config/atuin.db
      RUST_LOG: info,atuin_server=debug
```

## Setup Instructions

### 1. Prepare Directories

```bash
mkdir config
```

### 2. Configure Environment

```bash
# Copy example file and update UID/GID
cp .env.example .env
nano .env
# Set UID and GID to your user IDs: id -u and id -g
```

### 3. Start Services

```bash
docker compose up -d
```

### 4. Configure Clients

On each client machine, update `~/.config/atuin/config.toml`:

```toml
sync_address = "https://your-server.com:8888"
```

### 5. Register/Login

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

## Configuration Reference

### Server Config Options

- `ATUIN_HOST`: The host to listen on (default: 127.0.0.1)
- `ATUIN_PORT`: TCP port to listen on (default: 8888)
- `ATUIN_OPEN_REGISTRATION`: Accept new user registrations (default: false)
- `ATUIN_DB_URI`: SQLite path or PostgreSQL URI

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
- Use reverse proxy (nginx/caddy) for HTTPS in production
- Backup the SQLite database regularly: `/config/atuin.db`
- Store encryption key in password manager

## Troubleshooting

### Sync Fails
- Check server is running: `docker compose ps`
- Check server logs: `docker compose logs atuin`
- Verify sync_address in client config
- Check network connectivity

### Database Issues
- Check database file exists: `ls -la config/atuin.db`
- Check file permissions on config directory
- Verify server logs for database errors

### Permissions Issues
- Ensure config directory has proper ownership
- Check UID/GID in .env match your user IDs

## Resources

- [Atuin Documentation](https://docs.atuin.sh)
- [GitHub Repository](https://github.com/atuinsh/atuin)
- [Releases](https://github.com/atuinsh/atuin/releases)
- [Community Forum](https://forum.atuin.sh)
- [Discord](https://discord.gg/jR3tfchVvW)
