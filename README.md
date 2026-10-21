# control-repo

Central, reusable GitHub Actions workflow for the `@bugseeai` helper. Each Bugsee repo
has a thin caller (`.github/workflows/bugseeai.yml`) that calls
[`.github/workflows/bugseeai.yml`](.github/workflows/bugseeai.yml) here.

## Calling it

```yaml
jobs:
  bugseeai:
    uses: bugsee/control-repo/.github/workflows/bugseeai.yml@<full commit SHA> # <tag>
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
      BUGSEEAI_TOKEN: ${{ secrets.BUGSEEAI_TOKEN }}
```

Pin to a commit SHA, not `@main`: the callers hand this workflow the org's secrets.
Don't use `secrets: inherit`.

## Inputs

| Input | Default | Meaning |
| --- | --- | --- |
| `allowed_tools` | `Bash(npm ci),Bash(npm test:*),Bash(npm run lint:*)` | Shell commands Claude may run |
| `trusted_authors_only` | `true` | Respond only on issues/PRs opened by owners, members or collaborators |

## Security model

The job holds `BUGSEEAI_TOKEN` (write access) and `CLAUDE_CODE_OAUTH_TOKEN` and may run the
repository's own code, so it must never run on untrusted input:

- Only owners, members and collaborators can trigger it.
- It refuses pull requests whose head is in a fork.
- With `trusted_authors_only`, the issue/PR must also be opened by a trusted author.
- Claude only sees comments from org members.

Because of `trusted_authors_only`, bot-authored PRs (e.g. Dependabot) are ignored.
- Third-party actions are pinned to commit SHAs.
