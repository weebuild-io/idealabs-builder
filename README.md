# idealabs-builder

CI release pipeline for a **private** monorepo (`weebuild-io/idealabs-vn`).

This repo is public so the release workflow runs on GitHub-hosted runners with
public-repo minutes (ubuntu + windows + macos). It contains **no source code**:
the workflow checks out the private repo with a fine-grained PAT and publishes
build output to Cloudflare (Workers / D1 / R2). Nothing is ever published to
this repo — no artifacts, no releases, no packages.

Workflow logs are public; steps are kept quiet and all credentials live in
Actions secrets.

## Trigger

```sh
# full release (check → version → server → dashboard → desktop mac+win)
gh workflow run release.yml -R weebuild-io/idealabs-builder

# examples
gh workflow run release.yml -R weebuild-io/idealabs-builder \
  -f desktop=win -f run-server=false -f run-dashboard=false   # desktop win only
gh workflow run release.yml -R weebuild-io/idealabs-builder \
  -f run-version=false                                        # redeploy current versions
```

From the private repo: `bun run release:ci`.

## Required secrets

| Secret | Purpose |
| --- | --- |
| `PRIVATE_REPO_PAT` | Fine-grained PAT, repo `weebuild-io/idealabs-vn` only, **Contents: read & write** (write = version-bump commit + tags) |
| `CLOUDFLARE_API_TOKEN` | Scoped token: Workers Scripts Edit + D1 Edit on the prod account |
| `R2_ACCESS_KEY_ID` / `R2_SECRET_ACCESS_KEY` | R2 object read/write — desktop artifact upload + prune |
| `APPLE_ID` / `APPLE_APP_SPECIFIC_PASSWORD` / `APPLE_TEAM_ID` | macOS notarization |
| `CSC_LINK` / `CSC_KEY_PASSWORD` | Base64-encoded Developer ID `.p12` + its password — macOS code signing |

## Hardening

- `permissions: {}` — jobs never use this repo's `GITHUB_TOKEN`.
- Issues / wiki / projects disabled; fork-PR workflow runs require approval.
- Only users with write access can dispatch the workflow.
