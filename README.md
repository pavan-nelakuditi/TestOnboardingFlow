# TestOnboardingFlow

This repository is a small sandbox for exercising the full Postman API onboarding flow end to end:

1. Bootstrap a Postman workspace from an OpenAPI 3.0 spec
2. Generate baseline, smoke, and contract collections
3. Run repo sync to export Postman artifacts back into the repository
4. Validate branch behavior in the upstream actions before approving merges

## What is in this repo

- `specs/dummy-orders-api.openapi.yaml`: dummy OpenAPI 3.0 source-of-truth spec
- `.github/workflows/postman-onboarding-e2e.yml`: manual workflow that runs upstream bootstrap and repo-sync actions directly

## One-time setup

Create these repository secrets before the first run:

- `POSTMAN_API_KEY`: required
- `POSTMAN_ACCESS_TOKEN`: required for the full workspace-link and repo-sync path

Optional repository variable:

- `POSTMAN_TEAM_ID`: explicit team header override for Bifrost-backed calls

## Running the full flow

1. Push this repo so GitHub can serve the spec over HTTPS.
2. Open the `Postman Onboarding E2E` workflow in GitHub Actions.
3. Run it with the default inputs, or override them from the dispatch form.

The workflow defaults `spec-url` to the raw GitHub URL for `specs/dummy-orders-api.openapi.yaml` at the exact commit SHA being run. That keeps bootstrap and repo sync pointed at the same immutable spec revision.

The current workflow intentionally avoids the composite action so it can exercise a specific upstream branch combination directly:

- `postman-cs/postman-bootstrap-action@main`
- `postman-cs/postman-repo-sync-action@feat/monitor-run-once-when-no-cron`

Repo sync writes generated assets back into this repository, including:

- `postman/collections/`
- `postman/environments/`
- `postman/specs/`
- `.postman/resources.yaml`
- `.postman/workflows.yaml`

## Notes

- The onboarding workflow is manual-only on purpose, so repo-sync pushes do not recursively trigger it.
- This test workflow currently sets `generate-ci-workflow: false` so repo-sync does not try to commit a generated workflow file back into `.github/workflows/`.
- The workflow exposes an optional `workspace_team_id` dispatch input because the direct bootstrap action supports `workspace-team-id` for org-mode teams.
- If this repository is private, the default raw GitHub spec URL will not be fetchable anonymously. In that case, override the `spec_url` workflow input with another public HTTPS URL that serves the same spec.
