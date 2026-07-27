# APP BUNDLES KNOWLEDGE BASE

## OVERVIEW
Bundle overlays aggregate app bases and encode version policy only.

## WHERE TO LOOK
| Task | Location | Notes |
| --- | --- | --- |
| Local/dev version policy | `dev-flex/dev-flex.yaml` | Loose wildcard/pre-release constraints. |
| Prod version policy | `prod-stable/prod-stable.yaml` | Exact semver pins; validation enforces this. |
| Bundle membership | `*/kustomization.yaml` | Imports `../../base/<app>` entries. |

## CONVENTIONS
- Bundle YAML contains partial HelmRelease documents that patch `spec.chart.spec.version`.
- Keep bundle changes limited to membership and version constraints.
- Production pins must match `^[0-9]+\.[0-9]+\.[0-9]+([-.][0-9A-Za-z]+)*$`.
- `prod-stable/prod-stable.yaml` must keep exact semver pins; `scripts/validate.sh` enforces this.
- Bundle files may include apps that a cluster later removes with delete patches.

## ANTI-PATTERNS
- Do not put Helm values, routes, PVCs, or secrets in bundles.
- Do not loosen `prod-stable` to `1.x`, `*`, or floating tags.
- Do not assume a bundled app is active in every cluster; cluster overlays can delete inherited resources.

## UNIQUE STYLES
- `dev-flex` can float aggressively for local experimentation.
- `prod-stable` is both runtime policy and CI validation target.
