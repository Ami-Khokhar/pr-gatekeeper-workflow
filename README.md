# pr-gatekeeper-workflow

Public caller for the private [pr-gatekeeper](https://github.com/Ami-Khokhar/pr-gatekeeper)
reusable workflow.

GitHub does not allow a **public** repository to call a reusable workflow stored in a
**private** repository. Target repositories that are public call this thin wrapper instead;
it checks out the private implementation at `gatekeeper_ref` using the
`GATEKEEPER_READ_TOKEN` secret. Only the workflow orchestration lives here — the review
logic and prompts stay private.

Callers reference it as:

```yaml
uses: Ami-Khokhar/pr-gatekeeper-workflow/.github/workflows/gatekeeper.yml@v1
```

## Release channel

`gatekeeper-version.json` in this repository holds the gate revision that every
caller following `v1` runs. The workflow reads it at run time (job `pr_body`
fetches it, validates `schema` and the commit sha, and passes the commit to
`gates` and `review`). A release is one edit to that file. A caller that needs a
stable gate passes a full commit sha in `gatekeeper_ref` instead of `v1`.

The file is the release trust anchor: it is public, and changing it changes the
gate for every caller on `v1`. Put the new sha here only after the gate repository
passes its own verify command and its interface metadata compares compatible with
the previous release.

### Cutting a release

1. Edit `gatekeeper-version.json`: `gate_revision`, `released_at`, and a `note`
   only when it says something new. Merge it to `main`.
2. Move the tag: `git tag -f v1 <main sha> && git push --force origin refs/tags/v1`.
   **Both steps are required.** Callers fetch the workflow *and* the version file
   through `v1`; editing the file alone changes nothing, because the tag still
   points at the commit that holds the old copy.
3. Roll back the same way in one command: `git tag -f v1 <previous sha> && git
   push --force origin refs/tags/v1`. Do that before debugging anything, so no
   caller runs a gate that cannot review.

`raw.githubusercontent.com` serves the version file through a cache, and right
after a tag move it can still return the previous revision for a couple of
minutes. A caller that must not see a half-released gate passes a full commit
sha in `gatekeeper_ref`, which skips the file entirely.
