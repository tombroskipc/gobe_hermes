# gobe_hermes

Docker Compose setup for running NousResearch Hermes Agent with persistent data.

## Requirements

- Docker Engine or Docker Desktop
- Docker Compose v2

## Quick Start

Create a local environment file:

```bash
cp .env.example .env
```

Start Hermes:

```bash
HERMES_UID=$(id -u) HERMES_GID=$(id -g) docker compose up -d
```

Open the dashboard:

```text
http://127.0.0.1:9119
```

Follow logs:

```bash
docker compose logs -f hermes
```

Stop Hermes:

```bash
docker compose down
```

`docker compose down` keeps the Docker volume. Do not run `docker compose down -v`
unless you intentionally want to delete Hermes data.

## Services

| Service | Purpose | Local URL |
| --- | --- | --- |
| `hermes` | Hermes gateway and dashboard | `http://127.0.0.1:9119` |
| `hermes` | Gateway API port | `http://127.0.0.1:8642` |

By default, ports bind to `127.0.0.1` only. Keep that default unless the service
is behind a reverse proxy or tunnel with authentication.

## Persistent Volume

The compose file creates a Docker named volume:

```text
gobe_hermes_data -> /opt/data
```

Inspect it:

```bash
docker volume inspect gobe_hermes_data
```

Back it up:

```bash
mkdir -p backups
docker run --rm \
  -v gobe_hermes_data:/data:ro \
  -v "$PWD/backups:/backup" \
  alpine \
  tar czf "/backup/hermes-data-$(date +%Y%m%d-%H%M%S).tgz" -C /data .
```

Restore from a backup:

```bash
docker compose down
docker run --rm \
  -v gobe_hermes_data:/data \
  -v "$PWD/backups:/backup" \
  alpine \
  sh -c 'cd /data && tar xzf /backup/REPLACE_WITH_BACKUP_FILE.tgz'
HERMES_UID=$(id -u) HERMES_GID=$(id -g) docker compose up -d
```

## Configuration

Edit `.env` to change ports, image tag, volume name, or optional provider
tokens. Common values:

```dotenv
HERMES_BIND_HOST=127.0.0.1
HERMES_GATEWAY_PORT=8642
HERMES_DASHBOARD_PORT=9119
HERMES_VOLUME_NAME=gobe_hermes_data
```

To update Hermes:

```bash
docker compose pull
HERMES_UID=$(id -u) HERMES_GID=$(id -g) docker compose up -d
```

## Useful Commands

```bash
docker compose ps
docker compose logs --tail=100 hermes
docker compose exec hermes hermes --help
docker inspect gobe-hermes --format '{{json .Mounts}}'
```
