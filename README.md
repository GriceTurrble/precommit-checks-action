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
      - uses: actions/checkout@v4
      - uses: griceturrble/precommit-checks-action@v2
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
| `python_version`[^1] | Version of Python to use when installing pre-commit                                                  | `"3.13"`       |
| `pre_commit_version` | Version of pre-commit to install                                                                     | `"4.3.0"`      |
| `suggest_fixes`      | Whether to create a PR review suggesting fixes if pre-commit checks fail (set to `"true"` to enable) | `"false"`      |

[^1]:
    If your `.pre-commit-config.yaml` file requests a specific `default_language_version.python` version,
    please ensure this version matches what the workflow will install in the GitHub Action runner.
    Otherwise, your checks may fail because the requested python version
    cannot be found in the runner's environment.

## Auto-suggesting fixes

If your pre-commit checks are configured to auto-fix issues
and the `suggest_fixes` option is set to `"true"`,
fixes will appear as a PR review with suggested changes in their comments
using a call to [reviewdog/action-suggester].

Note this can only suggest changes to lines that are within the scope of the PR.
If your linters fail on files or lines outside the PR's scope
(lines that a reviewer cannot comment on within the PR),
these cannot be marked as suggested changes;
instead, you will want to run linters manually to address those changes.

### Required permissions

The following permissions are **required** if the `suggest_fixes` option is set to `"true"`:

```yaml
permissions:
  # Ability to write pre-commit changes to the GitHub Action runner's file system:
  contents: write
  # Ability to make changes to a Pull Request, such as adding a review and comments:
  pull-requests: write
```

[reviewdog/action-suggester]: https://github.com/reviewdog/action-suggester
