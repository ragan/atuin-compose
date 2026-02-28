# Atuin Self-Hosted Sync Server

Self-hosted Atuin sync server using Docker Compose with PostgreSQL.

## Quick Start

```bash
# Configure user ID and group ID
cp .env.example .env
# Edit .env if needed (defaults to UID=1000, GID=1000)

# Start the server
docker compose up -d

# Check status
docker compose ps

# View logs
docker compose logs -f
```

## Configuration

- **Server URL**: http://services.lan:8888
- **Database**: PostgreSQL 16 (managed by Docker)
- **Registration**: Open (anyone can register)

## Client Setup

```bash
# Register with your server
atuin register -u YOUR_USERNAME -e YOUR_EMAIL -s http://services.lan:8888

# Sync history
atuin sync
```

## Services

- **PostgreSQL**: Stores encrypted shell history
- **Atuin Server**: Handles sync requests and authentication

## Persistence

Database is stored in a Docker volume: atuin-compose_postgres_data

## Logs

```bash
docker compose logs -f
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
