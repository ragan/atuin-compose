# Atuin Self-Hosted Sync Server

Self-hosted Atuin sync server using Docker Compose with PostgreSQL.

## Quick Start

```bash
# Copy and configure environment file
cp .env.example .env
# IMPORTANT: Set POSTGRES_PASSWORD to a strong password in .env

# Start the server
docker compose up -d

# Check status
docker compose ps

# View logs
docker compose logs -f
```

## Configuration

- **Server URL**: http://localhost:8888
- **Database**: PostgreSQL 16 (managed by Docker)
- **Registration**: Closed by default (must be explicitly enabled)
- **Config**: Environment variables in `.env` file

## Environment Variables

Key variables in `.env`:

- `POSTGRES_PASSWORD` (required): Strong database password (must be set)
- `POSTGRES_DB`: Database name (default: atuin)
- `POSTGRES_USER`: Database user (default: atuin)
- `ATUIN_OPEN_REGISTRATION`: Enable open registration (default: false)
- `UID/GID`: Container user permissions (default: 1000:1000)

## Client Setup

### Install Atuin Client

```bash
# Linux/macOS
curl --proto='https' --tlsv1.2 -LsSf https://setup.atuin.sh | sh

# Or visit https://github.com/atuinsh/atuin for other installation methods
```

### Register with Server

```bash
# Replace with your actual server URL, username, and email
# Use your server's IP or domain if accessing from another machine
atuin register -u YOUR_USERNAME -e YOUR_EMAIL -s http://localhost:8888
```

### Import and Sync Existing History

```bash
# Import existing shell history
atuin import auto

# Sync to server
atuin sync
```

### Configure Shell Integration

Add to your shell configuration file (`~/.bashrc`, `~/.zshrc`, or `~/.config/fish/config.fish`):

```bash
# For Bash
eval "$(atuin init bash)"

# For Zsh
eval "$(atuin init zsh)"

# For Fish
atuin init fish | source
```

### Multi-Machine Setup

To sync history across multiple machines:

1. Install Atuin on each machine using the steps above
2. Register with the same credentials on each machine
3. Import existing history from each machine
4. Atuin will automatically merge and sync history

### Automatic Syncing

Atuin automatically syncs when you run commands. To manually sync:

```bash
atuin sync
```

## Services

- **PostgreSQL**: Stores encrypted shell history with health checks
- **Atuin Server**: Handles sync requests and authentication with health checks

## Persistence

Database is stored in a Docker volume: postgres_data

## Logs

```bash
# View all logs
docker compose logs -f

# View specific service logs
docker compose logs -f atuin
docker compose logs -f postgres
```

## Stop

```bash
docker compose down
```

## Update

```bash
docker compose pull
docker compose up -d
```
