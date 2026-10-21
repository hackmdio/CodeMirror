# Releasing `@hackmd/codemirror`

Versions are published to npm by the
[`Publish Package`](.github/workflows/publish.yml) workflow using
[npm trusted publishing](https://docs.npmjs.com/trusted-publishers) (OIDC).
No npm token is involved, and every version is published with
[provenance](https://docs.npmjs.com/generating-provenance-statements).
Don't run `npm publish` locally.

## Release with the Bump Version workflow (recommended)

1. Make sure CI is green on `master`.
2. In the Actions tab, choose **Bump Version > Run workflow** on `master` and
   pick `patch`, `minor`, `major` or `prerelease`, or type an explicit
   `version` (it overrides the bump). For pre-releases, `preid` sets the
   identifier, e.g. `beta` gives `5.65.12-beta.0`.
3. The workflow:
   1. bumps `package.json` (there is no lockfile) and checks that the new
      version is higher than the current one, has no tag yet and isn't on npm;
   2. runs `npm install`, `npm run build` and `npm test`;
   3. commits `chore: bump version to <version> [skip ci]` to `master`, tags
      it with the bare version (e.g. `5.65.12`, no `v`) and pushes both in one
      atomic push, so if `master` moved meanwhile, neither lands;
   4. starts **Publish Package** on the new tag with `workflow_dispatch`.
4. Follow the **Publish Package** run. When it finishes, check that the new
   version on npmjs.com shows a provenance badge.

Tick **dry-run** to bump, build and test without pushing, tagging or
publishing.

### Why the bump workflow dispatches the publish

- Pushes made with the workflow's `GITHUB_TOKEN` don't start other
  workflows, so the tag push alone wouldn't run **Publish Package**.
  `workflow_dispatch` is the documented exception.
- `[skip ci]` only suppresses `push` and `pull_request` runs, so it stops CI
  on the bump commit but not the dispatched publish run.
- **Publish Package** runs as its own top-level workflow, not through
  `workflow_call`. With `workflow_call`, npm checks the trusted publisher
  filename against the calling workflow (`bump-version.yml`) rather than
  `publish.yml`.

## Release by hand

1. On `master`, bump `version` in `package.json` and commit it, for example
   with `npm version patch --no-git-tag-version`. Don't put `[skip ci]` in the
   message. Wait for CI to pass.
2. Tag that commit with the bare version and push the tag:

   ```sh
   git tag 5.65.12
   git push origin 5.65.12
   ```

3. The tag push starts **Publish Package**.

The tag-push trigger only matches plain `X.Y.Z` tags. To publish a
pre-release tag such as `5.65.12-beta.0`, run **Publish Package** from that
tag (see "Manual publish runs" below).

GitHub may skip the tag-push run when the tagged commit's message contains
`[skip ci]`, for example a bot bump commit whose tag you push by hand. If a
tag push doesn't start a run, publish from the tag manually as described below.

## What Publish Package checks

- The tag must equal the `version` in `package.json`.
- That version must not already be on npm.
- `npm install`, `npm run build` and `npm test` must pass. Tests run in
  puppeteer's bundled Chromium.
- `npm publish` runs `prepublishOnly`, which rebuilds and minifies
  `codemirror.min.js` via `./release`. If that fails, nothing is uploaded.
- Versions with a pre-release suffix (`-beta.0` etc.) go to the `next`
  dist-tag; all others go to `latest`.

## Manual publish runs

From the Actions tab, choose **Publish Package > Run workflow**:

- With **dry-run** ticked (the default), any branch or tag works. The workflow
  runs every check and calls `npm publish --dry-run`, or `npm pack --dry-run`
  if the version is already on npm.
- With **dry-run** unticked, it only publishes when started from a version
  tag. Use this to retry a failed publish or to publish a tag whose push
  didn't start a run.

## GitHub repository settings

The bump workflow pushes to `master` and dispatches workflows as
`github-actions[bot]` with `GITHUB_TOKEN` (`contents: write`,
`actions: write`, requested in the workflow file). Keep these settings
compatible:

- **Branch protection or rulesets on `master`:** right now there's no classic
  protection and no required checks. The only ruleset is the org-level
  automatic Copilot code review, which doesn't block pushes. If you later
  require pull requests or status checks, the bot push will be rejected and
  nothing gets tagged or published. In that case, create a GitHub App, add it
  as a bypass actor for the rule, and have the workflow push with that app's
  token (for example via `actions/create-github-app-token`) instead of
  `GITHUB_TOKEN`. A push made with an app token does start other workflows,
  so the tag push would then also trigger **Publish Package**. That is safe:
  the second run fails the "already on npm" check, or GitHub skips it because
  of `[skip ci]`.
- **Tag rulesets:** if you add one for version tags, allow GitHub Actions to
  create them.
- **Settings > Actions > General > Workflow permissions:** the workflow
  requests its own permissions, but an organization policy that forbids
  write access for `GITHUB_TOKEN` would block the push.

## Trusted publisher settings on npmjs.com

Under the package's **Settings > Trusted Publisher**, choose GitHub Actions
and enter:

| Field                | Value                         |
| -------------------- | ----------------------------- |
| Organization or user | `hackmdio`                    |
| Repository           | `CodeMirror`                  |
| Workflow filename    | `publish.yml`                 |
| Environment name     | (leave empty)                 |
| Allowed actions      | tick **`npm publish`**        |

Register only `publish.yml`, not `bump-version.yml`: every publish, whether
from a tag push or a dispatch, runs as `publish.yml`. New trusted publisher
configurations only allow `npm stage publish` by default, so `npm publish`
must be ticked. A new configuration also expires if it isn't used for a
successful publish within 2 days, so register it shortly before the first
release.

After the first successful publish, consider setting **Settings > Publishing
access** to "Require two-factor authentication and disallow tokens".
