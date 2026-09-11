# .github

The organisation defaults of droneey: reusable GitHub workflows, and the community files every repository inherits. Pin an exact release, for example `@v2.0.0`; Renovate opens the pull request when a newer one exists. Releases are immutable, nothing here is retagged.

The core knows only git and shell. The version is the latest `vX.Y.Z` tag, release notes come from the commit subjects, and the only place a language appears is a deploy workflow for a specific target.

## ⚙️ Workflows

### 🏷️ cd-version

Push to `main`: the merged branch prefix decides the bump (`feature/*` minor; `fix/*`, `hotfix/*`, `renovate/*` and `dependabot/*` patch), the next version is computed from the latest tag, and the tag is pushed. A repository that keeps the version in its own files hands over one command; what it changes is committed as `chore: Release vX.Y.Z` before the tag.

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
    uses: droneey/.github/.github/workflows/cd-version.yml@v2.0.0
    permissions:
      contents: write
      pull-requests: read
    with:
      version-command: npm version --no-git-tag-version "$VERSION"
    secrets:
      token: ${{ secrets.GH_TOKEN }}
```

| Input / secret | Default | Meaning |
|---|---|---|
| `version-command` | empty | Shell run with `VERSION` set before the tag; its changes become the release commit. Empty tags the merge commit as it is |
| `token` | required | A personal access token, because a tag pushed with `GITHUB_TOKEN` starts no other workflow |

| Stack | `version-command` |
|---|---|
| JavaScript, one package | `npm version --no-git-tag-version "$VERSION"` |
| JavaScript, workspaces | `for f in package.json packages/*/package.json; do jq --arg v "$VERSION" '.version = $v' "$f" > "$f.tmp" && mv "$f.tmp" "$f"; done` |
| Python | `uv version "$VERSION"` |
| Rust | `cargo set-version "$VERSION"` |
| Go, infrastructure | none: the tag is the version |

### 🔖 cd-release

Opens the GitHub release for the tag that triggered it, with the `feat` and `fix` subjects since the previous tag as notes. A second run edits the release instead of failing.

```yaml
name: 🔖 Release
on:
  push:
    tags: ['v*']
jobs:
  release:
    uses: droneey/.github/.github/workflows/cd-release.yml@v2.0.0
    permissions:
      contents: write
```

| Input | Default | Meaning |
|---|---|---|
| `prerelease` | `false` | Open it as a pre-release, to be promoted by hand |

### 🔖 cd-pre-release

A repository's `cd-pre-release.yml` calls `cd-release.yml` with `prerelease: true`: the tag opens a pre-release, a human promotes it, and the promotion triggers the repository's `cd-deploy`.

```yaml
name: 🔖 Pre-release
on:
  push:
    tags: ['v*']
jobs:
  pre-release:
    uses: droneey/.github/.github/workflows/cd-release.yml@v2.0.0
    permissions:
      contents: write
    with:
      prerelease: true
```

### 📤 cd-deploy-npm

One deploy target among those a repository may pick. Builds every matched package that has a `build` script, then publishes every one that is not `private`. Without a token it publishes through npm's trusted publishing: the job's OIDC token is exchanged for a short-lived publish credential, so no secret exists to leak or expire. Provenance is attempted first and dropped on refusal, and a version already on the registry is a skip rather than a failure.

```yaml
name: 🚀 Deploy
on:
  release:
    types: [released]
jobs:
  deploy:
    uses: droneey/.github/.github/workflows/cd-deploy-npm.yml@v2.0.0
    permissions:
      contents: read
      id-token: write
```

| Input / secret | Default | Meaning |
|---|---|---|
| `packages` | `package.json` | Space-separated globs of the `package.json` files to build and publish |
| `node-version` | `24` | The Node version that publishes; trusted publishing needs the npm it ships to be 11.5.1 or newer |
| `npm_token` | none | A publish token, only for a registry without trusted publishing or the first version of a package that does not exist yet |

On npmjs.com every package carries a trusted publisher: GitHub Actions, organisation `droneey`, the repository, and the file name of the **calling** workflow, `cd-deploy.yml`, with no environment. A brand-new package is published once by hand, then gets its trusted publisher like the others.

## 🔄 Renovate

`default.json` is the fleet's Renovate preset: on the first day of each month, in the morning Warsaw time, one pull request per repository for the minor and patch updates, one per major, the lockfile refreshed the same day; `chore: Update …` commits in the fleet's format, caret ranges bumped. Security updates ignore the schedule, and the Dependency Dashboard issue of a repository triggers any update on demand. A repository opts in with one file:

```json
// renovate.json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>droneey/.github"]
}
```

`renovate-config.json` points at the same preset, so a repository Renovate onboards on its own gets it too. The Renovate GitHub App is installed once on the organisation.

## 📄 License

MIT
