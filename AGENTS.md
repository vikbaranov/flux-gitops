# PROJECT KNOWLEDGE BASE

**Generated:** 2026-05-01
**Commit:** 6366e05
**Branch:** main

## OVERVIEW
FluxCD GitOps configuration for a Kubernetes homelab. The repo is manifest-first: Flux, Kustomize, HelmRelease, SOPS, Kind, kubeconform, and GitHub Actions.

## STRUCTURE
```
homelab/
├── apps/                 # reusable app bases + version bundles
├── clusters/             # local/homelab cluster entrypoints and overlays
├── crds/                 # CRDs reconciled before app bundles
├── scripts/validate.sh   # canonical manifest validation
├── Makefile              # local and CI command surface
└── .github/workflows/    # validation, commitlint
```

## WHERE TO LOOK
| Task | Location | Notes |
| --- | --- | --- |
| Add or edit an app default | `apps/base/<app>/` | Shared HelmRelease/HelmRepository/namespace; no env-specific values. |
| Change app version policy | `apps/bundles/<bundle>/<bundle>.yaml` | `dev-flex`, `prod-stable` differ only by version constraints. |
| Change environment behavior | `clusters/<env>/apps/` | Cluster overlays add storage, routes, secrets, values patches, and delete patches. |
| Change Flux roots | `clusters/<env>/{kustomization.yaml,crds.yaml,bundle.yaml}` | CRDs reconcile before app bundle. |
| Change operator CRDs | `crds/base/` then `crds/<env>/` | CRDs are split from app bundles. |
| Validate manifests | `scripts/validate.sh` | Invoked by `make validate` and CI. |
| Track image updates | `renovate.json` | Renovate opens PRs for Docker image bumps detected in cluster values files. |
| Local Kind bootstrap | `Makefile`, `kind-config.yaml`, `clusters/local/` | Requires Flux, kubectl, kind, SOPS age key, GitHub env vars. |

## CODE MAP
| Symbol / Resource | Type | Location | Role |
| --- | --- | --- | --- |
| `prod-stable` | Bundle | `apps/bundles/prod-stable/` | Exact-pinned production HelmRelease versions. |
| `dev-flex` | Bundle | `apps/bundles/dev-flex/` | Wildcard/pre-release local versions. |
| `bundle` | Flux Kustomization | `clusters/*/bundle.yaml` | Points Flux at `clusters/<env>/apps`. |
| `crds` | Flux Kustomization | `clusters/*/crds.yaml` | Points Flux at `crds/<env>` before apps. |
| `validate.sh` | Shell gate | `scripts/validate.sh` | YAML parse, prod pin check, kubeconform, kustomize build. |
| `test.yaml` | GitHub workflow | `.github/workflows/test.yaml` | Manifest validation and linting on PRs. |

## CONVENTIONS
- Three-layer manifest flow: `apps/base/<app>` -> `apps/bundles/<bundle>` -> `clusters/<env>/apps`.
- `apps/base` must stay environment-neutral; put storage, routes, SOPS/ExternalSecret refs, and values patches under `clusters/<env>/apps`.
- Bundle semantics: `dev-flex` = wildcard/pre-release, `prod-stable` = exact semver pins only.
- `scripts/validate.sh` mirrors Flux kustomize-controller with `--load-restrictor=LoadRestrictionsNone`.
- `kubeconform` skips `Secret` because SOPS fields break strict schema validation.
- Flux source includes only `/clusters/`, `/apps/`, and `/crds/`; root docs/tools are ignored by `.sourceignore`.
- Commit messages are checked as Conventional Commits in `.github/workflows/commitlint.yaml`.
- `.gitignore` excludes `AGENTS.md`, `.envrc`, age keys, and secret files; generated instruction files are intentionally local-only unless force-added.

## ANTI-PATTERNS (THIS PROJECT)
- Do not edit `clusters/local/flux-system/gotk-components.yaml` or `gotk-sync.yaml`; Flux generated them and marks them `DO NOT EDIT`.
- Do not put environment-specific values into `apps/base`; use cluster overlays.
- Do not loosen `apps/bundles/prod-stable/prod-stable.yaml`; `make validate` rejects non-pinned versions.
- Do not assume every app directory is active in every cluster; cluster overlays may remove inherited resources with `rm-*` delete patches.
- Do not commit secrets, `.envrc`, or age keys.

## UNIQUE STYLES
- Cluster overlays often use negative files named `rm-*.yaml` with `$patch: delete` to remove inherited bundle members.
- `clusters/homelab/apps/kustomization.yaml` also imports `../misc`, so production cluster-level misc resources are not purely app-local.
- `crds/base/*` is a separate base layer for CRDs, despite README examples emphasizing cluster CRD paths.
- Most apps are HelmRelease-based; `it-tools` and `qbittorrent` include raw Deployment/Service manifests.
- Renovate tracks Docker image tags in cluster values files and opens PRs for bumps.

## COMMANDS
```bash
make validate      # YAML parse, prod pin check, kubeconform, kustomize builds
make bootstrap     # create local Kind cluster and bootstrap Flux
make reconcile     # reconcile Flux source and Kustomizations
make wait          # wait for Flux Kustomizations
make smoke         # podinfo readiness and HTTP probe
make e2e           # bootstrap -> reconcile -> wait -> smoke
make ci-e2e        # CI-flavored bootstrap/wait/smoke flow
make clean         # delete local Kind cluster
```

## NOTES
- No package.json, Docker Compose, Taskfile, Terraform, or conventional unit test suite was found.
- Actual workflows are `test.yaml` and `commitlint.yaml`; README mentions `e2e.yaml`, but that file is absent.
- `.envrc` points at `clusters/local/sops.agekey`; Makefile local bootstrap expects `clusters/local/sops.agekey`.
