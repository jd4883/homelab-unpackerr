# 📦 Unpackerr chart — CUE splitting, 1Password, Argo CD, release automation

**Branch:** `feature/unpackerr-chart-cue-splitting` (or your branch name)

---

## TL;DR

| What | Why safe | Proof |
|------|----------|--------|
| New Unpackerr Helm chart + Lidarr README CUE section; *arr URLs by namespace; 1Password + Argo CD + release workflows | Additive only; no changes to existing chart defaults or running workloads; render and lint pass | `helm template unpackerr . -f values.yaml -n unpackerr` ✅ |

---

## Summary

| Change | Description |
|--------|-------------|
| 📦 | **Unpackerr chart** (`homelab/helm/unpackerr`): bjw-s app-template, onepassworditem, *arr URLs `lidarr.lidarr.svc.cluster.local`, etc., FLAC+CUE splitting (unstable image), same download PVC layout as Lidarr. |
| 🔐 | **Secret handling:** Chart dependency **onepassworditem** syncs 1Password item `vaults/Kubernetes/items/unpackerr` → secret **unpackerr** (envFrom). **onepassword-secrets** values updated with **unpackerr** namespace. |
| 🌐 | **Namespaces/URLs:** *arr URLs aligned with Argo CD project = namespace: `lidarr.lidarr`, `radarr.radarr`, `sonarr.sonarr`, `readarr.readarr`. Default deploy namespace **unpackerr**. |
| 📖 | **Lidarr README:** CUE splitting section (Lidarr #515, Unpackerr #141, Splittarr); link to unpackerr chart. |
| 🚀 | **Argo CD:** New project **unpackerr** in `config.yaml` (path `"."`, sourceRepo `jd4883/homelab-unpackerr`). |
| 📝 | **Release notes / automation:** GitHub Actions `release-on-merge-unpackerr.yml` and `release-notes-unpackerr.yml`; CHANGELOG.md for unpackerr chart. |

---

## Setup requirements

- **1Password:** Create item `vaults/Kubernetes/items/unpackerr` with fields named `UN_LIDARR_0_API_KEY` (and optionally `UN_RADARR_0_API_KEY`, etc.). Values = API keys from each *arr.
- **Download PVCs** in namespace **unpackerr** (same names as qBittorrent/SABnzbd); replicate or create before deploy.
- **Namespace unpackerr** created by Argo CD project when applied.

---

## Render & validation

> `cd homelab/helm/unpackerr && helm dependency update && helm template unpackerr . -f values.yaml -n unpackerr`

| Check | Result |
|-------|--------|
| `helm dependency update` | ✅ |
| `helm template unpackerr . -f values.yaml -n unpackerr` | ✅ |
| `helm lint .` | ✅ |

---

## Supporting evidence

<details>
<summary>🔍 Unpackerr deployment env (excerpt)</summary>

```yaml
- name: UN_LIDARR_0_URL
  value: http://lidarr.lidarr.svc.cluster.local:8686
- name: UN_LIDARR_0_PATHS_0
  value: /downloads
envFrom:
- secretRef:
    name: unpackerr
```

</details>

<details>
<summary>🔍 OnePasswordItem CR (excerpt)</summary>

```yaml
apiVersion: onepassword.com/v1
kind: OnePasswordItem
metadata:
  name: unpackerr
  namespace: unpackerr
spec:
  itemPath: vaults/Kubernetes/items/unpackerr
```

</details>

---

## Why safe & correct

| Change | What we did | Why it's safe | Proof in render |
|--------|-------------|---------------|------------------|
| Unpackerr chart | New chart only; no edits to existing charts’ defaults | Additive | New manifests only |
| *arr URLs | Use `lidarr.lidarr`, `radarr.radarr`, etc. | Matches Argo CD project = namespace | Env vars in deployment |
| 1Password | onepassworditem + onepassword-secrets entry | Same pattern as lidarr/prowlarr | OnePasswordItem CR + values |
| Argo CD | New project **unpackerr** | New app only | config.yaml |
| Workflows | New workflow files for unpackerr | Paths filter to `homelab/helm/unpackerr/**` | No impact on other charts |

---

## Next steps

1. Create 1Password item **unpackerr** with `UN_LIDARR_0_API_KEY` (and other *arr keys if used).
2. Ensure download PVCs exist in namespace **unpackerr** (replicator or manual).
3. Merge PR; Argo CD syncs when project is applied. Optionally run release-notes workflow after first release.
4. (Optional) Pin image tag from `unstable` to a stable version when Unpackerr releases CUE splitting in stable.
