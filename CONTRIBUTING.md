# Contributing

Every change reaches `main` through a pull request, squash-merged once its check passes. A vulnerability is reported privately, as the security policy says, never in an issue.

| Convention | Rule |
|---|---|
| Issue | Every change starts from one; its number is the `<taskId>` |
| Branch | `feature/<taskId>-<name>`, `fix/…` or `hotfix/…`, off `main`; `<name>` in kebab-case |
| Version | A merged `feature` bumps the minor version; a `fix` or `hotfix`, the patch |
| Commit | One line, `type: Subject`, at most 100 characters: no scope, body or trailers |
| Subject | Imperative and capitalised; it names the change, not the files |
| Type | `feat` `fix` `perf` `refactor` `style` `test` `docs` `build` `ci` `chore` |
| Size | One logical change per commit |
| Check | The command from the README, run before every push; CI runs the same one |
| Pull request | One concern; the description says why |
| Merge | Squash: the pull request title becomes the commit, so it follows the commit format |

For issue 42, the branch and the pull request title:

```text
feature/42-dark-theme
feat: Add the dark theme
```
