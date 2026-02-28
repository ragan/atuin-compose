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

```bash
# Register with your server (replace with your actual URL)
atuin register -u YOUR_USERNAME -e YOUR_EMAIL -s http://localhost:8888

# Sync history
atuin sync
```

## Services

- **PostgreSQL**: Stores encrypted shell history with health checks
- **Atuin Server**: Handles sync requests and authentication with health checks

## Persistence

Database is stored in a Docker volume: atuin-compose_postgres_data

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
