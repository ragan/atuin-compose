# Atuin Server - Quick Reference

## Setup

```bash
# 1. Copy .env.example to .env and update password
cp .env.example .env
nano .env  # Change ATUIN_DB_PASSWORD

# 2. Create config directory
mkdir config
chown 1000:1000 config

# 3. Start services
docker compose up -d

# 4. Check status
docker compose ps

# 5. View logs
docker compose logs -f atuin
```

## Client Setup

```bash
# On each client machine, add to ~/.config/atuin/config.toml:
sync_address = "http://your-server:8888"

# Register on first machine
atuin register -u username -e email@example.com

# Save your key!
atuin key

# Sync
atuin sync
```

## Useful Commands

```bash
# Stop services
docker compose down

# Start services
docker compose up -d

# View logs
docker compose logs -f

# Restart services
docker compose restart
```

## Documentation

See [SETUP.md](SETUP.md) for complete documentation.
