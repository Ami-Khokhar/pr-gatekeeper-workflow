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
