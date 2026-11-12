# Lab 11 — CI/CD with GitHub Actions

Flask application for Practice 11. Endpoints: `/`, `/healthz`, `/version`.

## Local checks

```sh
python3.12 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
ruff check .
ruff format --check .
pytest -q --junitxml=report-local.xml
python app.py
```

## Pipeline

Pull requests run `lint`, `test (3.11)`, `test (3.12)` and `secrets-scan`.
Test reports are uploaded even on failure. Pushes to main and v* tags also build,
scan and publish the Docker image. Only image has `packages: write`.
Trivy fails on fixable HIGH/CRITICAL findings; exceptions require a real scan,
a reason, owner and review date. No sample CVEs are accepted blindly.

## Demonstration

1. On the CI PR, remove `.strip()` from greet and push: tests must fail.
2. Restore `.strip()` and push to the same PR: tests must pass.
3. Protect main with an active ruleset: require PR, all four checks, block force
   pushes and deletions, no bypass. Verify direct pushes are rejected.
4. Merge PR and inspect image job and actual Trivy results.
5. Publish tag v1.0.0 after the image job passes:

```sh
git switch main
git pull --ff-only
git tag v1.0.0
git push origin v1.0.0
```

6. Check GHCR tags main, sha-… and 1.0.0. Set package visibility to Public.
7. Run the published image with Docker:

```sh
docker run --rm -d --name lab11 -p 8011:8000 ghcr.io/darhanqfzx/lab11-cicd:1.0.0
curl --fail http://localhost:8011/version
docker rm -f lab11
```

Expected version: 1.0.0. Actual CI/Trivy/Docker success requires real runs.
Stretch deployment is optional and requires a VM or a dedicated runner.
