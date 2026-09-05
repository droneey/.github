# workflows

Reusable GitHub workflows shared by droneey repositories. Pin the floating major: `@v1`.

## ⚙️ Workflows

### 🏷️ cd-version

Push to `main`: the merged branch prefix decides the bump (`feature/*` minor, `fix/*` and `hotfix/*` patch), every listed `package.json` receives the version, and `chore: Release vX.Y.Z` is pushed together with the `vX.Y.Z` tag.

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
    uses: droneey/workflows/.github/workflows/cd-version.yml@v1
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

### 🔖 cd-pre-release

Tag `v*`: a GitHub pre-release whose notes list the packages and the `feat` and `fix` subjects since the previous tag. Promote it by hand; the repository's own `cd-deploy` runs on `release: released`.

```yaml
name: 🔖 Pre-release
on:
  push:
    tags: ['v*']
jobs:
  pre-release:
    uses: droneey/workflows/.github/workflows/cd-pre-release.yml@v1
    permissions:
      contents: write
```

| Input | Default | Meaning |
|---|---|---|
| `packages` | `package.json` | Space-separated globs of the `package.json` files named in the notes |

## 📄 License

MIT
