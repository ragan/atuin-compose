# Atuin Self-Hosted Sync Server

This repository provides a Docker Compose setup for running a self-hosted Atuin sync server with PostgreSQL backend. Atuin is a shell history synchronization tool that enables sharing command history across multiple machines.

## Repository Type
Docker Compose project for deploying a self-hosted Atuin server with PostgreSQL database.

## Build/Lint/Test Commands
No specific build, lint, or test commands required as this is a deployment configuration.

## Code Style Guidelines
- Configuration files use standard Docker Compose syntax
- Environment variables follow Atuin project conventions with `ATUIN_` prefix
- Service names use lowercase with hyphens for consistency
- Database credentials follow PostgreSQL standard naming conventions

## Project Structure
- `docker-compose.yml` - Main Docker configuration with postgres and atuin services
- `.env.example` - Example environment variables file
- `.env` - Current environment configuration (UID/GID settings)
- `atuin.service` - Optional systemd service for automatic startup
- `README.md` - Project documentation and usage instructions
- `SETUP.md` - Detailed setup and configuration guide
- `config/` - Empty directory for configuration files (mounted as volume)

## Working with This Repository
This is a deployment-ready project that uses Docker Compose. The configuration:
1. Sets up PostgreSQL 16 with health checks
2. Configures Atuin server version 18.3.0
3. Uses environment variables for flexible configuration
4. Mounts a volume for persistent database storage
5. Implements proper service dependencies and restart policies

## Additional Notes
- The `config/` directory is intended for configuration files but is currently empty
- Database is stored in a Docker volume named `atuin-compose_postgres_data`
- Open registration is enabled by default (can be configured via environment variable)
- Default database password should be changed for production use
- Requires Docker and Docker Compose to be installed on the host system
- Server listens on port 8888 by default (configurable)