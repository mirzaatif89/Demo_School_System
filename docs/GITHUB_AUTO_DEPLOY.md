# GitHub Actions Auto Deploy

Push to `main` will deploy this app to the live server through SSH.

## One-time server setup

Create the live app folder on Hostinger/cPanel:

```bash
mkdir -p /home/<cpanel_user>/my_school_app
cd /home/<cpanel_user>/my_school_app
```

Create production `.env` in that folder. Use `.env.example` values as a guide and set real DB/SMTP credentials.

Required live `.env` keys:

```bash
NODE_ENV=production
PORT=3000
DB_HOST=localhost
DB_PORT=3306
DB_NAME=your_database
DB_USER=your_database_user
DB_PASSWORD=your_database_password
JWT_SECRET=change_this_to_a_long_random_secret
```

## GitHub repository secrets

In GitHub:

`Repo -> Settings -> Secrets and variables -> Actions -> New repository secret`

Add these secrets:

```text
LIVE_HOST       Hostinger/cPanel SSH host, for example kidsrootsjand.com
LIVE_PORT       SSH port, usually 22
LIVE_USER       SSH username
LIVE_SSH_KEY    Private SSH key text
LIVE_APP_PATH   Live app folder, for example /home/<cpanel_user>/my_school_app
```

## Deploy flow

Commit and push from this app repo:

```bash
git add .github/workflows/deploy-live.yml .gitignore package.json package-lock.json docs/GITHUB_AUTO_DEPLOY.md
git commit -m "Add GitHub Actions live deploy"
git push origin main
```

After push, GitHub Actions will:

1. Build a tar archive of the app.
2. Upload it to the server through SSH.
3. Preserve live `.env`, `admin_credentials.json`, permissions files, logs, and uploads.
4. Install production dependencies with `npm ci --omit=dev`.
5. Run `npm run validate`.
6. Restart with PM2 if available, otherwise touch `tmp/restart.txt` for cPanel Node.js.

## Important

Do not commit `.env`. Production secrets must stay only in GitHub Secrets or the live server `.env`.
