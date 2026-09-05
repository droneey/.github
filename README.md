# .github

The organisation defaults of droneey: reusable GitHub workflows, and the community files every repository inherits. Pin an exact release, for example `@v1.4.0`; Dependabot opens the pull request when a newer one exists. Releases are immutable, nothing here is retagged.

## ⚙️ Workflows

### 🏷️ cd-version

Push to `main`: the merged branch prefix decides the bump (`feature/*` minor, `fix/*`, `hotfix/*` and `dependabot/*` patch), every listed `package.json` receives the version, and `chore: Release vX.Y.Z` is pushed together with the `vX.Y.Z` tag.

```yaml
name: 🏷️ Version
on:
  push:
    branches: [main]
concurrency:
  group: version
  cancel-in-progress: false
jobs:
  version:
    if: "!startsWith(github.event.head_commit.message, 'chore: Release v')"
    uses: droneey/.github/.github/workflows/cd-version.yml@v1.4.0
    permissions:
      contents: write
      pull-requests: read
    secrets:
      token: ${{ secrets.GH_TOKEN }}
```

| Input / secret | Default | Meaning |
|---|---|---|
| `packages` | `package.json` | Space-separated globs of the `package.json` files to bump; the first holds the current version |
| `token` | required | A personal access token, because a tag pushed with `GITHUB_TOKEN` starts no other workflow |

### 🔖 cd-release

Opens the GitHub release for the tag that triggered it, with notes listing the packages and the `feat` and `fix` subjects since the previous tag. A second run edits the release instead of failing, so a re-run after a fixed deploy is safe.

```yaml
name: 🔖 Release
on:
  push:
    tags: ['v*']
jobs:
  release:
    uses: droneey/.github/.github/workflows/cd-release.yml@v1.4.0
    permissions:
      contents: write
```

| Input | Default | Meaning |
|---|---|---|
| `packages` | none | Space-separated globs of the `package.json` files named in the notes; empty leaves the section out |
| `prerelease` | `false` | Open it as a pre-release, to be promoted by hand |

The notes themselves come from `.github/actions/changelog`, a composite action taking `tag` and `packages` and returning `body`; both release workflows use it, so the changelog exists once.

### 🔖 cd-pre-release

There is no separate pre-release workflow: a repository's `cd-pre-release.yml` calls `cd-release.yml` with `prerelease: true`, so a tag push opens a pre-release that a human promotes.

```yaml
name: 🔖 Pre-release
on:
  push:
    tags: ['v*']
jobs:
  pre-release:
    uses: droneey/.github/.github/workflows/cd-release.yml@v1.4.0
    permissions:
      contents: write
    with:
      packages: package.json
      prerelease: true
```

### 📤 cd-deploy-npm

Builds every matched package that has a `build` script, then publishes every one that is not `private`. Provenance is attempted first and dropped on refusal, and a version already on the registry is a skip rather than a failure.

```yaml
name: 🚀 Deploy
on:
  release:
    types: [released]
jobs:
  deploy:
    uses: droneey/.github/.github/workflows/cd-deploy-npm.yml@v1.4.0
    permissions:
      contents: read
      id-token: write
    secrets:
      npm_token: ${{ secrets.NPM_TOKEN }}
```

| Input / secret | Default | Meaning |
|---|---|---|
| `packages` | `package.json` | Space-separated globs of the `package.json` files to build and publish |
| `node-version` | `24` | The Node version that publishes |
| `npm_token` | required | An npm automation token with publish rights |

## 📄 License

MIT
