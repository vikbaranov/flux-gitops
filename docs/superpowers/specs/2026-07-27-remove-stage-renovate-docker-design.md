# Remove Stage Cluster and Switch to Renovate Docker Image Management

## Overview
Remove the entire `stage` (d-talos) cluster from the repository and switch application update management entirely to Renovate tracking Docker container image tags instead of Helm chart versions.

## Context

Currently, the repository has a three-environment setup:
- **local** — Kind development cluster using `dev-flex` bundle
- **stage** — Talos staging cluster using `stage-flex` bundle, driving promotion workflow
- **homelab** — Production Talos cluster using `prod-stable` bundle

Updates flow through a promotion pipeline: stage cluster upgrades → Flux alert → GitHub dispatch → promotion workflow patches `prod-stable` chart versions → PR → merge.

The user wants to eliminate the stage cluster and have Renovate open PRs directly for Docker image updates across all apps.

## Goals

1. Remove all stage cluster manifests, CRDs, and configuration
2. Delete the `stage-flex` bundle and the promotion GitHub workflow
3. Configure Renovate to track Docker images for **all** apps via `# renovate: datasource=docker` annotations
4. Disable Renovate Helm chart version tracking
5. Update all documentation and configuration files that reference stage

## Non-Goals

- Converting any apps away from HelmRelease (they stay Helm-based)
- Changing the production cluster infrastructure
- Modifying the local cluster setup beyond `.envrc` SOPS key path
- Removing apps from the production bundle

## Architecture

After removal, the repository has two environments:
- **local** — Kind cluster for development/testing
- **homelab** — Production Talos cluster

Application updates are managed by:
1. Renovate detects image tag changes in cluster values files
2. Renovate opens PRs with image tag bumps
3. CI validates manifests via `make validate`
4. User reviews and merges
5. Flux reconciles the production cluster

## File Changes

### Delete
- `clusters/stage/` — entire directory (apps, bundle, crds, namespaces, SOPS key, .DS_Store)
- `crds/stage/` — stage CRD selection
- `apps/bundles/stage-flex/` — stage version bundle
- `.github/workflows/promotion.yaml` — promotion workflow

### Modify
- `.sops.yaml` — remove stage encryption rules
- `.envrc` — update `SOPS_AGE_KEY_FILE` to `clusters/local/sops.agekey`
- `renovate.json`:
  - Remove `apps/bundles/**` from flux manager (disable chart version tracking)
  - Add packageRule to disable helm depType in flux manager
  - Keep docker depType tracking for image tags
- `README.md` — remove stage from structure, bundles, and promotion flow sections
- `AGENTS.md` (root, apps/, apps/bundles/, clusters/, clusters/homelab/apps/, crds/) — remove stage references
- `clusters/homelab/apps/*/values.yaml` — add `# renovate: datasource=docker` image tag annotations for all apps that don't already have them

## Renovate Configuration

### Before
```json
{
  "flux": {
    "managerFilePatterns": [
      "/apps/base/.+\\.ya?ml$/",
      "/apps/bundles/.+\\.ya?ml$/",
      "/clusters/.+\\.ya?ml$/"
    ]
  }
}
```

### After
```json
{
  "flux": {
    "managerFilePatterns": [
      "/apps/base/.+\\.ya?ml$/",
      "/clusters/.+\\.ya?ml$/"
    ]
  },
  "packageRules": [
    {
      "matchManagers": ["flux"],
      "matchDepTypes": ["helm"],
      "enabled": false
    }
  ]
}
```

This keeps the flux manager active for Docker image tags inside HelmRelease `values` blocks but disables chart version bump PRs.

## Image Tag Annotations

For apps that already have image overrides with renovate annotations (immich, n8n), no changes needed.

For apps that need new image tag annotations added to their cluster values files, the pattern is:

```yaml
spec:
  values:
    image:
      repository: <registry>/<image>
      # renovate: datasource=docker depName=<registry>/<image> versioning=semver
      tag: <current-tag>
```

Apps requiring new annotations:
- jellyfin (currently no explicit image/tag)
- podinfo (currently no explicit image/tag)
- goldpinger (currently no explicit image/tag)
- mattermost (has custom repo, needs tag pin)
- open-webui (has chart version override, needs image tag)
- vaultwarden (has no explicit image/tag)
- Additional infrastructure apps if their values expose image blocks

## Validation

`make validate` and `./scripts/validate.sh` will continue to work because they iterate over `clusters/*/`. Removing `clusters/stage/` simply removes one iteration target. The prod-stable pin check in `validate.sh` can remain (it still validates that production chart versions are exact semver) even though Renovate won't be opening chart version PRs anymore.

## Risks

- Some Helm charts may not use standard `image.repository`/`image.tag` values structure, requiring custom regex managers
- Infrastructure charts (cilium, cert-manager, victoria-metrics) images may not be exposed in their public values schemas in a way Renovate's flux manager can detect
- Production cluster will apply image tag bumps directly without staging validation

## Rollout

1. Write and commit design spec
2. Create implementation plan
3. Execute plan task-by-task
4. Run `make validate` after each batch of changes
5. Commit each logical unit
6. Final review and merge
