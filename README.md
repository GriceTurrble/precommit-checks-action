# precommit-checks-action

Runs [pre-commit](https://pre-commit.com) checks within GitHub Actions,
allowing you to use your pre-commit hooks as a continuous integration check.

Optionally allows automatic fixes to appear as suggested changes in a PR review.

## Usage

Include `precommit-checks-action` in a GitHub Actions workflow:

```yaml
# .github/workflows/ci.yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  pre-commit-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: griceturrble/precommit-checks-action@v3
```

The action is a simple call to `pre-commit run --all-files`.
It caches the pre-commit environment
(using the `python_version` and `pre_commit_version` inputs
along with the hashed content of your `.pre-commit-config.yaml` file
as the cache key)
for faster successive runs.

If you enable the `suggest_fixes` option,
the action will invoke [reviewdog/action-suggester]
to suggest any changes that your pre-commit checks write to the file system.

### Action inputs

All inputs are optional with appropriate defaults:

| Name                 | Description                                                                                          | Default        |
| -------------------- | ---------------------------------------------------------------------------------------------------- | -------------- |
| `token`              | Token to use for authenticating to GitHub.                                                           | `GITHUB_TOKEN` |
| `python_version`[^1] | Version of Python to use when installing pre-commit                                                  | `"3.14"`       |
| `pre_commit_version` | Version of pre-commit to install                                                                     | `"4.4.0"`      |
| `suggest_fixes`      | Whether to create a PR review suggesting fixes if pre-commit checks fail (set to `"true"` to enable) | `"false"`      |

[^1]:
    If your `.pre-commit-config.yaml` file requests a specific `default_language_version.python` version,
    please ensure this version matches what the workflow will install in the GitHub Action runner.
    Otherwise, your checks may fail because the requested python version
    cannot be found in the runner's environment.

## Auto-suggesting fixes

If your pre-commit checks are configured to auto-fix issues,
you may set the `suggest_fixes` option to `"true"`
to have the workflow suggest those changes as review comments on your PR.

> [!note]
> This requires `contents: write` and `pull-requests: write` permissions for the workflow
> in order to write changes to the GitHub Action runner's file system
> and to write review comments on a PR.

A complete sample of a working job with this option enabled looks like so:

```yaml
jobs:
  pre-commit-check:
    runs-on: ubuntu-latest
    # Elevating permissions for `contents` and `pull-requests` required
    # when using the `suggest_fixes` option:
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v5
      - uses: griceturrble/precommit-checks-action@v3
        with:
          suggest_fixes: "true"
```

For more on working with suggested changes in a PR,
see GitHub docs: [Applying suggested changes](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/incorporating-feedback-in-your-pull-request#applying-suggested-changes).

### Limitations

- Just as with a normal PR review,
  changes can only be suggested on lines within the scope of the PR.
  If your pre-commit checks make changes to lines or files outside that scope,
  you will need to address them manually.

- Any and all changes made be the workflow job will become suggested edits.
  If you mix in other work in the same workflow that makes file changes,
  you may find suggestions unrelated to your pre-commit checks.
  - If you find this to be a helpful side effect for you,
    consider making a more explicit workflow that makes use of
    [reviewdog/action-suggester] directly.

[reviewdog/action-suggester]: https://github.com/reviewdog/action-suggester
