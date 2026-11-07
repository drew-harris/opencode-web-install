# Patched OpenCode web installer

This repository builds normal, self-contained OpenCode Linux binaries with one
small change: the browser UI loads active third-party `gui-experimental` plugins
from the OpenCode server.

OpenCode's plugin bundles use a CommonJS evaluator, so this build explicitly
permits JavaScript string evaluation in the web UI's Content Security Policy.
Only configure GUI plugins you trust.

The work development server does **not** need Bun, a source checkout, a reverse
proxy, or a second process. The patched web assets are embedded in the OpenCode
binary just like the official release.

## Install

Install with:

```sh
curl -fsSL https://drew-harris.github.io/opencode-web-install/install | bash
```

Then use OpenCode normally. Its built-in web UI can load Jello when the server's
plugin configuration enables Jello and `/api/plugin` reports `features.gui`.

If an older OpenCode background service was already running during installation,
restart it once and print a browser link:

```sh
opencode service restart
opencode pair
```

Do not use `opencode upgrade` with this build; that command installs the official
binary and removes the patch. Rerun the installer above to update the patched
build instead.

## Publish a release

The tag names encode both the upstream OpenCode version and this patch revision:

```sh
git tag v2.2.6-web.1
git push origin main v2.2.6-web.1
```

The release workflow:

1. Clones the matching upstream tag (`v2.2.6`).
2. Applies `patches/web-gui-plugins.patch`.
3. Builds Linux glibc, musl, x64 baseline, x64 AVX2, and arm64 binaries.
4. Publishes checksummed release archives.

The Pages workflow publishes a rendered `/install` script that automatically
downloads the latest release from this repository.

## Updating OpenCode

1. Check whether the patch applies to the new upstream tag.
2. Update and runtime-test the patch if necessary.
3. Tag the repository using `v<upstream-version>-web.<revision>`.

## Local installer testing

The source installer contains a repository placeholder. Override it directly:

```sh
OPENCODE_WEB_REPOSITORY=OWNER/REPOSITORY bash install --help
```

## Limitations

- Release artifacts currently target Linux development servers.
- The web UI permits CSP `'unsafe-eval'` so trusted CommonJS GUI plugin bundles
  can run.
- Browser plugins run GUI `setup`; Electron `main`, native menus, and desktop IPC
  remain unavailable in web mode.
