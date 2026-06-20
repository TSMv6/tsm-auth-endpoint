# tsm-auth-endpoint

Tiny **static** Vercel project that serves the TSM authentication manifest at:

```
https://auth.tsm-nextgen.com/tsm_manifest.json
```

Every TSM model executable fetches this file at startup to confirm the user has
the latest approved model/data (and, with tokens on, a valid access token). See
the design in the `tsm_dev_guidelines` repo.

## This is the CANONICAL live manifest

`public/tsm_manifest.json` here is the **source of truth** that exes actually
read. The copy in `tsm_dev_guidelines/tsm_manifest.json` is a reference/template —
**edit the manifest HERE** (or copy it here) and push to deploy. Don't let the two
drift; if you edit it in `tsm_dev_guidelines`, copy it over before deploying.

## Deploy

No build step — it's static files in `public/`.

1. First time: `vercel link` (create a **new, separate** project, not the chatbot).
2. In the Vercel dashboard, add the custom domain **auth.tsm-nextgen.com** and
   point DNS as Vercel instructs.
3. Deploy: `vercel --prod` (or connect this repo for auto-deploy on push).

`vercel.json` sets:
- `Cache-Control: max-age=300, s-maxage=300, stale-while-revalidate=86400` — edits
  go live within ~5 minutes; the CDN keeps serving during brief origin blips.
- `Access-Control-Allow-Origin: *` — harmless here; lets a browser/help page read it too.
- `/` redirects to `/tsm_manifest.json` for convenience.

## Editing the manifest (the routine you'll actually use)

- **Annual roll:** bump `tsm_version`, set new `expires`, bump `min_version`, deploy.
- **Revoke one user:** add their `jti` to `revoked_token_ids`, deploy.
- **Retire a build:** add its version to `revoked_versions`, deploy.

Each is a one-line edit + `vercel --prod`. Takes effect for every user within the
cache window.

## Keep this fresh — it's the freshness oracle

This endpoint being reachable is what lets you extend/revoke after release. The
exes tolerate brief outages (7-day offline grace), but don't let the domain lapse.
