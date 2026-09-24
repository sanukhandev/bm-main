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
CPANEL_FTP_SERVER
CPANEL_FTP_USERNAME
CPANEL_FTP_PASSWORD
```

Get these from cPanel → **Files → FTP Accounts**. Use the FTP hostname shown
by Hostinger, the FTP account username, and its password. FTPS is used by the
workflows.

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

The backend workflow deliberately excludes `vendor/` and `.env`; upload the
already-built compatible `vendor/` directory to the Laravel root and keep it
on the server. The workflow does not delete the server environment or vendor.

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
