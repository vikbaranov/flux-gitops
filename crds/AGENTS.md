# CRDS KNOWLEDGE BASE

## OVERVIEW
CRDs and CRD-installing operators reconciled before app bundles. Keep API availability concerns here.

## STRUCTURE
```
crds/
├── base/        # reusable CRD sources and install manifests
├── local/       # local CRD selection
└── homelab/     # production CRD selection
```

## WHERE TO LOOK
| Task | Location | Notes |
| --- | --- | --- |
| Add shared CRD source | `crds/base/<operator>/` | Usually HelmRepository/HelmRelease, GitRepository, or raw CRDs. |
| Enable CRDs for env | `crds/<env>/kustomization.yaml` | Import the relevant `../base/<operator>` dirs. |
| Change Flux reconciliation | `clusters/<env>/crds.yaml` | Flux points here before app bundle reconciliation. |
| Validate schema effects | `scripts/validate.sh` | Downloads Flux schemas and uses kubeconform. |

## CONVENTIONS
- CRDs reconcile before applications through `clusters/<env>/crds.yaml`.
- CRD bases commonly run in `flux-system`; check each `kustomization.yaml` namespace.
- Operator CRDs are split out even when the operator app also has manifests in `apps/base`.
- `kubeconform` uses default schemas plus downloaded Flux CRD schemas and ignores missing schemas.

## ANTI-PATTERNS
- Do not bury CRD installation inside app overlays when an operator needs APIs available before bundles.
- Do not assume CRD base and app base are the same directory; linstor and other operators have both.
- Do not remove `-ignore-missing-schemas` from validation without first adding schemas for every custom resource in use.

## UNIQUE STYLES
- `crds/base` is an explicit extra layer beyond the README’s simple cluster CRD examples.
- Some CRD bases use HelmRelease to install CRDs; others may reference external Git sources.
- Environment CRD directories are small selectors over `crds/base`, not full copies.
