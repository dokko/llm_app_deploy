# llm_app_deploy (public) — Deploy pipeline for `llm_app` (private)

This repo holds deployment automation only. The actual service code lives in the private repo `llm_app`. On push to `main`, GitHub Actions clones `llm_app` (read-only), installs deps, and deploys to Cloudflare Workers via Wrangler.

## Repo layout
```
.github/workflows/deploy.yml   # CI/CD pipeline (main push only)
.gitignore                     # ignore wrangler state, node_modules, etc.
README.md
```

## Required GitHub secrets
- `LLM_APP_DEPLOY_KEY`: SSH private key with **read-only** access to `llm_app` (deploy key or PAT via `ssh-keygen` + `ssh-agent`).
- `LLM_APP_REPO_SSH`: SSH URL of the private repo, e.g. `git@github.com:YOUR_ORG/llm_app.git`.
- `CLOUDFLARE_API_TOKEN`: Token with Workers deploy perms (Workers Scripts + KV/R2 if used).
- `CLOUDFLARE_ACCOUNT_ID`: Cloudflare account ID.
- `SUMMARY_MODEL` (optional): Override model for deploys (e.g. `@cf/meta/llama-3-8b-instruct`).

## How the workflow works
1. Trigger: push to `main` only (no PR/fork secrets exposure).
2. Checkout this public repo.
3. Load the read-only deploy key into `ssh-agent`.
4. Clone the private repo `llm_app` with that key.
5. Install dependencies (`npm ci`) inside the cloned repo.
6. Deploy with Wrangler: `npx wrangler deploy src/index.ts --config wrangler.toml`.

## Security notes
- Secrets are only available on `main` push runs; forked PRs cannot access them.
- Private repo access uses a read-only deploy key or PAT; do not store keys in the repo.
- No service code is stored here—only automation.

## Quick setup
1. Create a read-only deploy key for `llm_app`; add the private key as `LLM_APP_DEPLOY_KEY`, the SSH URL as `LLM_APP_REPO_SSH`.
2. Add `CLOUDFLARE_API_TOKEN` and `CLOUDFLARE_ACCOUNT_ID` secrets.
3. Push to `main` in this repo; the Worker will deploy automatically.
