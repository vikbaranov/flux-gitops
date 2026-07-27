# HOMELAB APP OVERLAYS KNOWLEDGE BASE

## OVERVIEW
Production app overlays for the homelab Talos cluster. This layer turns `prod-stable` versions into real runtime configuration.

## WHERE TO LOOK
| Task | Location | Notes |
| --- | --- | --- |
| Add/remove prod app | `kustomization.yaml` | Imports `prod-stable`, app dirs, `../misc`, and patches. |
| Override Helm values | `<app>/*-values.yaml` | Target the inherited HelmRelease by metadata name. |
| Remove inherited app/resource | `<app>/rm-*.yaml` | `$patch: delete`; list under top-level `patches`. |
| Add prod route | `<app>/*http-route.yaml` | Gateway API routes live here. |
| Add prod secret wiring | `<app>/secrets.yaml` | Prefer ExternalSecret from Vaultwarden. |
| Shared prod misc | `../misc/` and `misc/` | Both are imported into production apps. |

## CONVENTIONS
- Production starts from `../../../apps/bundles/prod-stable`.
- Storage, database clusters, monitoring probes, dashboards, routes, and ExternalSecrets are prod-owned here.
- Delete patches intentionally remove unused inherited bundle members such as Mattermost/Open WebUI/CSI-NFS variants.
- Dashboard ConfigMaps often use generators with stable names and Flux substitution disabled.
- `bitwarden-cli` and `external-secrets` are part of the production secret bootstrap chain.
- Use the bundle HelmRelease metadata name when adding targeted value patches.

## ANTI-PATTERNS
- Do not assume prod runs every app listed in `prod-stable`.
- Do not move `../misc` resources without updating the app-level import.
- Do not replace ExternalSecrets with committed plaintext Kubernetes Secrets.
- Do not copy local SOPS Secret manifests into prod unless they are bootstrap-only.

## UNIQUE STYLES
- Production imports both app-local `misc/` and sibling `../misc` resources.
- Several prod directories exist only to remove inherited resources from the stable bundle.
