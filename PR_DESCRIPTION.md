## Summary
Initial setup of the expectedbehaviors fork: chart at repo root with simplified schema, defaultVault/vault parameterization, and release workflows on `main`.

## Changes
- **Chart at repo root:** onepassworditem 1.0.0 (was in `OnePasswordItem/`).
- **Values:** `enabled` + `items`; namespace always `.Release.Namespace`. Optional `defaultVault` and per-item `vault`; `item` can be full path or short form.
- **Annotations/labels:** Per-item `annotations`/`labels` applied to Secrets via post-install hook + RBAC (e.g. for kubernetes-replicator).
- **Workflows:** Release on merge to main, helm-publish (shared action), release-notes. Default branch is `main`.
- **README:** Fork attribution (vquie/OnePasswordItem-helm), docs, Support this project.
- **Removed:** `OnePasswordItem/` subdir, `renovate.json` (can re-add later if desired).

## After merge
- Enable **Settings → Pages → branch: gh-pages** once so the Helm repo is served.
- Helm repo URL: `https://expectedbehaviors.github.io/OnePasswordItem-helm`
