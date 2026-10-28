# Stack-specific values

Detect each stack from its manifest. A repo can have several (a Go API beside a Node frontend): fill every
value for each, with each manifest's directory where a value takes a path. The release-please type is the one
exception: use the type of the manifest at the repo root, or `simple` when there is none.

| Stack | Manifest | `.gitignore` template | Dependabot ecosystem | release-please type |
| --- | --- | --- | --- | --- |
| Node / TypeScript | `package.json` | `Node` | `npm` (also covers pnpm, yarn) or `bun` with `bun.lock` | `node` |
| Python | `pyproject.toml` | `Python` | `uv` with `uv.lock`, else `pip` | `python` |
| Go | `go.mod` | `Go` | `gomod` | `go` |
| Rust | `Cargo.toml` | `Rust` | `cargo` | `rust` |
| Anything else, or no stack yet | — | none | none | `simple` |

When the repo has no `.gitignore` yet, fetch the stack's template with
`gh api gitignore/templates/<name> -q .source` and append it under a `# <name>` heading after the universal entries.

Add a Dependabot entry for every independent package root, with its directory: the repo root, each separate
stack (a Go API beside a Node frontend), and each Go module. Include ecosystems this table doesn't list
(`Dockerfile` → `docker`, `Gemfile` → `bundler`, `composer.json` → `composer`); security updates for an
unlisted one get a "Bump x from…" title, which fails `pr-title`. Members of a workspace (npm, pnpm, Yarn,
Bun, Cargo, or uv workspaces) are covered by the workspace root's entry; don't add their own. Each entry
mirrors the `github-actions` one in `dependabot.yml`: same weekly schedule, 7-day `cooldown`, `commit-message`
(`prefix: chore`, `include: scope`), and a group named after the ecosystem with `update-types: [minor, patch]`.

## Install cooldown

Local installs wait 7 days for new releases too, like Dependabot, so a compromised release is usually caught
before anyone installs it. Write the setting for the package manager the project uses; skip the others.

| Package manager | File | Setting |
| --- | --- | --- |
| pnpm | `pnpm-workspace.yaml` | `minimumReleaseAge: 10080` (minutes) |
| Bun | `bunfig.toml` | `[install]` with `minimumReleaseAge = 604800` (seconds) |
| Yarn | `.yarnrc.yml` | `npmMinimalAgeGate: 7d` |
| uv | `pyproject.toml` | `[tool.uv]` with `exclude-newer = "7 days"` |
| npm 11.10 or later | `.npmrc` | `min-release-age=7` (days); older npm ignores it |
| pip, Go, Cargo | none | Nothing to write |

## Toolchain version file

Pin the toolchain in a file so local machines and CI use the same version. Keep an existing pin when CI's
setup action can read it, and point the setup step at that file:

| Stack | Setup reads | Write when there's no readable pin |
| --- | --- | --- |
| Node + npm/pnpm/yarn | `.nvmrc`, `.node-version`, `.tool-versions`, or `package.json` (`volta`, `engines`) | `.node-version` with the major version, such as `24` |
| Node + Bun | `.bun-version`, `.tool-versions`, or `package.json` (`packageManager`) | `.bun-version` with the exact version, such as `1.4.4` |
| Python | `.python-version`, `.tool-versions`, or `pyproject.toml` (`requires-python`) | `.python-version` with the minor version, such as `3.13` (`uv python pin 3.13` writes it) |
| Go | `go.mod`'s `go` line | Nothing |
| Rust | `rust-toolchain.toml` | `rust-toolchain.toml`: `[toolchain]` with `channel = "<x.y.z>"` and `components = ["rustfmt", "clippy"]` |

A pin only in a file setup can't read, such as `mise.toml`, counts as no readable pin: write the file above
with the same version. Use the version installed locally (`node --version` and so on) when there's no pin at
all. List any file you write in the project tree.

## Check command

CI and `{{CI_COMMANDS}}` run the same checks, so a contributor or agent who runs them locally sees what CI
sees. In this order: format check, lint, typecheck, test, build.

- **One command when the project has a task runner.** Use an existing `check` script or task. A Node project
  with none gets one: a `"check"` script in `package.json` that runs the others in order (`npm run format:check
  && npm run lint && …`). `{{CI_COMMANDS}}` is then that one command, such as `` `npm run check` ``.
- **Otherwise, the commands in order**, joined with `, then `. Add no task runner the project doesn't use.
- **Run what exists; add no tool the project doesn't use.** The exception is the checks that ship with the
  toolchain, which every Go and Rust repo gets:

| Stack | Always-on checks |
| --- | --- |
| Go | `test -z "$(gofmt -l .)"`, `go vet ./...`, `go test ./...` |
| Rust | `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, `cargo test` |

## CI steps

Fill `{{CI_STEPS}}` with setup, install, then the check command. Setup reads the toolchain version file, so CI
never names a version itself.

Pin every action to its latest release's full commit SHA, with the tag as a comment
(`actions/setup-node@<sha> # v7.0.0`). Resolve the tag with
`gh api repos/<owner>/<repo>/releases/latest -q .tag_name`, then the SHA with
`gh api repos/<owner>/<repo>/commits/<tag> -q .sha`. Dependabot keeps the pins current. The zizmor job in the Workflows workflow flags unpinned actions, and the
SHA-pinned actions setting blocks them.

| Stack | Setup and cache | Install |
| --- | --- | --- |
| Node + npm/pnpm/yarn | `actions/setup-node` with `node-version-file: <the pin file>` and `cache: <package manager>`; pnpm also needs `pnpm/action-setup` before it, which reads the version from `packageManager` in `package.json`; add that field (`pnpm@<installed version>`) when it's missing | `npm ci`, `pnpm install --frozen-lockfile`, `yarn install --immutable` |
| Node + Bun | `oven-sh/setup-bun` with `bun-version-file: <the pin file>` | `bun install --frozen-lockfile` |
| Python + uv | `astral-sh/setup-uv` with `enable-cache: true` (it reads `.python-version`) | `uv sync --locked` |
| Python + pip | `actions/setup-python` with `python-version-file: <the pin file>` and `cache: pip` | `pip install -e .[dev]` or the documented equivalent |
| Go | `actions/setup-go` with `go-version-file: <dir>/go.mod` (caches by default) | `go mod download` |
| Rust | `run: rustup toolchain install` (rustup is preinstalled and reads `rust-toolchain.toml`), then `Swatinem/rust-cache` | none (`cargo` fetches) |

In a monorepo, `working-directory: <manifest directory>` only works on `run:` steps. Point setup actions at the
files through their inputs instead: `go-version-file: <dir>/go.mod`, `node-version-file: <dir>/.node-version`,
`cache-dependency-path: <dir>/<lockfile>`, or `workspaces: <dir>` for `rust-cache`.

Python + uv with releases: release-please bumps the version in `pyproject.toml`, and `uv sync --locked` then
fails until `uv.lock` matches. If the first release PR fails that way, run `uv lock` on its branch and push; the
Report says so.
