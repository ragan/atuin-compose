# Atuin Self-Hosted Sync Server

Share your shell history across multiple machines using Atuin.

## Server Setup (one time)

```bash
cp .env.example .env
# Set POSTGRES_PASSWORD to something secure in .env
docker compose up -d
```

Done. Server is now running.

## Client Setup (on each machine)

```bash
# Install Atuin
curl --proto='https' --tlsv1.2 -LsSf https://setup.atuin.sh | sh

# Point to your server
echo 'sync_address = "http://YOUR_SERVER_IP:8888"' >> ~/.config/atuin/config.toml

# Register (use SAME username/email on ALL machines)
# You'll be asked to create an account password - use this SAME password on all machines
atuin register -u YOUR_USERNAME -e YOUR_EMAIL

# Import your existing history
atuin import auto

# Sync
atuin sync
```

Add to your `~/.zshrc`:
```bash
eval "$(atuin init zsh)"
```

That's it. Your history is now synced.

## Important Notes

- **Use the SAME username and email on ALL your machines** - this is how they share history
- Replace `YOUR_SERVER_IP` with your server's actual IP (e.g., `192.168.1.100`)
- Open registration is disabled by default for security
- Your history is end-to-end encrypted

## Management

```bash
# View logs
docker compose logs -f

# Stop server
docker compose down

# Start server
docker compose up -d

# Update server
docker compose pull && docker compose up -d
```
