# Deployment Guide: Stellar Syntec Chatwoot

## 1. Prepare Environment

Ensure you have the `docker-compose.yaml` and `.env` files in your deployment folder.

**Crucial:** Edit the `.env` file!
- Set `POSTGRES_PASSWORD` to a strong password.
- Set `REDIS_PASSWORD` to a strong password.
- Update `REDIS_URL` to match: `redis://:YOUR_REDIS_PASSWORD@redis:6379`.
- Set `SECRET_KEY_BASE` (generate one using `openssl rand -hex 64`).
- **Most Important:** Set `FRONTEND_URL` to the exact URL you use to access the dashboard (e.g., `http://localhost:3000` or `https://chat.stellarsyntec.com`).

## 2. Deploy the Stack

Run the stack in detached mode:

```bash
docker-compose up -d
```

## 3. Initialize Database (The Missing Step)

The 404 error often happens because the database is empty. You must run the setup script **once** after the first deployment.

Execute this command inside the running `web` container:

```bash
docker-compose exec web bundle exec rails db:chatwoot_prepare
```

*If using Portainer:*
1. Go to the **Containers** list.
2. Find the container named `stellar-chatwoot_web_1` (or similar).
3. Click **Console** ( >_ icon).
4. Select `/bin/bash` or `/bin/sh` and click Connect.
5. Run: `bundle exec rails db:chatwoot_prepare`

Wait for it to finish creating tables and seeding default data.

## 4. Verify & Access

1. Check logs: `docker-compose logs -f web`
2. Open your browser to the `FRONTEND_URL` you configured.
3. Login with default credentials (if seeded) or create a new account.

## Troubleshooting

- **Error 404:** Usually means `FRONTEND_URL` mismatch or Database not migrated.
- **Asset compilation errors:** Ensure `NODE_OPTIONS="--max-old-space-size=4096"` is in your environment variables.
