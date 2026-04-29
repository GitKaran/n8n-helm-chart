# Upgrade n8n Helm Chart

Upgrade the n8n helm chart to a new version. Pass the target version as an argument (e.g., `/upgrade-n8n 2.18.0`).

## Instructions

You are upgrading the n8n helm chart in this repository (a fork of 8gears/n8n-helm-chart with custom n8n v2 support).

### Step 1: Determine versions

- Read `charts/n8n/Chart.yaml` to get the current `appVersion` (current n8n version) and `version` (chart version).
- The target n8n version is: **$ARGUMENTS**
- If no argument was provided, fetch https://github.com/n8n-io/n8n/releases to find the latest stable version and confirm it with the user.

### Step 2: Review release notes and breaking changes

Fetch and review the n8n release notes between the current version and the target version:

1. Fetch `https://github.com/n8n-io/n8n/releases/tag/n8n@<version>` for each major/minor version between current and target.
2. Focus on:
   - **Breaking changes** that affect deployment (new/changed ports, health endpoints, env vars, worker/webhook architecture changes, Docker image changes)
   - **New environment variables** that should be documented in values.yaml
   - **Docker image changes** (e.g., runner image updates)
   - **Infrastructure changes** (new services, changed scaling behavior)
3. Summarize findings for the user before proceeding.

### Step 3: Determine if template changes are needed

Based on the release notes analysis:
- If there are breaking changes affecting the helm chart templates, **stop and present the required changes to the user** before proceeding. Use EnterPlanMode for complex changes.
- If no template changes are needed, proceed to Step 4.

### Step 4: Update Chart.yaml

Update `charts/n8n/Chart.yaml`:
- `appVersion`: set to the target version
- `version`: bump appropriately:
  - **Patch bump** (e.g., 2.3.0 → 2.3.1): if only bumping n8n patch version with no chart changes
  - **Minor bump** (e.g., 2.3.0 → 2.4.0): if bumping n8n minor/major version or adding chart features
  - **Major bump**: if there are breaking chart schema changes
- Update `annotations.artifacthub.io/changes` to describe what changed

### Step 5: Update values.yaml (if needed)

If the release notes revealed new environment variables, configuration options, or changed defaults that should be reflected:
- Add new configuration options with appropriate comments
- Update comments for changed behavior
- Do NOT change existing defaults unless a breaking change requires it

### Step 6: Validate

Run these commands and verify they pass:
```bash
helm template test-release ./charts/n8n
helm lint ./charts/n8n
```

Confirm:
- Image tag resolves to the target version in rendered output
- No rendering errors
- Lint passes with 0 failures

### Step 7: Create branch and commit

```bash
git checkout -b feat/upgrade-n8n-<target-version>
git add charts/n8n/Chart.yaml
# Add any other modified files
git commit -m "feat: upgrade n8n to <target-version>"
```

### Step 8: Create PR

Create a PR with:
- **Title**: `feat: upgrade n8n to <target-version>`
- **Body**: Include:
  - Summary of version jump (from → to)
  - Key changes from release notes relevant to deployment
  - Whether template changes were needed and why/why not
  - Verification steps performed (helm template, helm lint)

```bash
gh pr create --title "feat: upgrade n8n to <target-version>" --body "$(cat <<'EOF'
## Summary
- Upgraded n8n appVersion from <current> to <target>
- Chart version bumped to <new-chart-version>

## Release Notes Review
<summary of relevant changes between versions>

## Template Changes
<list changes or "No template changes required">

## Verification
- [x] `helm template` renders without errors
- [x] `helm lint` passes
- [x] Image tag resolves to correct version

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

### Important Notes

- This repo is a **fork** of 8gears/n8n-helm-chart. The upstream is still on n8n v1.x, so we maintain v2 support independently.
- Worker deployments use `n8nio/runners` image (configured via `worker.image.repository`), not the main `n8nio/n8n` image.
- Worker health checks are on port 5680 (runners image), main/webhook are on port 5678.
- Readiness probes use `/healthz/readiness` (since n8n 2.13.0+), liveness uses `/healthz`.
