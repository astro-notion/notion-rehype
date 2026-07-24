# Contributing and Releasing

This is the canonical contribution guide and maintainer release procedure for `@astro-notion/notion-rehype`.

## Prerequisites

- Node.js 24
- The pnpm version pinned in `package.json` through Corepack or `pnpm/action-setup`
- Git and a GitHub account
- For maintainers: permission to push release commits and tags to `main`, publish `@astro-notion/notion-rehype` on npm, and create GitHub Releases

Install dependencies and run the same checks as CI:

```sh
corepack enable
pnpm install --frozen-lockfile
pnpm exec prettier --check "src/**/*.ts"
pnpm typecheck
pnpm build
```

The repository does not currently have an automated test suite. Do not replace tests with a no-op script; add a real test runner and focused coverage when introducing tests.

## Contributor Workflow

1. Create a branch from the latest `main`.
2. Keep changes focused and verify behavior changes appropriately.
3. Run the checks above.
4. Open a pull request targeting `main`.

Contributors must not edit the package version or create release tags. Merging a pull request or pushing a branch never publishes to npm; only maintainers perform releases.

## Trusted Publisher Setup

Before the first tag-triggered release, configure the npm trusted publisher for `@astro-notion/notion-rehype` with these exact values:

| npm field            | Value           |
| -------------------- | --------------- |
| Organization or user | `astro-notion`  |
| Repository           | `notion-rehype` |
| Workflow filename    | `release.yml`   |
| Environment name     | `npm`           |
| Allowed action       | `npm publish`   |

The workflow filename is case-sensitive and must be entered without `.github/workflows/`. Do not add an `NPM_TOKEN`; the release workflow uses a GitHub-hosted runner and the `id-token: write` permission for npm OIDC trusted publishing.

References:

- [npm trusted publishing](https://docs.npmjs.com/trusted-publishers/)
- [npm provenance](https://docs.npmjs.com/generating-provenance-statements)

## Routine Maintainer Release

Start from an up-to-date, clean `main` and run the quality gates before changing the version:

```sh
git switch main
git pull --ff-only origin main
pnpm install --frozen-lockfile
pnpm exec prettier --check "src/**/*.ts"
pnpm typecheck
pnpm build
git status --short
```

Choose exactly one version command. `pnpm version` creates the release commit and matching annotated `v<version>` tag.

For a stable patch release:

```sh
pnpm version patch
```

Use `minor` or `major` instead of `patch` when appropriate. To promote a prerelease to a known stable version, pass it explicitly, for example `pnpm version 2.0.0`.

For the next beta prerelease:

```sh
pnpm version prerelease --preid beta
```

Use an explicit version such as `pnpm version 2.0.0-beta.0` when starting a new prerelease line. Inspect the generated commit and tag, then push `main` and that tag atomically:

```sh
VERSION=$(node -p "require('./package.json').version")
git show --stat "v${VERSION}"
git push --atomic origin main "v${VERSION}"
```

The tag starts `.github/workflows/release.yml`. The workflow rejects a tag that does not exactly equal `v<package.json version>` or whose commit is not contained in `origin/main`.

## Verify a Release

Wait for the `Release` workflow to finish, then verify the registry and GitHub state:

```sh
VERSION=$(node -p "require('./package.json').version")
npm view @astro-notion/notion-rehype dist-tags --json
npm view "@astro-notion/notion-rehype@${VERSION}" version
gh release view "v${VERSION}"
```

The npm package page must display provenance. Stable releases must update `latest`; prereleases must update `next` without changing `latest`. GitHub prereleases must be marked as prereleases and all GitHub Releases must contain generated notes.

## Recover a Failed Release

Determine whether npm published the version before changing tags or rerunning jobs:

```sh
VERSION=$(node -p "require('./package.json').version")
npm view "@astro-notion/notion-rehype@${VERSION}" version
```

### Validation or quality gate failed before publication

If npm does not contain the version, fix the release commit. An unpublished incorrect tag may be deleted and recreated on the corrected `main` commit:

```sh
FAILED_TAG=v1.2.1 # Replace with the tag named by the failed workflow.
git push origin ":refs/tags/${FAILED_TAG}"
git tag -d "${FAILED_TAG}"
# Fix and commit the issue, then recreate the tag.
EXPECTED_TAG="v$(node -p "require('./package.json').version")"
git tag -a "${EXPECTED_TAG}" -m "Release ${EXPECTED_TAG}"
git push --atomic origin main "${EXPECTED_TAG}"
```

Never move or reuse a tag after its package version has been published.

### npm publication failed

If npm does not contain the version, correct the trusted-publisher or registry issue and rerun the failed `Validate and publish` job. Confirm the npm publisher uses organization `astro-notion`, repository `notion-rehype`, workflow filename `release.yml`, and environment `npm` exactly.

### npm succeeded but GitHub Release creation failed

Do not rerun npm publication. Rerun only the failed `Create GitHub Release` job, or recover manually:

```sh
VERSION=$(node -p "require('./package.json').version")
gh release create "v${VERSION}" --generate-notes --verify-tag
```

Add `--prerelease` when recovering a prerelease.

### A published version is bad

npm versions are immutable. Do not delete and reuse the version or move its tag. Deprecate the bad version, restore the appropriate dist-tag if necessary, fix the problem, and publish a new version:

```sh
npm deprecate "@astro-notion/notion-rehype@<bad-version>" "Use <replacement-version> instead"
npm dist-tag add "@astro-notion/notion-rehype@<previous-stable-version>" latest
# For a bad prerelease, restore next instead:
npm dist-tag add "@astro-notion/notion-rehype@<previous-prerelease-version>" next
```
