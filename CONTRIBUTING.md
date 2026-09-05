# Contributing

## 🌱 Branches

Branch off `main` as `feature/*`, `fix/*` or `hotfix/*` - the prefixes `cd-version` reads in the repositories that call it. This hub carries no `package.json` and releases by hand; see Releasing.

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

Tag the merged commit `vX.Y.Z`, then move the floating major so pinned consumers pick the release up:

```bash
git tag v1.2.0
git tag -f v1 && git push -f origin v1
git push origin v1.2.0
```

Consumers pin `@v1`, so a change that breaks them is a new major and a new floating tag, never a move of this one.
