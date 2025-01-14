# precommit-checks-action

Runs [pre-commit](https://pre-commit.com) checks within GitHub Actions

## Usage

Include the following in a GitHub Action workflow:

```yaml
jobs:
  pre-commit-check:
  runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: griceturrble/precommit-checks-action@v1
        with:
          # optional, defaults to 3.12:
          python_version: "3.12"
          # optional, defaults to 4.0.1:
          pre_commit_version: "4.0.1"
```
