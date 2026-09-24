# Hostinger cPanel deployment

The production subdomain is expected to use one document root, for example:

```text
https://erp.dw-digitalplatforms.in/
```

Laravel is deployed to the configured backend path and Angular production
files are deployed into Laravel's `public` directory. This lets Apache route
`/api/*` and `/sanctum/*` to Laravel while all other client-side paths fall
back to Angular's `index.html`.

## GitHub configuration

Repository Variables (`Settings → Secrets and variables → Actions → Variables`):

```text
CPANEL_BACKEND_PATH=/home/CPANEL_USER/public_html/erp
CPANEL_FRONTEND_PATH=/home/CPANEL_USER/public_html/erp/public
BACKEND_URL=
```

`BACKEND_URL` should normally be empty because the frontend and Laravel API
share the same origin. If the API is intentionally hosted elsewhere, set it
to the HTTPS API origin/path, for example `https://api.example.com`.

Repository Secrets:

```text
CPANEL_SSH_HOST
CPANEL_SSH_USERNAME
CPANEL_SSH_KEY
CPANEL_SSH_KNOWN_HOSTS
```

Enable SSH access in Hostinger/cPanel and use the SSH host, SSH username, and
private key for that account. The workflows use the native `ssh` and `scp`
clients to upload ZIP archives and run `unzip` on the server.

Optional repository variable:

```text
CPANEL_SSH_PORT=22
```

Use the SSH port shown by Hostinger; shared hosting commonly provides a
non-default port.

Create `CPANEL_SSH_KNOWN_HOSTS` locally after verifying the host fingerprint
with Hostinger:

```bash
ssh-keyscan -p <SSH_PORT> <SSH_HOST>
```

Store the complete output as the GitHub secret. Do not disable host-key
verification in the workflows.

## Server-only Laravel environment

Create `backend/.env` on the server manually. Never store it in GitHub or the
repository. At minimum configure:

```text
APP_ENV=production
APP_DEBUG=false
APP_URL=https://erp.dw-digitalplatforms.in
APP_KEY=<generated Laravel application key>

DB_CONNECTION=mysql
DB_HOST=<Hostinger MySQL host>
DB_PORT=3306
DB_DATABASE=<cpanel database name>
DB_USERNAME=<cpanel database user>
DB_PASSWORD=<database password>

GEMINI_API_KEY=<Gemini API key>
GEMINI_MODEL=gemini-3.1-flash-lite
GEMINI_BASE_URL=https://generativelanguage.googleapis.com/v1beta

CORS_ALLOWED_ORIGINS=https://erp.dw-digitalplatforms.in
SANCTUM_STATEFUL_DOMAINS=erp.dw-digitalplatforms.in
SESSION_SECURE_COOKIE=true
SESSION_SAME_SITE=lax
```

Generate `APP_KEY` once with `php artisan key:generate --show` in a secure
local/server shell, then place the value in the server `.env`. Generate the
Gemini key in Google AI Studio and restrict it to the required API usage.

The backend workflow packages Laravel into a ZIP, excludes `vendor/` and
`.env`, uploads the archive over SSH, and extracts it on the server. It copies
the extracted files over the existing application, so the already-uploaded
compatible `vendor/` directory and server `.env` remain in place.

## First deployment checklist

1. Create the subdomain and confirm its document root.
2. Create the MySQL database/user in cPanel.
3. Create the FTP account and add the three GitHub secrets.
4. Upload Laravel `vendor/` to `CPANEL_BACKEND_PATH/vendor`.
5. Create the server `.env` with production values.
6. Run the backend workflow once, then verify `/api/v1/auth/me` responds with
   the normal unauthenticated API response.
7. Run the frontend workflow and open the subdomain.
8. Confirm login, API requests, Angular deep links, PDF downloads, storage,
   and Zaakiy SSE behavior.

The workflows deploy only from `main` or manual dispatch. They do not enable
an implicit all-branch ERP scope and they do not push Git changes.
