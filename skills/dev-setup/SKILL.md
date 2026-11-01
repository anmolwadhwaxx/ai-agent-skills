---
name: dev-setup
description: Start the application locally the way this project expects. Use whenever the user asks to run, start, boot, set up, or "get this working locally", install dependencies, fix a startup error, or check that the dev server is healthy. Covers install commands, environment variables, localhost ports, and a startup health check for a Next.js app. Trigger even if the user only says "run it" or "why won't it start".
---

# Dev Setup

Start the app locally with the correct commands, environment, and port, then confirm it actually works.

## Project configuration (edit this block for your project)

```
Package manager:      npm            # npm | pnpm | yarn
Node version:         20.x           # check .nvmrc / package.json "engines"
Install command:      npm ci         # use `npm install` only if no lockfile
Dev command:          npm run dev
Build command:        npm run build
Dev port:             3000
Health URL:           http://localhost:3000/
API health URL:       http://localhost:3000/api/health   # if it exists
Env template:         .env.example
Local env file:       .env.local     # never committed
```

If the repository's README or `package.json` scripts disagree with this block, trust the repository and tell the user about the mismatch.

## Rules

1. **Never invent environment values.** If a variable is missing, list it and ask the user. Do not make up keys or URLs.
2. **Never commit or print secrets.** `.env.local` stays out of git. When showing env status, show variable names only, not values.
3. **Never run destructive commands** (`rm -rf`, database resets, `git clean -fdx`, dropping tables) without explicit approval.
4. **Local only.** This skill starts a local dev server. It does not deploy anything.
5. **Do not kill unknown processes.** If the port is taken, report what is using it and ask before stopping it.

## Procedure

### Step 1: Pre-flight checks
- Confirm Node and package manager versions match the configuration above (`node -v`, `npm -v`).
- Confirm a lockfile exists and matches the package manager.
- Confirm you are in the project root (a `package.json` is present).

### Step 2: Install dependencies
- Run the install command.
- If it fails, read the error. Common causes: wrong Node version, private registry auth missing, stale `node_modules`. Report the cause and the fix, and ask before deleting `node_modules`.

### Step 3: Environment variables
1. Compare `.env.example` with `.env.local` (names only).
2. List variables that are **required but missing** and variables that are **present but empty**.
3. Remind the user that in Next.js:
   - Variables needed in the browser must start with `NEXT_PUBLIC_`.
   - Anything with that prefix is public, so never put a secret in one.
   - Server-only values stay without the prefix.
   - Env changes need a dev-server restart.
4. If anything required is missing, stop and ask the user for it. Do not proceed with guesses.

### Step 4: Check the port
- Check whether the dev port is free. If occupied, report the process and offer: (a) stop it, or (b) use another port (`npm run dev -- -p 3001`).

### Step 5: Start the dev server
- Run the dev command in the background or a separate terminal so it keeps running.
- Wait for the "ready" line in the output.

### Step 6: Startup health check
- Request the Health URL (`curl -i <url>`). Expect HTTP 200.
- If an API health URL exists, check it too.
- Scan the server output for errors or warnings about missing env vars.

### Step 7: Report
```
## Dev Setup Report
- Node / package manager: <versions>
- Install: OK / failed (<reason>)
- Env vars: <N> required, <N> missing: <names>
- Port: <port> free / used by <process>
- Server: running at <url>
- Health check: <status code> / failed
- Next steps: <if anything needs the user>
```

## Troubleshooting quick reference
| Symptom | Likely cause | Action |
|---------|--------------|--------|
| `EADDRINUSE` | Port taken | Identify the process, ask before stopping, or use another port |
| `Module not found` | Deps not installed or wrong branch | Re-run the install command |
| Blank page + env errors | Missing `NEXT_PUBLIC_*` variable | Add to `.env.local`, restart server |
| Works on one machine only | Node version mismatch | Align with `.nvmrc` / `engines` |
| API calls fail with 401/403 | Missing or expired local credentials | Ask the user; do not fabricate |
