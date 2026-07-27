# CLUSTERS KNOWLEDGE BASE

## OVERVIEW
Environment entrypoints and overlays for `local` and `homelab`. This is where runtime differences are expressed.

## STRUCTURE
```
clusters/
├── local/       # Kind development cluster
└── homelab/     # production Talos cluster
```

## WHERE TO LOOK
| Task | Location | Notes |
| --- | --- | --- |
| Change cluster root order | `clusters/<env>/kustomization.yaml` | Local also includes `flux-system`. |
| Change Flux app reconciliation | `clusters/<env>/bundle.yaml` | Points to `./clusters/<env>/apps`. |
| Change Flux CRD reconciliation | `clusters/<env>/crds.yaml` | Points to `./crds/<env>`. |
| Add app to an env | `clusters/<env>/apps/kustomization.yaml` | Include app overlay dir and target patches. |
| Override chart values | `clusters/<env>/apps/<app>/*-values.yaml` | Patch matching HelmRelease target. |
| Add routing | `clusters/<env>/apps/<app>/*http-route.yaml` | Gateway API HTTPRoute lives at cluster layer. |
| Wire secrets | `clusters/<env>/apps/<app>/secrets.yaml` | Local can use SOPS Secret; homelab usually uses ExternalSecret. |

## CONVENTIONS
- Cluster app kustomizations start by importing exactly one bundle: local -> `dev-flex`, homelab -> `prod-stable`.
- Cluster overlays patch inherited HelmReleases with `target.kind/name`; keep target names aligned with bundle metadata.
- Delete inherited resources with `rm-*.yaml` files and list them under `patches`.
- Storage, certificates, Gateway API routes, DB clusters, monitoring probes, and secret refs belong here, not in `apps/base`.
- Homelab production secrets prefer External Secrets from Vaultwarden; SOPS is only for bootstrap/chicken-and-egg cases.
- Local bootstrap uses Kind and `clusters/local/sops.agekey` through the Makefile.

## ANTI-PATTERNS
- Do not edit generated Flux files under `clusters/local/flux-system/gotk-*.yaml`.
- Do not assume all clusters run the same app sets; overlays remove bundle members with delete patches.
- Do not add prod-only resources to `local` just because the app exists in the bundle.
- Do not move `clusters/homelab/misc` into an app without checking `clusters/homelab/apps/kustomization.yaml`, which imports `../misc`.

## UNIQUE STYLES
- `clusters/homelab/apps/kustomization.yaml` has many removal patches for inherited bundle resources plus `../misc`.
- `clusters/local` contains generated Flux bootstrap manifests plus Kind-oriented overlays.
