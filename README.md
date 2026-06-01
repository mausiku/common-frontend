# Common Service Portal — Frontend (Azure Static Web Apps)

> **GitHub deployment:** this repo deploys via GitHub Actions (`.github/workflows/deploy-frontend.yml`). Push to `main` (or run the workflow manually) and the prebuilt files at the repo root are uploaded to your Static Web App — no build runs. Set the repo secret `AZURE_STATIC_WEB_APPS_API_TOKEN` (Static Web App → Manage deployment token) before the first push.

Prebuilt single-page app. **No build step** — the bundle is shipped ready to
serve, as required.

## Contents

| File | Purpose |
|------|---------|
| `index.html` | App entry point; loads `/assets/main-4GRIUDY5.js` |
| `assets/main-4GRIUDY5.js` | Prebuilt application bundle (React) |
| `staticwebapp_config.json` | Routing, security headers, SPA fallback |

## How it talks to the backend

The app calls the API using **relative `/api/*` paths**. You must make those
paths reach the backend App Service. Two supported approaches:

### Option A — Linked backend (recommended)

In the Azure Portal, open your Static Web App → **APIs** → **Link** and link
your backend **App Service**. Azure then routes `/api/*` to it automatically.
No code changes needed.

### Option B — Proxy via configuration

Add a rewrite to `staticwebapp_config.json` pointing `/api/*` at your backend
URL (replace with your App Service hostname):

```json
{
  "routes": [
    { "route": "/api/*", "rewrite": "https://<your-app-service>.azurewebsites.net/api/*" }
  ]
}
```

If you use a separate backend domain, set `CORS_ORIGIN` on the backend to this
Static Web App's URL.

## Deploy

### Option A — SWA CLI

```bash
# from inside this frontend folder
npx @azure/static-web-apps-cli deploy . \
  --deployment-token <your-swa-deployment-token> \
  --app-location . \
  --output-location .
```

### Option B — Azure CLI (zip)

```bash
zip -r ../frontend.zip .
az staticwebapp deploy \
  --name <your-swa> \
  --resource-group <your-rg> \
  --source ../frontend.zip
```

### Option C — GitHub Actions

When you create the Static Web App from a repo, set:

- **App location:** `/` (this folder's contents at repo root)
- **Api location:** *(leave empty — backend deploys separately)*
- **Output location:** *(leave empty — already prebuilt)*

Crucially, because the app is prebuilt, leave the build command empty so Azure
**does not attempt to build from source**.

## Default logins (seeded by backend)

- `admin@roomsync.na` / `admin123` (Admin)
- `editor@roomsync.na` / `editor123` (Editor)

Change these after first sign-in (Settings → Users). The sample logo can be
replaced in Settings.

## Notes

- The config sets a strict `Cache-Control: no-store` and security headers
  (`X-Frame-Options`, `X-Content-Type-Options`, etc.).
- SPA deep links fall back to `index.html`; `/api/*` and `/assets/*` are
  excluded from the fallback.
