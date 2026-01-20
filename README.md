# Matrix Docker Configuration

Personal Matrix stack running:

- Synapse (homeserver)
- Postgres (Synapse DB)
- Element Web (client UI)
- Synapse Admin (admin UI)
- Nginx (TLS + reverse proxy)

## Prerequisites

- Docker + Docker Compose v2 (`docker compose`)
- Two DNS records pointing to this host:
    - `${MATRIX_DOMAIN}` (your homeserver domain)
    - `${ELEMENT_WEB_DOMAIN}` (your Element Web domain)
    - `${SYNAPSE_ADMIN_DOMAIN}` (your admin UI domain)
- A TLS certificate that covers both domains (SAN cert), or a wildcard cert.
  You can use Let's Encrypt or any other CA.
- An Htpasswd file for basic auth on the admin UI.

## Files you must edit

1) Create `.env` from the example:

```bash
cp .env.example .env
```

Set at least:

- `MATRIX_DOMAIN`
- `SYNAPSE_ADMIN_DOMAIN`
- `POSTGRES_PASSWORD`
- `TLS_CERT_FILE`, `TLS_KEY_FILE`, `HTPASSWD_FILE` (paths on the host)

By default the example expects:

- `nginx/.keys/fullchain.pem`
- `nginx/.keys/privkey.pem`
- `nginx/.keys/htpasswd`

Alternatively to static certs, you can use Let's Encrypt with Certbot or similar tools to generate certs.

2) Update Nginx domains in [nginx/nginx.conf](nginx/nginx.conf):

- Replace `example.com` with your `MATRIX_DOMAIN`
- Replace `element.example.com` with your `ELEMENT_WEB_DOMAIN`
- Replace `matrix-admin.example.com` with your `SYNAPSE_ADMIN_DOMAIN`

3) Update Element config in [element/config.json](element/config.json):

- Replace `example.com` with your `MATRIX_DOMAIN`

## Htpasswd file

Generate an Htpasswd file for basic auth on the Synapse Admin UI.

```bash
sudo apt update && sudo apt install apache2-utils -y
htpasswd -c nginx/.keys/htpasswd adminuser
chmod 600 nginx/.keys/htpasswd
```

## TLS certificates

Place your cert/key where `TLS_CERT_FILE` / `TLS_KEY_FILE` point.

```bash
cp /path/to/fullchain.pem nginx/.keys/fullchain.pem
cp /path/to/privkey.pem nginx/.keys/privkey.pem
chmod 600 nginx/.keys/privkey.pem
```

## First start

Pre-generate the Synapse config:

```bash
docker compose run --rm synapse generate
```

This creates `synapse/data/homeserver.yaml`. Since we are using Postgres, edit the database section like so:

```yaml
database:
  name: psycopg2
  args:
    user: synapse
    password: <pass> # use the value from your .env POSTGRES_PASSWORD
    dbname: synapse
    host: postgres
    cp_min: 5
    cp_max: 10
```

Next, edit any other Synapse settings you want in `synapse/data/homeserver.yaml`.

Read the Synapse docs for more info: https://element-hq.github.io/synapse/latest/welcome_and_overview.html

Bring the stack up:

```bash
docker compose up -d
```

## Create your first admin user

Navigate to project root and run the registration helper:

```bash
docker compose exec synapse register_new_matrix_user -c /data/homeserver.yaml -a http://localhost:8008
```

Follow the prompts to create your first admin user.

## Accessing the services

Login with your newly created user at the URLs you set up in your env and nginx config:
Should be something like:
- Element Web: `https://element.example.com/`
- Synapse Admin UI: `https://matrix-admin.example.com/`

You may download any Matrix client (Element, FluffyChat, etc.) and connect to your homeserver from any device at `https://$MATRIX_DOMAIN/`.
List of clients: https://matrix.org/ecosystem/clients/

Your Matrix ID (MXID) will be in the format: `@username:$MATRIX_DOMAIN`

## URLs

- Element: `https://$ELEMENT_WEB_DOMAIN/`
- Synapse Client-Server API: `https://$MATRIX_DOMAIN/_matrix/...`
- Federation: `https://$MATRIX_DOMAIN:8448/`
- Synapse Admin UI: `https://$SYNAPSE_ADMIN_DOMAIN/`

## Stop / logs

```bash
docker compose logs -f --tail=200
docker compose down
```

# Contact

Reach out on Matrix: [@ayman:alsuleihi.com](https://matrix.to/#/@ayman:alsuleihi.com)

# Contributing

Feel free to open issues or submit pull requests.

# License

MIT License. See [LICENSE](LICENSE) file for details.