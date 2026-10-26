# preview

Shared host for pull request previews across awesome-foundation repos. GitHub
Pages serves the `gh-pages` branch, and every preview lands at:

    https://awesome-foundation.github.io/preview/<repo>/pr-<n>/

`main` holds two reusable workflows. A source repo builds its own site, uploads
it as an artifact, and calls them:

```yaml
permissions:
  contents: read
  pull-requests: write

jobs:
  build:
    if: github.event.action != 'closed'
    # repo-specific: overlay data, install, build, validate,
    # then actions/upload-artifact with name: preview-site

  publish:
    needs: build
    if: github.event.action != 'closed'
    uses: awesome-foundation/preview/.github/workflows/publish.yml@main
    with:
      project: my-repo
      note: Built from this PR.
    secrets: inherit

  teardown:
    if: github.event.action == 'closed'
    uses: awesome-foundation/preview/.github/workflows/teardown.yml@main
    with:
      project: my-repo
    secrets: inherit
```

`secrets: inherit` passes `PREVIEW_DEPLOY_KEY`, which every source repo holds as
a secret. The matching public key is a write deploy key on this repo.

## Adding a source repo

GitHub does not show a secret again after you set it. For that reason, the
private repo `awesome-foundation/preview-keys` keeps the private key as the
Actions variable `PREVIEW_DEPLOY_KEY`. To add a new source repo, copy the key
from that variable into a secret on the source repo:

```sh
gh variable get PREVIEW_DEPLOY_KEY -R awesome-foundation/preview-keys \
  | gh secret set PREVIEW_DEPLOY_KEY -R awesome-foundation/<repo>
```

Actions does not mask variables in logs. No workflow may ever print
`vars.PREVIEW_DEPLOY_KEY`, in `preview-keys` or anywhere else.

## What a source repo must handle itself

The preview is served from a subdirectory, not from a domain root. A site that
writes `/style.css` or navigates to `/en/` will break there. Give the build a
base path, or keep every internal reference relative.

Analytics and any write-back endpoint should be off in a preview. Previews
render data that may never ship, and their taps would land in the live counts.

## History

Preview builds are committed here and never removed from history, so the repo
grows. Nothing here is worth keeping. Force-push an orphan commit any time the
size bothers you.
