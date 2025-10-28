# Flight Operations — Kong Air (API team repository)

This repository is an example of best practices by Kong Professional Services for an API team. It represents the Flight Operations team for the fictional airline "Kong Air" and demonstrates how an API team consumes platform services provided by the Platform Engineering team.

This README explains:

- How this API team consumes platform-provided actions/workflows (`dp-deploy.yaml`, `provision-konnect-resources.yaml`, `publish-api-configuration.yaml`).
- Where OpenAPI specs and documentation live (`apis/`).
- Where requested Konnect resources live (`konnect/resources.yaml`).

## How we consume platform actions / workflows

The Platform team exposes GitHub Actions and reusable workflows from the `KongHQ-CX/kw-platform-ops` repository. Our repo uses three workflows that call those actions to perform common tasks in a controlled and auditable way.

1) `dp-deploy.yaml` — Deploy data-plane

- Purpose: Let the API team request a dataplane deployment (Kong Gateway dataplane) for a given environment.
- How we use it: Trigger the workflow (manual dispatch) and provide `environment` input (e.g., `dev`, `staging`, `prod`).
- What it does under the hood: the workflow retrieves secrets from Vault (via `hashicorp/vault-action`), then calls the platform's `deploy-dp` action from `KongHQ-CX/kw-platform-ops` to perform the deployment with the correct clustering certs, control plane endpoint, and image tag.

2) `provision-konnect-resources.yaml` — Provision Konnect Team resources

- Purpose: Provision team-scoped Konnect resources (control planes, APIs, dashboards, publications, etc.) declared in a `konnect/resources.yaml` file.
- How we use it: Trigger via workflow dispatch and choose action `provision` or `destroy`. The workflow will fetch Vault secrets (system account token) using the configured `KONNECT_TEAM_NAME` role and then call the platform's `provision-konnect-resources` action to apply the requested resources.
- Inputs & expectations:
  - `KONNECT_TEAM_NAME` organization variable should be set for this repo (used to generate Vault role names and secret paths).
  - The workflow expects `konnect/resources.yaml` in the `konnect/` directory — this is the source-of-truth that the platform will reconcile into Konnect.

3) `publish-api-configuration.yaml` — Publish API configuration to a control plane

- Purpose: Publish an API (OpenAPI) and plugins to a specified Konnect control plane.
- How we use it: Manually trigger the workflow (or wire it into CI) and provide inputs:
  - `openapi_spec` (path inside repo, e.g., `apis/flights/openapi.yaml`),
  - `control_plane_name` (e.g., `flight-operations-dev`),
  - `api_team_plugins_path` (path to plugin definitions in the repo).
- What it does: The workflow checks out the repo, retrieves a system account token from Vault, then calls the `publish-api-configuration` action in the platform repo (`KongHQ-CX/kw-platform-ops`) which performs the actual registration/publication of the API to the control plane.

## Where the API specs and docs live

- OpenAPI specs and API docs are under the `apis/` directory in this repo.
  - Example:
    - `apis/flights/openapi.yaml` — Flights API OpenAPI spec
    - `apis/flights/docs/flights-api.md` — Flights API documentation
    - `apis/routes/openapi.yaml` — Routes API OpenAPI spec
    - `apis/routes/docs/routes-api.md` — Routes API documentation

When you update an OpenAPI spec and want it published to Konnect, run the `publish-api-configuration` workflow with the appropriate `openapi_spec` input.

## Requested Konnect resources: `konnect/resources.yaml`

- The file `konnect/resources.yaml` (present in this repo) contains the resources this team requests the platform to provision for them. It follows the conventions expected by the platform action. Typical entries include `konnect.control_plane`, `konnect.api`, `konnect.api_document`, `konnect.api_publication`, `konnect.dashboard`, etc.
- Example (excerpt):

  - `konnect.control_plane` entries for `flight-operations-dev`, `flight-operations-staging`, and `flight-operations-prod` are defined.

- How it is used:
  1. The repository triggers `provision-konnect-resources.yaml` (manual dispatch), or the platform operators run the platform workflow that reads this file.
  2. The workflow retrieves a system account token from Vault and calls the platform action `provision-konnect-resources` with the path to this file.
  3. The platform action translates the YAML into Terraform or API calls (using the Konnect Terraform provider or Konnect APIs) and creates/updates the requested resources in Konnect.

## Security / secrets and required variables

The workflows expect some repository/organization-level variables and secrets to be configured:

- `KONNECT_TEAM_NAME` (repository/organization variable) — used to build Vault paths and identify the team
- `KONNECT_SERVER_URL` (organization variable) — Konnect control plane API URL
- `VAULT_ADDR` (variable) and `VAULT_TOKEN` / OIDC-to-Vault roles — used by `hashicorp/vault-action` to fetch system account tokens
- AWS credentials (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`) if provisioning Terraform state or S3-backed state is involved
- `KUBECONFIG_CONTENTS` (for dataplane deployments) — base64 or raw kubeconfig content

Note: These secrets/vars are usually provided by the platform team at the org level or configured for this repository. The workflows make use of the `hashicorp/vault-action` to fetch system account tokens with the correct role.

## Typical workflows for API team developers

1. Add or update `apis/<api>/openapi.yaml` and any `apis/<api>/plugins` definitions.
2. Run `publish-api-configuration` via the GitHub UI (Actions → Publish API Configuration) and choose the correct `openapi_spec` and `control_plane_name`.
3. If new Konnect resources are required (new control plane entry, dashboard, or publication), add them to `konnect/resources.yaml` and run `provision-konnect-resources` to request provisioning.
4. If you need a dataplane provisioned for development/testing, run `dp-deploy` with the desired `environment` and coordinate with the platform team for kubeconfig and cluster access.

## Where to find platform implementations

- The platform actions this repo calls live in the `KongHQ-CX/kw-platform-ops` repository under `.github/actions/` and are consumed via the `uses:` references in the workflows (for example `KongHQ-CX/kw-platform-ops/.github/actions/publish-api-configuration@main`). Inspect those actions for exact behavior and inputs.
