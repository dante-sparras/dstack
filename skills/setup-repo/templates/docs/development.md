# Development

## Prerequisites

{{PREREQUISITES}}

## First checkout

```sh
git clone https://github.com/{{OWNER}}/{{REPO}}.git
cd {{REPO}}
{{SETUP_COMMANDS}}
```

{{ENVIRONMENT_NOTES}}

## Commands

{{COMMANDS_TABLE}}

Before opening a PR, run the checks CI runs: {{CI_COMMANDS}}.
<!-- optional:releases -->

## Releasing

release-please keeps a release PR open that collects the changes merged into `{{DEFAULT_BRANCH}}`, with the next version and its changelog. Merging that PR tags the release and publishes it.

- **Pick the version yourself:** when merging a PR, add `Release-As: x.y.z` to the end of the squash commit message.
- **The release PR's checks don't start:** without the `RELEASE_PLEASE_TOKEN` secret, close and reopen the release PR right before merging it.
<!-- /optional:releases -->
