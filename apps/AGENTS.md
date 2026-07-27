# APPS KNOWLEDGE BASE

## OVERVIEW
Reusable app definitions and version bundles. Keep app defaults here; cluster-specific behavior belongs under `clusters/<env>/apps`.

## STRUCTURE
```
apps/
├── base/        # shared app manifests
└── bundles/     # dev/prod version aggregations
```

## WHERE TO LOOK
| Task | Location | Notes |
| --- | --- | --- |
| Add a new app baseline | `apps/base/<app>/` | Include `kustomization.yaml`; usually HelmRelease + HelmRepository + namespace. |
| Set release defaults | `apps/base/<app>/helm-release.yaml` | Shared chart source and default values only. |
| Add chart source | `apps/base/<app>/helm-repo.yaml` or `oci-repo.yaml` | Match existing source type. |
| Raw workload manifests | `apps/base/it-tools`, `apps/base/qbittorrent` | These are Deployment/Service based, not only HelmRelease. |
| Change bundle versions | `apps/bundles/{dev-flex,prod-stable}/` | Edit only the bundle file for version constraints. |

## CONVENTIONS
- `apps/base/<app>` is environment-neutral: no per-cluster storage, DNS, public/private route, or secret wiring.
- Kustomizations commonly set `namespace` at the directory level instead of repeating it in every resource.
- Bundle files are multi-document YAML containing partial HelmRelease objects that patch chart versions.
- `prod-stable` versions must be exact semver pins; `scripts/validate.sh` enforces this.
- `renovate.json` scans `apps/base/**`, `apps/bundles/**`, and `clusters/**`; image tags in values need `# renovate: datasource=... depName=...` comments.

## ANTI-PATTERNS
- Do not place env-specific values or secrets in `apps/base`.
- Do not update `prod-stable` to ranges like `1.x`, `*`, or pre-release wildcards.
- Do not duplicate a whole base app into a cluster just to change values; patch it from `clusters/<env>/apps`.

## UNIQUE STYLES
- Bundle policy encodes rollout risk: `dev-flex` loose, `prod-stable` exact.
- Some base app names differ from HelmRelease names; check bundle metadata before targeting patches.
- CRD-producing operators may have related resources under `crds/base`, not only under `apps/base`.
