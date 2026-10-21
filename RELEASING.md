# Releasing `@hackmd/codemirror`

## Bump Version

In the Actions tab, run **Bump Version** on `master`. Pick `patch`, `minor`,
`major` or `prerelease` (optionally with a `preid` such as `beta`), or enter
an explicit `version`. Tick **dry-run** to only bump, build and test.

The workflow tests, commits `chore: bump version to X [skip ci]`, pushes a
bare tag `X` (e.g. `5.65.12`) and starts **Publish Package** on that tag,
which publishes to npm with provenance via trusted publishing.

If you push a tag by hand onto a `[skip ci]` commit and no publish run
starts, run **Publish Package** from that tag with **dry-run** unticked.

## npm Trusted Publisher (npmjs.com > package Settings > Trusted Publisher)

| Field                | Value              |
| -------------------- | ------------------ |
| Organization or user | `hackmdio`         |
| Repository           | `CodeMirror`       |
| Workflow filename    | `publish.yml`      |
| Environment name     | (leave empty)      |
| Allowed actions      | tick `npm publish` |
