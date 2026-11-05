# homelab-workflows

Reusable GitHub Actions workflows for my homelab. Public on purpose: GitHub only lets public
repositories call reusable workflows that live in a public repository.

## `pr-preview`

Builds the image of a pull request as `ghcr.io/<owner>/<repo>:pr-<head sha>` and comments the preview
URL on the PR. The homelab (an ArgoCD ApplicationSet in a private repo) deploys it at
`https://<repo>-pr-<N>.brunocabado.com`, reachable only from my LAN / tailnet.

To enable it in a repository:

1. A `Dockerfile` at the root serving HTTP on port **8080** (the container gets `PORT=8080`).
2. The GitHub topic `homelab-preview` and a `preview` label.
3. This caller in `.github/workflows/pr-preview.yml`:

```yaml
name: pr-preview
on:
  pull_request:
    types: [opened, synchronize, reopened, labeled]
jobs:
  preview:
    if: contains(github.event.pull_request.labels.*.name, 'preview')
    uses: Kr4is/homelab-workflows/.github/workflows/pr-preview.yml@main
    permissions:
      contents: read
      packages: write
      pull-requests: write
```

Then add the `preview` label to a PR. Removing the label, closing or merging the PR deletes the preview.
Forks are skipped (their token cannot push to GHCR).
