# GHCR images and Arcane

The [images workflow](https://github.com/nabbl/catbots/actions/workflows/images.yml)
tests the engine and builds the engine and dashboard before publishing all three
images for `linux/amd64` and `linux/arm64`:

- `ghcr.io/nabbl/catfolio-engine`
- `ghcr.io/nabbl/catfolio-web`
- `ghcr.io/nabbl/catfolio-backup`

Pushes to `main` publish `edge` and the full source commit SHA. Tags beginning with
`v` publish the version without `v`, `latest`, and the full source commit SHA.
The workflow can also be run manually on `main` or a version tag. Pull requests
run validation only. Publishing uses the repository's `GITHUB_TOKEN` with
`packages: write`; no additional publishing secret is needed. Each image has
source/revision labels, and job summaries record its published digest.

The deployable project lives in
[`nabbl/gitops-arcane/catfolio-docktail`](https://github.com/nabbl/gitops-arcane/tree/main/catfolio-docktail).
In Arcane create a Pull Git Sync from that repository's `main` branch with Compose
path `catfolio-docktail/compose.yaml`. Set private configuration in Arcane's
Environment editor using that folder's `.env.example` as the starting point.
The project uses prebuilt images, DockTail HTTPS, and persistent named volumes;
it forces paper mode. Local `docker-compose.yml` continues to build locally.

The [Argon deployment update](https://github.com/nabbl/gitops-arcane/pull/11)
selects Gemini/upstream and passes only the Gemini key, optional owner key and
required paper/container settings to the engine. Set `GEMINI_API_KEY` in Arcane;
keep `COACH_ADMIN_TOKEN` only if manual owner controls are needed. Model, budget,
pacing and trading/risk settings use application defaults, and the trainer is
benched. The GitOps README lists obsolete environment variables that can be
removed; its allowlist ignores them even if still saved in Arcane. Merge and
sync that deployment update before relying on the allowlist.

The local `.env.example` is also minimal, but local Compose still loads `.env`
and permits explicit overrides and legacy providers. Neither example edits an
existing private environment. Use the GitOps folder's example for Arcane.

New GHCR packages are private by default. Configure `ghcr.io` in Arcane's container
registries with username `nabbl` and a GitHub classic PAT with `read:packages`, or
change all three packages to public in their GitHub package settings. Git repository
authentication and registry authentication are separate. Never commit registry or
provider credentials.

For controlled upgrades, update all three image references in the GitOps Compose
file to the successful build's full source commit SHA (or pin each image by its
digest), then sync and redeploy through Arcane.
The Argon deployment update pins versions directly in Compose and ignores the
former `CATFOLIO_IMAGE_TAG` override. Keep the database volume through upgrades
and do not run two engines against the same database.

References: [Arcane Git Sync](https://getarcane.app/docs/docker/git-sync) and
[GHCR authentication](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry).
