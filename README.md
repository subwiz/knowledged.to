# knowledged.to

Hugo site for `https://knowledged.to/`. Content under `content/` is synced from `subwiz/knowledged-publisher`; do not edit by hand (except `content/_index.md`).

Built and deployed automatically by GitHub Actions on every push to `main`.

## Operational setup (one-time)

| Step | Where | What |
|---|---|---|
| A records | Domain registrar | `@` → `185.199.108.153`, `.109.153`, `.110.153`, `.111.153` |
| TXT verification | Domain registrar + GitHub | `_github-pages-challenge-subwiz.knowledged.to` |
| Pages source | GitHub Settings → Pages | Source = GitHub Actions |
| Custom domain | GitHub Settings → Pages | `knowledged.to` |
| HTTPS | GitHub Settings → Pages | Enforce HTTPS (after cert provisions) |
| Submodule init | New clone | `git submodule update --init --recursive` |
