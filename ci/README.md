# CI Setup

The GitHub Actions workflow in this directory needs to be moved to
`.github/workflows/` to activate. This requires a GitHub token with the
`workflow` scope.

```bash
mkdir -p .github/workflows
cp ci/check.yml .github/workflows/check.yml
git add .github/workflows/check.yml
git commit -m "ci: activate conformance check workflow"
git push
```

After activating, this `ci/` directory can be removed.
