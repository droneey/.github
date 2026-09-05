# Contributing

## 🌱 Branches

Branch off `main` as `feature/*`, `fix/*` or `hotfix/*` - the prefixes `cd-version` reads in the repositories that call it. The merge bumps the version and tags it, as in every other repository; see Releasing.

## 📝 Commits

One-line Conventional Commits, `type: Subject`, no body. The subject starts with a capital and describes the change, not the file it touched.

## ✅ Checks

Every workflow has to parse before it is pushed, because a broken reusable workflow breaks every repository that pins it:

```bash
for file in .github/actions/*/action.yml .github/workflows/*.yml; do
  bunx --package @action-validator/cli action-validator "$file"
done
```

The same loop runs on every pull request.

## 🔀 Pull requests

One concern per pull request. Fill in the template, keep the README in step with the inputs and secrets a workflow takes, and wait for the check to pass.

## 🚀 Releasing

Merging to `main` bumps `package.json` and pushes `vX.Y.Z`; the tag opens a pre-release. Promoting it to a release moves the floating `v1` tag onto it, which is what pinned consumers follow. Nothing is tagged by hand.
