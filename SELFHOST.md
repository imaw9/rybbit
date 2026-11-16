# Self-hosted client fork

This is a modified version of [Rybbit](https://github.com/rybbit-io/rybbit),
maintained by imaw9. The self-hosted navigation and build changes were added on
2026-11-11. The project remains licensed under [AGPL-3.0](LICENSE.md).

## Available analytics

The client built by this fork exposes Pages, Bots, Performance, Query and
Dashboards in the sidebar and command palette. Web Vitals can also be configured
when creating or editing a web site. Mobile sites continue to hide browser
performance settings.

The build sets `NEXT_PUBLIC_DEPLOYMENT=selfhost` and
`NEXT_PUBLIC_CLOUD=false`. Cloud authentication, billing and subscription rules
remain unchanged. The backend continues to enforce authentication and site
access. Navigation does not enable collection: turn Web Vitals on in the site's
Tracking settings to collect browser performance data.

## Google Search Console

This client exposes Search Console on the main dashboard and the Integrations
tab in Site Settings. Configure Google OAuth on the backend before connecting a
site; displaying the integration does not grant access to Google data.

Create a dedicated Google Cloud project, enable the Search Console API, and
create a Web application OAuth client. Register
`https://your.domain.com/api/gsc/callback` as its authorized redirect URI. Set
`GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET` in the compose project's `.env`.
The compose file passes these values to the backend and defaults the callback
URI to `${BASE_URL}/api/gsc/callback`.

For a separately managed backend container, supply these environment variables:

- `GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET`: the OAuth client credentials.
- `GOOGLE_REDIRECT_URI`: the exact authorized redirect URI.
- `SERVER_URL` and `CLIENT_URL`: the public Rybbit origin, matching `BASE_URL`, so
  the backend's OAuth redirects resolve correctly.

Keep the client secret outside git and recreate the backend container after
updating its environment. In Site Settings > Integrations, connect the Google
account that has access to the site's Search Console property and select that
property. The integration requests read-only Search Console access. Each site
has its own connection; organization and site permissions still control who can
connect it and read its search data.

The image uses same-origin API requests and disables public signup in the
client. Keep `DISABLE_SIGNUP=true` on the backend as well. Use the official
backend image; this workflow only publishes the modified client.

## Build and publish

The **Docker Publish** workflow validates pull requests with client tests, lint,
type checking and a production Docker build. Pull requests do not publish images
and receive no package-write permission in this workflow.

After a pull request is merged into `master`, the same validation runs again.
On success, GitHub Actions publishes the client for `linux/amd64` and
`linux/arm64` using the repository's built-in `GITHUB_TOKEN`:

- `ghcr.io/imaw9/rybbit-client:selfhost` tracks the latest validated `master` build.
- `ghcr.io/imaw9/rybbit-client:sha-<full-commit-sha>` identifies its source revision.

Each architecture builds on a native GitHub-hosted runner. The final publishing
job combines both image digests into a multi-platform manifest, so ARM64 builds
do not depend on QEMU emulation.

You can also run **Docker Publish** manually: choose a pull-request branch for
validation only, or `master` to validate and publish. No Docker Hub account or
separate registry secret is required. A newly created GHCR package is private
by default; change its package visibility to public if anonymous pulls are
wanted, or authenticate Docker to pull a private package.

To validate a pull-request branch manually from the CLI:

```sh
gh workflow run docker-publish.yml --repo imaw9/rybbit --ref <branch-name>
```

Publishing does not update the running server. Deploy a reviewed image revision
or digest explicitly, and retain the previous image reference for rollback.

## Corresponding source and upgrades

The image's GitHub footer links point to the exact source commit built by the
workflow. The image also carries source and revision metadata. Keep this source
and its build instructions available to users of the modified version.

When updating from upstream, review the navigation changes, run the checks, and
build a new image before replacing an existing deployment. The fork's `master`
branch can contain unreleased upstream changes; compatibility with an older
backend must be checked before deployment.
