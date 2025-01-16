# precommit-checks-action

Runs [pre-commit](https://pre-commit.com) checks within GitHub Actions,
allowing you to use your pre-commit hooks as a continuous integration check.

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

      - uses: griceturrble/precommit-checks-action@v1
        with:
          # All arguments below are optional
          # and are shown with their default values:

          # Version of Python to use when installing pre-commit
          python_version: "3.12"
          # Version of pre-commit to install
          pre_commit_version: "4.0.1"
```

The action is a simple call to `pre-commit run --all-files`.
It caches the pre-commit environment
(using the `python_version` and `pre_commit_version` inputs
along with the hashed content of your `.pre-commit-config.yaml` file
as the cache key)
for faster successive runs.
