# Paseo Hub

Self-hosted [Paseo Hub](https://paseo.sh/docs/hub/self-hosting) at `paseo.k8s.s6n.jp`.

The chart-less upstream ships only a Docker Compose stack, so the Deployment, the PostgreSQL
StatefulSet, the Service, and the Ingress are written out here directly.

## Secrets

Both secrets are created out of band and are not committed.

`paseo-hub-postgres` holds the database password, shared by the StatefulSet and by the
`DATABASE_URL` the Hub builds:

```sh
kubectl create secret generic paseo-hub-postgres \
  --namespace paseo-hub \
  --from-literal=password="$(openssl rand -hex 32)"
```

`paseo-hub` is mounted with `envFrom`, so every key becomes an environment variable. Only
`PASEO_HUB_AUTH_SECRET` is required; drop the bootstrap keys after the first sign-in, and add
only the provider credentials you actually connect.

```sh
kubectl create secret generic paseo-hub \
  --namespace paseo-hub \
  --from-literal=PASEO_HUB_AUTH_SECRET="$(openssl rand -hex 32)" \
  --from-literal=PASEO_BOOTSTRAP_ORGANIZATION="s6n" \
  --from-literal=PASEO_BOOTSTRAP_OWNER_EMAIL="me@s6n.jp" \
  --from-literal=PASEO_BOOTSTRAP_OWNER_PASSWORD="<at least 12 characters>"
```

Rotating `PASEO_HUB_AUTH_SECRET` signs everyone out, so it has to persist across restarts.

Optional provider keys, each set added only when that integration is connected:

- GitHub: `GITHUB_APP_SLUG`, `GITHUB_APP_ID`, `GITHUB_APP_CLIENT_ID`, `GITHUB_APP_CLIENT_SECRET`,
  `GITHUB_APP_PRIVATE_KEY`, `GITHUB_WEBHOOK_SECRET`
- Slack: `SLACK_APP_ID`, `SLACK_CLIENT_ID`, `SLACK_CLIENT_SECRET`, `SLACK_SIGNING_SECRET`
- Discord: `DISCORD_CLIENT_ID`, `DISCORD_CLIENT_SECRET`, `DISCORD_BOT_TOKEN`

`STRIPE_SECRET_KEY` and `STRIPE_WEBHOOK_SECRET` are left unset, which keeps the billing surface
off entirely. Setting one without the other fails startup.

## After the first sync

Sign in at <https://paseo.k8s.s6n.jp> with the bootstrap account, replace the temporary password,
then connect a daemon:

```sh
paseo hub connect https://paseo.k8s.s6n.jp
```
