# Lighting Beetle Openinary fork

This fork updates the self-hosted Docker images for the October 2026 Node.js
security release, including CVE-2026-111497.

## Upstream and scope

- Upstream: <https://github.com/openinary/openinary>
- Starting revision: `1f05325dae164c6df91220e37b76bd67277b1aec` (`main`, 23 October 2026).
- Both `docker/full.Dockerfile` and `docker/api.Dockerfile` use
  `node:22.23.4-slim`, pinned to the multi-platform index digest
  `sha256:de170b39439eb0e5a3444ad7d60b015614a381e10fecae2e30ec9e735122f38d`.
- All API, web and final stages use the same Node version so native dependencies
  are built and run against the same Node ABI. The base remains Debian Bookworm.
- Application code, dependency lockfile, persistent data and configuration are
  unchanged. Upstream Cloud images and CI Node settings are outside this patch.

The minimum patched Node 22 release is 22.23.4. See the
[Node.js security release](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases)
and [Vercel advisory](https://vercel.com/kb/bulletin/node-js-security-advisory).

## Publish with GitHub Actions

The normal build runs on GitHub-hosted runners and does not use a developer's
local Docker installation. In Actions, run **Publish Lighting Beetle Openinary**
from `main`. It builds the full image on native amd64 and arm64 runners, verifies
Node 22.23.4, SQLite and image transformation, then publishes to
`ghcr.io/lightingbeetle/openinary` using the workflow's `GITHUB_TOKEN`.

The workflow is manual. It does not build the API-only image or deploy to Coolify.
It replaces the inherited Docker Hub publisher; no author registry credentials
are needed. Action dependencies are pinned to commits.

After the first publication, an organization/package administrator must set the
GHCR package visibility to **Public** in its package settings. Publishing a
package from a public repository does not make the package public automatically.
Once public, it can be pulled anonymously without a registry token.

The combined multi-platform image is tagged `node-22.23.4` and
`sha-<full-source-commit>`. The workflow summary provides an immutable
`ghcr.io/lightingbeetle/openinary@sha256:...` reference; use that digest for
deployment. Architecture tags end in `-amd64` or `-arm64` and are intermediate
outputs, not the combined image.

## Optional local build and verification

Run from the repository root:

```sh
docker build --pull -f docker/full.Dockerfile \
  --build-arg IMAGE_TAG=lb-node-22.23.4 \
  -t openinary-lb:node-22.23.4 .

docker build --pull -f docker/api.Dockerfile \
  --build-arg IMAGE_TAG=lb-node-22.23.4 \
  -t openinary-api-lb:node-22.23.4 .

docker run --rm --network none --entrypoint node openinary-lb:node-22.23.4 --version
docker run --rm --network none --entrypoint node openinary-api-lb:node-22.23.4 --version
```

Both version checks must print `v22.23.4`. Before deployment, also verify startup,
login, upload, media delivery and image/video transformations using disposable
local data. Never point local checks at the production database or storage.

These optional commands build local images; they do not publish an image or
update Coolify. They can consume substantial RAM and disk space; prefer the
GitHub Actions workflow above.

The inherited API-only entrypoint invokes pnpm through Corepack as the `node`
user. Its first startup may download pnpm from `registry.npmjs.org`; a container
with networking disabled cannot perform that download. The full image starts
the compiled applications directly and does not need that startup download.

## Maintenance

Review the four `FROM` pins together when updating the base image. Replace both
the version and multi-platform digest after checking the new official image.
Return to the official Openinary image once its Node version is patched and the
replacement has passed the same checks. Updating this repository alone does not
change any deployed container.
