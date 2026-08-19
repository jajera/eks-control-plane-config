# Amazon EKS control plane parameters

Local Zensical site using the Patina theme.

## GitHub Actions

Same layout as [jajera/privatelink-conduit `.github`](https://github.com/jajera/privatelink-conduit/tree/main/.github): Dependabot, commit-message checks, markdown lint, and a docs workflow.

This site is Zensical, not Jekyll, so `docs.yml` calls [`zensical-pages-deploy`](https://github.com/actionsforge/actions/blob/main/.github/workflows/zensical-pages-deploy.yml) (`pip install -r requirements.txt`, `zensical build --clean`, output `site/`). Pull requests only build. Pushes to `main` deploy GitHub Pages.

Set **Settings → Pages → Build and deployment → Source → GitHub Actions**. After a successful Docs run on `main`, the site is at [jajera.github.io/eks-control-plane-config](https://jajera.github.io/eks-control-plane-config/).

## Build locally

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
.venv/bin/zensical serve
```

Open the URL printed by Zensical (usually `http://127.0.0.1:8000`).

Static output (same command CI uses):

```bash
.venv/bin/zensical build --clean
```

That writes HTML into `site/` (gitignored).
