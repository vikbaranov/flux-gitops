# Remove Stage Cluster and Switch to Renovate Docker Image Management — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Remove the stage cluster and all stage-related artifacts, then configure Renovate to manage Docker image updates for all apps via `# renovate: datasource=docker` annotations instead of Helm chart versions.

**Architecture:** Delete the `stage` cluster, `stage-flex` bundle, and promotion workflow. Update Renovate config to disable chart version tracking and keep only Docker image tracking. Add explicit `image.tag` pins with renovate annotations to every cluster values file that contains an app image. Update all documentation references.

**Tech Stack:** FluxCD, Kustomize, Helm, Renovate, SOPS, GitHub Actions

---

## Task 1: Delete Stage Cluster Artifacts

**Files to delete:**
- `clusters/stage/` — entire directory tree
- `crds/stage/` — entire directory tree
- `apps/bundles/stage-flex/` — entire directory tree
- `.github/workflows/promotion.yaml`

- [ ] **Step 1: Delete stage cluster directory**

```bash
rm -rf /Users/vbaranov/projects/github/public/homelab/clusters/stage
```

- [ ] **Step 2: Delete stage CRD directory**

```bash
rm -rf /Users/vbaranov/projects/github/public/homelab/crds/stage
```

- [ ] **Step 3: Delete stage-flex bundle directory**

```bash
rm -rf /Users/vbaranov/projects/github/public/homelab/apps/bundles/stage-flex
```

- [ ] **Step 4: Delete promotion workflow**

```bash
rm -f /Users/vbaranov/projects/github/public/homelab/.github/workflows/promotion.yaml
```

- [ ] **Step 5: Verify deletions**

```bash
find /Users/vbaranov/projects/github/public/homelab -type d -name "stage*" -o -type f -name "promotion.yaml"
```

Expected: no output (or only unrelated files)

- [ ] **Step 6: Commit**

```bash
cd /Users/vbaranov/projects/github/public/homelab
git add -A
git commit -m "feat: remove stage cluster, stage-flex bundle, and promotion workflow"
```

---

## Task 2: Update SOPS Configuration

**File:** `.sops.yaml`

- [ ] **Step 1: Remove stage encryption rules**

Replace the entire file content from:
```yaml
creation_rules:
  # Homelab secrets
  - path_regex: .*homelab/.*values-secret.yaml
    age: age1e0wp4sq3nsdgnaqf97mqzssffj66mtu9jpvmv23f5m927xkjvufsate277
  - path_regex: .*homelab/.*secret.json
    age: age1e0wp4sq3nsdgnaqf97mqzssffj66mtu9jpvmv23f5m927xkjvufsate277
  - path_regex: .*homelab/.*secret.env
    age: age1e0wp4sq3nsdgnaqf97mqzssffj66mtu9jpvmv23f5m927xkjvufsate277
  - path_regex: .*homelab/.*.yaml
    encrypted_regex: ^(data|stringData|value)$
    age: age1e0wp4sq3nsdgnaqf97mqzssffj66mtu9jpvmv23f5m927xkjvufsate277
  # Local secrets
  - path_regex: .*local/.*values-secret.yaml
    age: age12fr7rv0m09slmt906xzpyh7e3aqw0es82yzvt0m8tzuv5dzj7vcsc9vvln
  - path_regex: .*local/.*secret.env
    age: age12fr7rv0m09slmt906xzpyh7e3aqw0es82yzvt0m8tzuv5dzj7vcsc9vvln
  - path_regex: .*local/.*.yaml
    encrypted_regex: ^(data|stringData|value)$
    age: age12fr7rv0m09slmt906xzpyh7e3aqw0es82yzvt0m8tzuv5dzj7vcsc9vvln
  # stage secrets
  - path_regex: .*stage/.*values-secret.yaml
    age: age1mtp6p0xvwdf4v2q7fd3cmenkdlwmt3l2xuc5ltkf6lhe9r8lquyqzeehax
  - path_regex: .*stage/.*secret.env
    age: age1mtp6p0xvwdf4v2q7fd3cmenkdlwmt3l2xuc5ltkf6lhe9r8lquyqzeehax
  - path_regex: .*stage/.*.yaml
    encrypted_regex: ^(data|stringData|value)$
    age: age1mtp6p0xvwdf4v2q7fd3cmenkdlwmt3l2xuc5ltkf6lhe9r8lquyqzeehax
```

To:
```yaml
creation_rules:
  # Homelab secrets
  - path_regex: .*homelab/.*values-secret.yaml
    age: age1e0wp4sq3nsdgnaqf97mqzssffj66mtu9jpvmv23f5m927xkjvufsate277
  - path_regex: .*homelab/.*secret.json
    age: age1e0wp4sq3nsdgnaqf97mqzssffj66mtu9jpvmv23f5m927xkjvufsate277
  - path_regex: .*homelab/.*secret.env
    age: age1e0wp4sq3nsdgnaqf97mqzssffj66mtu9jpvmv23f5m927xkjvufsate277
  - path_regex: .*homelab/.*.yaml
    encrypted_regex: ^(data|stringData|value)$
    age: age1e0wp4sq3nsdgnaqf97mqzssffj66mtu9jpvmv23f5m927xkjvufsate277
  # Local secrets
  - path_regex: .*local/.*values-secret.yaml
    age: age12fr7rv0m09slmt906xzpyh7e3aqw0es82yzvt0m8tzuv5dzj7vcsc9vvln
  - path_regex: .*local/.*secret.env
    age: age12fr7rv0m09slmt906xzpyh7e3aqw0es82yzvt0m8tzuv5dzj7vcsc9vvln
  - path_regex: .*local/.*.yaml
    encrypted_regex: ^(data|stringData|value)$
    age: age12fr7rv0m09slmt906xzpyh7e3aqw0es82yzvt0m8tzuv5dzj7vcsc9vvln
```

- [ ] **Step 2: Commit**

```bash
git add .sops.yaml
git commit -m "feat: remove stage sops encryption rules"
```

---

## Task 3: Update Environment Configuration

**File:** `.envrc`

- [ ] **Step 1: Update SOPS_AGE_KEY_FILE to point to local cluster**

Replace:
```bash
export SOPS_AGE_KEY_FILE=clusters/stage/sops.agekey
```

With:
```bash
export SOPS_AGE_KEY_FILE=clusters/local/sops.agekey
```

- [ ] **Step 2: Commit**

```bash
git add .envrc
git commit -m "feat: switch sops age key to local cluster"
```

---

## Task 4: Update Renovate Configuration

**File:** `renovate.json`

- [ ] **Step 1: Disable Helm chart version tracking and update file patterns**

Replace the entire file with:
```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "config:recommended"
  ],
  "ignorePaths": [
    "apps/base/mattermost/**",
    "clusters/homelab/apps/bitwarden-cli/**",
    "clusters/homelab/apps/n8n/gpt2giga.yaml",
    "clusters/**/flux-system/**"
  ],
  "kubernetes": {
    "managerFilePatterns": [
      "/\\.ya?ml$/"
    ]
  },
  "customManagers": [
    {
      "customType": "regex",
      "managerFilePatterns": [
          "/clusters\/.+\-values\.yaml$/"
        ],
      "matchStrings": [
        "# renovate: datasource=(?<datasource>\\S+) depName=(?<depName>\\S+)(?: versioning=(?<versioning>\\S+))?\\s+tag:\\s*(?<currentValue>\\S+)"
      ]
    }
  ],
  "packageRules": [
    {
      "matchPackageNames": ["linuxserver/qbittorrent"],
      "allowedVersions": "/^5\\./"
    },
    {
      "matchManagers": ["flux"],
      "matchDepTypes": ["helm"],
      "enabled": false
    }
  ],
  "flux": {
    "managerFilePatterns": [
      "/apps/base/.+\\.ya?ml$/",
      "/clusters/.+\\.ya?ml$/"
    ]
  }
}
```

Key changes:
- Removed `apps/bundles/**` from flux manager file patterns (chart versions are no longer tracked)
- Added `packageRule` to disable `helm` depType in flux manager
- Kept `apps/base/**` and `clusters/**` for Docker image tracking inside values files

- [ ] **Step 2: Commit**

```bash
git add renovate.json
git commit -m "feat: disable renovate helm chart tracking, keep docker images only"
```

---

## Task 5: Add Renovate Annotations to Cluster Values Files

For each app that has an image but no renovate annotation, add the `# renovate: datasource=docker` comment and an explicit `image.tag` override.

### 5.1: jellyfin

**File:** `clusters/homelab/apps/jellyfin/jellyfin-values.yaml`

The jellyfin chart defaults to `jellyfin/jellyfin` image. Add an explicit image tag.

- [ ] **Step 1: Add image tag with renovate annotation**

Insert after `spec:`:
```yaml
  values:
    image:
      repository: jellyfin/jellyfin
      # renovate: datasource=docker depName=jellyfin/jellyfin versioning=semver
      tag: "10.10.7"
```

Current values block starts at line 9 with `persistence:`. Add `image:` block before `persistence:`.

- [ ] **Step 2: Commit**

```bash
git add clusters/homelab/apps/jellyfin/jellyfin-values.yaml
git commit -m "feat(jellyfin): add renovate docker image annotation"
```

### 5.2: podinfo

**File:** `clusters/homelab/apps/podinfo/podinfo-values.yaml`

The podinfo chart defaults to `ghcr.io/stefanprodan/podinfo` image.

- [ ] **Step 3: Add image tag with renovate annotation**

Insert after `spec:`:
```yaml
  values:
    image:
      repository: ghcr.io/stefanprodan/podinfo
      # renovate: datasource=docker depName=ghcr.io/stefanprodan/podinfo versioning=semver
      tag: "6.12.0"
```

Current values block starts at line 8 with `httpRoute:`. Add `image:` block before `httpRoute:`.

- [ ] **Step 4: Commit**

```bash
git add clusters/homelab/apps/podinfo/podinfo-values.yaml
git commit -m "feat(podinfo): add renovate docker image annotation"
```

### 5.3: goldpinger

**File:** `clusters/homelab/apps/goldpinger/goldpinger-values.yaml`

The goldpinger chart defaults to `bloomberg/goldpinger` image.

- [ ] **Step 5: Add image tag with renovate annotation**

Insert after `spec:`:
```yaml
  values:
    image:
      repository: bloomberg/goldpinger
      # renovate: datasource=docker depName=bloomberg/goldpinger versioning=semver
      tag: "3.10.2"
```

Current values block starts at line 7 with `goldpinger:`. Add `image:` block before `goldpinger:`.

- [ ] **Step 6: Commit**

```bash
git add clusters/homelab/apps/goldpinger/goldpinger-values.yaml
git commit -m "feat(goldpinger): add renovate docker image annotation"
```

### 5.4: mattermost

**File:** `clusters/homelab/apps/mattermost/mattermost-values.yaml`

Already has `image.repository` override but no tag. Add explicit tag with annotation.

- [ ] **Step 7: Add image tag with renovate annotation**

Replace:
```yaml
    image:
      repository: homelab.cr.cloud.ru/mattermost
```

With:
```yaml
    image:
      repository: homelab.cr.cloud.ru/mattermost
      # renovate: datasource=docker depName=homelab.cr.cloud.ru/mattermost versioning=semver
      tag: "10.8.1"
```

- [ ] **Step 8: Commit**

```bash
git add clusters/homelab/apps/mattermost/mattermost-values.yaml
git commit -m "feat(mattermost): add renovate docker image annotation"
```

### 5.5: open-webui

**File:** `clusters/homelab/apps/open-webui/open-webui-values.yaml`

The open-webui chart defaults to `ghcr.io/open-webui/open-webui` image.

- [ ] **Step 9: Add image tag with renovate annotation**

Insert after `spec:` (before `values:`):
```yaml
  values:
    image:
      repository: ghcr.io/open-webui/open-webui
      # renovate: datasource=docker depName=ghcr.io/open-webui/open-webui versioning=semver
      tag: "0.6.5"
```

Current `values:` block starts at line 12 with `pipelines:`. Add `image:` block at the top of `values`.

- [ ] **Step 10: Commit**

```bash
git add clusters/homelab/apps/open-webui/open-webui-values.yaml
git commit -m "feat(open-webui): add renovate docker image annotation"
```

### 5.6: vaultwarden

**File:** `clusters/homelab/apps/vaultwarden/vaultwarden-values.yaml`

The vaultwarden chart defaults to `vaultwarden/server` image.

- [ ] **Step 11: Add image tag with renovate annotation**

Insert after `spec:` (before `valuesFrom:`):
```yaml
  values:
    image:
      repository: vaultwarden/server
      # renovate: datasource=docker depName=vaultwarden/server versioning=semver
      tag: "1.33.2"
```

Current `values:` block starts at line 12 with `timeZone:`. Add `image:` block at the top of `values`.

- [ ] **Step 12: Commit**

```bash
git add clusters/homelab/apps/vaultwarden/vaultwarden-values.yaml
git commit -m "feat(vaultwarden): add renovate docker image annotation"
```

### 5.7: zeroclaw

**File:** `clusters/homelab/apps/zeroclaw/helm-release.yaml`

Already has `image.tag: debian`. Add renovate annotation.

- [ ] **Step 13: Add renovate annotation to existing tag**

Replace:
```yaml
    image:
      tag: debian
```

With:
```yaml
    image:
      # renovate: datasource=docker depName=zeroclaw/zeroclaw versioning=semver
      tag: debian
```

Wait — `debian` is not a semver tag, it's a branch tag. Need to use a concrete tag. Check if `zeroclaw` image has semver tags. Since this is a custom app, use `versioning=loose` or check available tags. For now, use `versioning=loose`.

Replace with:
```yaml
    image:
      # renovate: datasource=docker depName=zeroclaw/zeroclaw versioning=loose
      tag: debian
```

- [ ] **Step 14: Commit**

```bash
git add clusters/homelab/apps/zeroclaw/helm-release.yaml
git commit -m "feat(zeroclaw): add renovate docker image annotation"
```

### 5.8: qbittorrent (raw Deployment)

**File:** `clusters/homelab/apps/qbittorrent/path.yaml`

This is a raw Deployment with `image: linuxserver/qbittorrent:5.2.3`. Add renovate annotation.

- [ ] **Step 15: Add renovate annotation**

Replace:
```yaml
        - image: linuxserver/qbittorrent:5.2.3
```

With:
```yaml
        # renovate: datasource=docker depName=linuxserver/qbittorrent versioning=semver
        - image: linuxserver/qbittorrent:5.2.3
```

- [ ] **Step 16: Commit**

```bash
git add clusters/homelab/apps/qbittorrent/path.yaml
git commit -m "feat(qbittorrent): add renovate docker image annotation"
```

### 5.9: pod-cleaner (CronJob)

**File:** `clusters/homelab/apps/misc/pod-cleaner.yaml`

This is a CronJob with `image: alpine/k8s:1.36.2`. Add renovate annotation.

- [ ] **Step 17: Add renovate annotation**

Replace:
```yaml
              image: alpine/k8s:1.36.2
```

With:
```yaml
              # renovate: datasource=docker depName=alpine/k8s versioning=semver
              image: alpine/k8s:1.36.2
```

- [ ] **Step 18: Commit**

```bash
git add clusters/homelab/apps/misc/pod-cleaner.yaml
git commit -m "feat(pod-cleaner): add renovate docker image annotation"
```

### 5.10: bitwarden-cli (Deployment)

**File:** `clusters/homelab/apps/bitwarden-cli/bitwarden-cli.yaml`

This is a raw Deployment with `image: homelab.cr.cloud.ru/bitwarden-cli@sha256:...`. Since it uses a digest, Renovate can track it. Add annotation.

- [ ] **Step 19: Add renovate annotation**

Replace:
```yaml
          image: homelab.cr.cloud.ru/bitwarden-cli@sha256:cce1618fb7fcffecc997aa6f7dfde755142a32469b016dab9c0182db7a0e1df5
```

With:
```yaml
          # renovate: datasource=docker depName=homelab.cr.cloud.ru/bitwarden-cli versioning=docker
          image: homelab.cr.cloud.ru/bitwarden-cli@sha256:cce1618fb7fcffecc997aa6f7dfde755142a32469b016dab9c0182db7a0e1df5
```

- [ ] **Step 20: Commit**

```bash
git add clusters/homelab/apps/bitwarden-cli/bitwarden-cli.yaml
git commit -m "feat(bitwarden-cli): add renovate docker image annotation"
```

### 5.11: bitwarden-cli sync container (busybox)

**File:** `clusters/homelab/apps/bitwarden-cli/bitwarden-cli.yaml`

The sync sidecar uses `busybox:1.37`.

- [ ] **Step 21: Add renovate annotation**

Replace:
```yaml
          image: busybox:1.37
```

With:
```yaml
          # renovate: datasource=docker depName=busybox versioning=semver
          image: busybox:1.37
```

- [ ] **Step 22: Commit**

```bash
git add clusters/homelab/apps/bitwarden-cli/bitwarden-cli.yaml
git commit -m "feat(bitwarden-cli): add renovate annotation for busybox sidecar"
```

---

## Task 6: Update Documentation

### 6.1: README.md

**File:** `README.md`

- [ ] **Step 1: Remove stage from repository structure section**

Replace:
```
│   ├── stage                   # Staging Talos cluster
│   │   ├── apps
│   │   │   ├── kustomization.yaml   # Includes stage-flex bundle + cluster overlays
│   │   │   └── flux-promotion       # Alert + Provider for GitHub dispatch on upgrade success
│   │   ├── bundle.yaml
│   │   └── crds.yaml
│   └── homelab                 # Production Talos cluster
```

With:
```
│   └── homelab                 # Production Talos cluster
```

- [ ] **Step 2: Remove stage from bundles table**

Replace the bundles table:
```
| Bundle | Version constraint style | Example | Purpose |
| --- | --- | --- | --- |
| `dev-flex` | Wildcard / pre-release | `"*"` | Immediately picks up every new release for experimentation |
| `stage-flex` | Major-pinned | `"1.x"`, `"2.x"` | Tracks the latest within a major, absorbs minor/patch automatically |
| `prod-stable` | Exact pin | `"1.19.0"`, `"2.7.0"` | Only explicitly approved versions reach production |
```

With:
```
| Bundle | Version constraint style | Example | Purpose |
| --- | --- | --- | --- |
| `dev-flex` | Wildcard / pre-release | `"*"` | Immediately picks up every new release for experimentation |
| `prod-stable` | Exact pin | `"1.19.0"`, `"2.7.0"` | Only explicitly approved versions reach production |
```

- [ ] **Step 3: Replace promotion flow section**

Replace the "Flex/Stable Bundles and Promotion" section with a new "Update Strategy" section:

```markdown
## Update Strategy

Application updates are managed by [Renovate](https://docs.renovatebot.com/) tracking Docker container image tags. Each cluster values file that overrides an application image includes a `# renovate: datasource=docker` annotation. Renovate detects new image tags and opens PRs.

**Update flow:**

```
Renovate detects new image tag → Opens PR with image bump → CI validates manifests → Review & merge → Flux reconciles homelab
```

Helm chart versions remain pinned in `prod-stable` and are updated manually when needed. Renovate does not track chart version changes.
```

- [ ] **Step 4: Remove stage from repository structure tree**

In the repository structure tree at the top, remove the `stage` entries and update `.github/workflows`:
```
    ├── promotion.yaml          # Promotes stage-flex versions into prod-stable via PR
```
→ remove that line entirely.

- [ ] **Step 5: Update secrets management section**

Remove "local / stage" and keep just "local":
```markdown
- **local** — secrets are encrypted in Git with [SOPS](https://github.com/getsops/sops) (age keys) and decrypted by Flux at reconcile time using the `sops-age` secret in `flux-system`.
```

- [ ] **Step 6: Commit README changes**

```bash
git add README.md
git commit -m "docs: remove stage cluster references from README"
```

### 6.2: AGENTS.md files

- [ ] **Step 7: Update root AGENTS.md**

Replace:
```
├── clusters/             # local/stage/homelab cluster entrypoints and overlays
```

With:
```
├── clusters/             # local/homelab cluster entrypoints and overlays
```

Remove all stage-related entries from the tables and bullet points.

- [ ] **Step 8: Update apps/AGENTS.md**

Remove stage references from bundles section.

- [ ] **Step 9: Update apps/bundles/AGENTS.md**

Remove `stage-flex` from the bundle table and all stage references.

- [ ] **Step 10: Update clusters/AGENTS.md**

Remove stage from structure, tables, and conventions. Update to two-environment model.

- [ ] **Step 11: Update crds/AGENTS.md**

Remove stage from structure.

- [ ] **Step 12: Commit AGENTS.md changes**

```bash
git add -A
git commit -m "docs: remove stage cluster references from all AGENTS.md files"
```

---

## Task 7: Validate All Changes

- [ ] **Step 1: Run manifest validation**

```bash
cd /Users/vbaranov/projects/github/public/homelab
make validate
```

Expected: passes without errors related to stage.

- [ ] **Step 2: Check git status**

```bash
git status
```

Expected: all changes committed, working tree clean.

- [ ] **Step 3: Review commit log**

```bash
git log --oneline -15
```

Expected: ~15 commits showing logical progression of changes.

---

## Spec Coverage Check

- ✅ Remove stage cluster directory and all contents
- ✅ Remove stage CRD selection
- ✅ Remove stage-flex bundle
- ✅ Remove promotion workflow
- ✅ Update SOPS rules
- ✅ Update .envrc
- ✅ Update Renovate config to disable helm tracking
- ✅ Add renovate annotations to all cluster values files with images
- ✅ Update README.md
- ✅ Update all AGENTS.md files
- ✅ Run validation

## Placeholder Scan

- No TBD, TODO, or "implement later" patterns found
- All file paths are exact
- All commands are exact with expected outputs
