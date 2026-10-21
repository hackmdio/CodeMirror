# Releasing `@hackmd/codemirror`

Versions are published to npm by the
[`Publish Package`](.github/workflows/publish.yml) workflow using
[npm trusted publishing](https://docs.npmjs.com/trusted-publishers) (OIDC).
No npm token is involved, and every version is published with
[provenance](https://docs.npmjs.com/generating-provenance-statements).
Don't run `npm publish` locally.

## Steps

1. On `master`, bump `version` in `package.json` and commit it, for example
   with `npm version patch --no-git-tag-version`. Wait for CI to pass.
2. Tag that commit with the bare version (no `v` prefix) and push the tag:

   ```sh
   git tag 5.65.12
   git push origin 5.65.12
   ```

3. The tag push starts the workflow. When it finishes, check that the new
   version on npmjs.com shows a provenance badge.

If a run fails for a transient reason, re-run it from the Actions tab.

## What the workflow checks

- The tag must equal the `version` in `package.json`.
- That version must not already be on npm.
- `npm install`, `npm run build` and `npm test` must pass. Tests run in
  puppeteer's bundled Chromium.
- `npm publish` runs `prepublishOnly`, which rebuilds and minifies
  `codemirror.min.js` via `./release`. If that fails, nothing is uploaded.
- Versions with a pre-release suffix (`-beta.0` etc.) go to the `next`
  dist-tag; all others go to `latest`.

## Dry runs

From the Actions tab, choose **Publish Package > Run workflow** on any branch
or tag. **dry-run** is ticked by default: the workflow runs every check and
calls `npm publish --dry-run`, or `npm pack --dry-run` if the version is
already on npm. Unticking it only publishes when run from a version tag.

## Trusted publisher settings on npmjs.com

Under the package's **Settings > Trusted Publisher**, choose GitHub Actions
and enter:

| Field                | Value         |
| -------------------- | ------------- |
| Organization or user | `hackmdio`    |
| Repository           | `CodeMirror`  |
| Workflow filename    | `publish.yml` |
| Environment name     | (leave empty) |

After the first successful publish, consider setting **Settings > Publishing
access** to "Require two-factor authentication and disallow tokens".
