# Hostinger manual steps

## Important setup steps

1. Create the real `.env` from the example and fill in the active OpenClaw values.
2. Ensure `NEXT_PUBLIC_GATEWAY_URL` is set, e.g.:
   ```env
   NEXT_PUBLIC_GATEWAY_URL=ws://localhost:51299
   ```
3. Ensure the gateway token in Mission Control matches the running OpenClaw token exactly.
4. Ensure these OpenClaw paths exist on the host:
   - `/docker/openclaw-xunf/data/.openclaw`
   - `/docker/openclaw-xunf/data/.openclaw/openclaw.json`
5. Fix permissions so the Mission Control container can read the mounted OpenClaw config:
   ```bash
   chmod 755 /docker/openclaw-xunf/data/.openclaw
   chmod 644 /docker/openclaw-xunf/data/.openclaw/openclaw.json
   ```
6. Restart with compose:
   ```bash
   docker compose -f docker-compose.yml -f docker-compose.override.yml --env-file .env down
   docker compose -f docker-compose.yml -f docker-compose.override.yml --env-file .env up -d
   ```

## Important compose change

In `docker-compose.yml`, remove the `mission-control` `ports:` entry.

Reason:
- the base file had:
  ```yaml
  - "${MC_PORT:-3000}:${PORT:-3000}"
  ```
- adding another `ports:` entry in the override caused duplicate host-port binding

Keep the Hostinger-specific port binding only in `docker-compose.override.yml`:

```yaml
services:
  mission-control:
    ports:
      - "127.0.0.1:${MC_PORT:-3000}:${PORT:-3000}"
    networks:
      - default
      - openclaw-external
    volumes:
      - /docker/${OPENCLAW_INSTANCE_NAME}/data:/run/openclaw-data:ro

networks:
  openclaw-external:
    name: ${OPENCLAW_NETWORK}
    external: true
```

## Useful checks

```bash
docker exec mission-control env | grep OPENCLAW
docker exec mission-control env | grep NEXT_PUBLIC_GATEWAY_URL
docker exec mission-control sh -c 'head -n 5 /run/openclaw-data/.openclaw/openclaw.json'
docker compose -f docker-compose.yml -f docker-compose.override.yml --env-file .env logs -f mission-control
```
