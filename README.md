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
