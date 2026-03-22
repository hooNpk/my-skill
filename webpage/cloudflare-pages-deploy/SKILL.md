---
name: cloudflare-pages-deploy
description: |
  Deploy a static website to Cloudflare Pages using the wrangler CLI. Use this skill whenever the user asks to deploy, publish, or push changes to Cloudflare Pages, or mentions "cloudflare", "wrangler", "pages deploy", or wants to make their site live. Covers both first-time project setup and incremental update deployments.
---

# Cloudflare Pages Deploy

Deploy static websites to Cloudflare Pages using the wrangler CLI. This handles both initial setup and subsequent update deployments.

## Prerequisites

- **Cloudflare API Token**: The user needs a Cloudflare API token with Pages edit permissions. Store it securely (never commit to git).
- **Node.js**: Required to run wrangler. If not installed globally, a portable version can be used.
- **wrangler CLI**: Cloudflare's CLI tool for Pages deployment.

## Setup (First Time)

### 1. Install wrangler

If Node.js is available globally:
```bash
npm install -g wrangler
```

If Node.js is not globally available (e.g., Windows without global install), use a portable approach:
```bash
# Download portable Node.js
curl -o /tmp/node.zip https://nodejs.org/dist/v22.14.0/node-v22.14.0-win-x64.zip
unzip -o /tmp/node.zip "node-v22.14.0-win-x64/node.exe" -d /tmp/

# Install wrangler locally
cd /tmp/node-v22.14.0-win-x64
./node.exe -e "require('child_process').execSync('npm install wrangler', {stdio:'inherit'})"
```

### 2. Store the API token

Save the token in a file that's gitignored:
```bash
echo "YOUR_TOKEN_HERE" > .cloudflare-token
echo ".cloudflare-token" >> .gitignore
```

Or use an environment variable:
```bash
export CLOUDFLARE_API_TOKEN="YOUR_TOKEN_HERE"
```

### 3. First deployment

```bash
CLOUDFLARE_API_TOKEN="$(cat .cloudflare-token)" wrangler pages deploy . \
  --project-name=my-project \
  --branch=main
```

On first deploy, Cloudflare creates the project automatically. The output includes a unique deployment URL like `https://abc123.my-project.pages.dev`.

## Update Deployment

For subsequent deployments after code changes, run the same command:

```bash
CLOUDFLARE_API_TOKEN="$(cat .cloudflare-token)" wrangler pages deploy . \
  --project-name=my-project \
  --branch=main
```

Wrangler automatically detects which files changed and only uploads the diff, making updates fast (usually 1-3 seconds for small changes).

### Using portable Node.js (Windows)

If using the portable Node.js approach, the node.exe may disappear between sessions. Re-extract if needed:
```bash
# Check if node exists, re-extract if not
test -f /tmp/node-v22.14.0-win-x64/node.exe || \
  unzip -o /tmp/node.zip "node-v22.14.0-win-x64/node.exe" -d /tmp/

# Deploy using portable node
CLOUDFLARE_API_TOKEN="$(cat .cloudflare-token)" \
  /tmp/node-v22.14.0-win-x64/node.exe \
  /tmp/node-v22.14.0-win-x64/node_modules/wrangler/bin/wrangler.js \
  pages deploy . --project-name=my-project --branch=main
```

## Deployment Output

A successful deployment shows:
```
✨ Success! Uploaded 2 files (128 already uploaded) (1.01 sec)
🌎 Deploying...
✨ Deployment complete! Take a peek over at https://abc123.my-project.pages.dev
```

Always share the deployment URL with the user after a successful deploy.

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Token expired | User needs to generate a new token at Cloudflare dashboard → API Tokens |
| `node.exe` not found | Re-extract from zip: `unzip -o /tmp/node.zip ...` |
| 403 Forbidden | Token lacks Pages edit permission — regenerate with correct scope |
| Project not found | First deploy creates the project; check `--project-name` spelling |

## Token Security

- Never commit API tokens to git
- Store in `.cloudflare-token` and add to `.gitignore`
- If token is shared in chat, save to a gitignored file immediately
- Tokens can be rotated at: Cloudflare Dashboard → My Profile → API Tokens
