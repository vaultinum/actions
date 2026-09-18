# vaultinum/actions

Shared GitHub Actions for Vaultinum repositories: reusable workflows (`workflow_call`) and composite actions for CI, release, and deployment (Portainer/Docker Swarm, GCP Cloud Functions).

## Reusable workflows

| Workflow | Purpose | Key inputs | Secrets |
|---|---|---|---|
| `ci.yaml` | Node/Bun CI: install, lint, build, test, then SonarQube + PR size labeling | `run_eslint`, `run_build`, `run_tests`, `run_build_runtime`, `brand`, `working-directory` | `NPM_PACKAGE_ACCESS`, `GH_TOKEN`, `SONAR_TOKEN`, `SONAR_URL` |
| `ci-rust.yaml` | Rust CI: fmt, clippy, build, test | `run_fmt`, `run_clippy`, `run_build`, `run_tests`, `test_features` | `GH_TOKEN` |
| `release.yaml` | Node/Bun release: changelog, tag, npm publish | `run_build_runtime`, `publish_package`, `brand` | `NPM_PACKAGE_ACCESS`, `GH_TOKEN`, `SONAR_TOKEN`, `SONAR_URL` |
| `release-rust.yaml` | Rust release: version bump via release-plz, multi-OS build matrix, optional macOS signing/notarization and Windows Azure Artifact Signing, GitHub Release | `binary_name`, `extra_feature`, `azure_*` | `GH_TOKEN`, `NPM_PACKAGE_ACCESS`, `MACOS_CERT_*`, `APPLE_API_*`, `LAUNCHER_TOKEN` |
| `publish-snapshot.yaml` | Publish an npm pre-release snapshot and post the install command as a PR comment | `snapshot-suffix`, `brand` | `NPM_PACKAGE_ACCESS`, `GH_TOKEN`, `SONAR_TOKEN`, `SONAR_URL` |
| `deploy-portainer.yaml` | Deploy a single Docker container to one or more Portainer endpoints (by tag or direct endpoint) | `image_name`, `docker_component`, `deploy_environment`, `deploy_environment_tag`, `volumes`, ports | `PORTAINER_URL`, `PORTAINER_API_KEY`, `PORTAINER_REGISTRY_AUTH`, `GH_TOKEN` |
| `deploy-stack.yaml` | Deploy/update a Docker Swarm stack via the Portainer API, using the caller repo's `docker-compose.yml` | `image_name`, `stack_name`, `deploy_environment`, `deploy_environment_tag`, `secrets_suffix` | `PORTAINER_URL`, `PORTAINER_API_KEY` |
| `get-portainer-endpoints.yaml` | Internal: resolves target Portainer endpoint ID(s) from `deploy_environment_tag`. Shared by both Portainer deploy workflows. | `deploy_environment_tag` | `PORTAINER_URL`, `PORTAINER_API_KEY` |
| `build-test-deploy-functions.yaml` | Build, test and deploy Firebase/GCP Cloud Functions (`functions/` dir) | `run_tests`, `run_deploy`, `functions_environment` | `NPM_PACKAGE_ACCESS`, `GH_TOKEN`, `GCP_SA_KEY`, `STRIPE_TEST_TOKEN` |
| `unit-test-functions.yaml` | Unit tests for Cloud Functions (api/common/services suites) | `vaultinum_owner_account_id` | `NPM_PACKAGE_ACCESS`, `STRIPE_TEST_TOKEN` |
| `sonarqube.yaml` | SonarQube scan; skipped automatically when no relevant source changed | `run_tests`, `brand`, `working-directory` | `SONAR_TOKEN`, `SONAR_URL` |
| `chromatic.yaml` | Publish Storybook to Chromatic; skipped when no relevant change | – | `GH_TOKEN`, `CHROMATIC_PROJECT_TOKEN`, `NPM_PACKAGE_ACCESS` |
| `e2e-tests.yaml` | Dispatch the E2E suite in `vaultinum-e2e` | `application`, `url`, `sleep` | `e2e_access_token` |
| `staging-deployment.yaml` | `develop` → `staging` changelog/merge, then opens the `staging` → `main` PR | `run_tests` | `NPM_PACKAGE_ACCESS`, `GH_TOKEN`, `E2E_ACCESS_TOKEN` |
| `open-deployment-pr.yaml` | Open a draft deployment PR to an arbitrary target branch | `target_branch`, `pr_title`, `pr_body` | `GH_TOKEN`, `SONAR_TOKEN`, `SONAR_URL` |
| `open-staging-pr.yaml` | **Deprecated** — use `open-deployment-pr.yaml` instead | – | `GH_TOKEN`, `SONAR_TOKEN`, `SONAR_URL` |
| `update-deployment-tag.yaml` | Move an environment tag (`staging`, `prod`, ...) to a release tag; returns the previously deployed version | `version`, `environment` | `GH_TOKEN` |
| `labeler.yaml` | Label PRs by diff size | – | `GH_TOKEN` |
| `merge-main.yaml` | Auto-increment tag on push to this repo's own `main` (not reusable) | – | – |

## Composite actions

- `ci/`, `ci-node/`, `ci-bun/`, `ci-rust/` — install/lint/build/test steps used by the CI workflows above.
- `check-changes/` — detects whether a given path pattern changed; used to skip SonarQube/Chromatic when irrelevant.
- `deploy-functions/` — deploys Cloud Functions to GCP.
- `generate-suffix/` — generates the snapshot version suffix.
- `clean-old-packages/`, `clean-old-dependencies-prs/` — GitHub Packages / Dependabot PR housekeeping.

## Versioning

Downstream repositories pin to a tag, e.g. `uses: vaultinum/actions/ci@v47` or `uses: vaultinum/actions/.github/workflows/ci.yaml@v47`. Workflows in this repo that call each other locally (e.g. `ci.yaml` → `./.github/workflows/sonarqube.yaml`) always resolve against the current ref, not the last tagged `vNN`. Keep this in mind when testing a change to a reusable workflow from a branch: consumers pinned to a tag won't see it until a new tag is cut.

## Linting

Every push/PR touching `.github/workflows/**` or `**/action.yaml` runs [actionlint](https://github.com/rhysd/actionlint) (`lint.yaml`) to catch YAML and expression errors before they reach downstream consumers.

## Deployment environments

`deploy-portainer.yaml`, `deploy-stack.yaml` and `build-test-deploy-functions.yaml` set a GitHub [`environment:`](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment) on their deploy job (from `deploy_environment` / `functions_environment`). Configure protection rules (required reviewers, wait timers) per environment in each downstream repo's Settings → Environments if you want deploys gated in the GitHub UI — this repo only sets the environment name, it doesn't create the protection rules.
