# siege-perilous-ux

Shared design system for the Siege Perilous tool ecosystem. CSS tokens, base styles, and common components are maintained here and injected into each app at build time — so deployed HTML files remain fully self-contained and downloadable for offline use.

## How it works

```
siege-perilous-ux          tools-site (and other app repos)
──────────────────         ────────────────────────────────
siege.css   ──────────▶   GitHub Action fetches siege.css
                           Python injects it into index.html
                           Built file deployed to gh-pages
                           User downloads one self-contained HTML file
```

Each app's `index.html` source contains a single marker comment where the shared CSS should be inserted:

```html
<style>
  /* @siege-ux-inject */

  /* app-specific styles below... */
</style>
```

At build time, the marker is replaced with the full contents of `siege.css`. The app-specific styles that follow are untouched.

## Triggering downstream rebuilds

When you push a change to `siege.css`, you need to tell each app repo to rebuild. The cleanest way is a `repository_dispatch` event sent from a workflow in this repo.

Add this file to `siege-perilous-ux`:

```yaml
# .github/workflows/notify-consumers.yml
name: Notify consumers

on:
  push:
    branches: [main]
    paths: [siege.css]

jobs:
  notify:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        repo: [tools-site, crusade-manager, army-builder, battle-assistant]
    steps:
      - name: Dispatch to ${{ matrix.repo }}
        uses: peter-evans/repository-dispatch@v3
        with:
          token: ${{ secrets.DISPATCH_PAT }}
          repository: SethGochey/${{ matrix.repo }}
          event-type: ux-updated
```

`DISPATCH_PAT` is a GitHub Personal Access Token (classic) with `repo` scope, added as a secret in this repo's settings. Each consumer repo's workflow already listens for the `ux-updated` event (see `repository_dispatch` trigger in their workflow files).

## Adding a new app

1. Add the `/* @siege-ux-inject */` marker to the app's `index.html` source
2. Copy the `build-and-deploy.yml` workflow from an existing app repo
3. Add the new repo to the `matrix.repo` list in `notify-consumers.yml` above

## What lives here vs in each app

| Here (`siege.css`) | App's `index.html` |
|---|---|
| CSS custom properties (`:root`) | App-specific layout classes |
| Body/reset styles | View-specific components |
| Header & footer base styles | App navigation styles |
| Shared utility classes (`.divider`) | Print styles |
| Scrollbar styling | Any overrides |
