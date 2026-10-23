<!--
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: 2025 The Linux Foundation
-->

# 🛠️ Build/install Node.js Project

Builds/installs a Node.js project, including gathering build metadata,
installing dependencies, and executing build scripts defined in package.json.

Supports these build/install tools:

- NPM (`npm`)
- Yarn classic, 1.x (`yarn`)
- Yarn Berry, 2 and later (`yarn-berry`)
- pnpm (`pnpm`)
- Bun (`bun`)

Any other `build_tool` value fails the action.

## node-build-action

This action automatically:

- Validates the presence of package.json
- Installs project dependencies using the selected tool
- Executes specified build scripts from package.json with that tool

## Usage Examples

<!-- markdownlint-disable MD046 -->

```yaml
  # Checkout code repository performed in earlier step
  - name: "Build/install Node.js project"
    uses: lfreleng-actions/node-build-action@main
    with:
      scripts: 'build'  # Execute the 'build' script from package.json
```

```yaml
  - name: "Build/install Node.js project with tests"
    uses: lfreleng-actions/node-build-action@main
    with:
      scripts: 'test,build,lint'  # Comma-separated format
```

```yaml
  - name: "Build/install Node.js project with tests (space-separated)"
    uses: lfreleng-actions/node-build-action@main
    with:
      scripts: 'test build lint'  # Space-separated format
```

```yaml
  - name: "Build/install Node.js project with tests (multi-line)"
    uses: lfreleng-actions/node-build-action@main
    with:
      scripts: |  # Multi-line format
        test
        build
        lint
```

### Script Names

Script names may contain letters, digits and the characters `_ - . : / @`,
which covers names such as `lint:ci`, `test:unit` and `build.prod`. The
action rejects a name containing any other character, such as a shell
metacharacter. A rejected name fails the action, as a missing script does,
unless `exit_on_fail` is `false`.

### Use Yarn Instead of NPM

```yaml
  - name: "Build/install Node.js project with Yarn"
    uses: lfreleng-actions/node-build-action@main
    with:
      build_tool: 'yarn'
      scripts: 'build'
```

### Use Yarn Berry, pnpm or Bun

```yaml
  - name: "Build/install Node.js project with pnpm"
    uses: lfreleng-actions/node-build-action@main
    with:
      build_tool: 'pnpm'  # or 'yarn-berry', or 'bun'
      scripts: 'build test'
```

Yarn Berry and pnpm run through a pinned release of
[Corepack](https://github.com/nodejs/corepack), which the action installs
because Node.js 25 and later no longer bundle it. Corepack runs the release
that the `packageManager` field of `package.json` pins:

- `yarn-berry` requires that pin, and fails unless it names Yarn 2 or later
  (for example `"packageManager": "yarn@4.9.1"`). Without it, Corepack would
  run Yarn classic.
- `pnpm` fails when `packageManager` names another tool. Without a pin,
  Corepack runs its own default pnpm release.

The Corepack release needs Node.js `^22.22.2`, `^24.15.0` or `>=26.0.0`.
The action puts Corepack's `yarn` and `pnpm` executables first on the
`PATH` of its own install and script steps alone; it leaves the `PATH` of
later steps in the job unchanged. Corepack never edits `package.json`.

`bun` installs the exact Bun release named by `bun_version` with
[`oven-sh/setup-bun`](https://github.com/oven-sh/setup-bun), with its cache
disabled.

Under harden-runner `block` egress, these tools reach `registry.npmjs.org`
(Corepack, pnpm, Bun installs), `repo.yarnpkg.com` (Yarn Berry), and
`github.com` with `release-assets.githubusercontent.com` (Bun). The
`lfreleng-actions` allow-list includes each of them.

### Continue on Missing Scripts (Warning Mode)

```yaml
  - name: "Build/install Node.js project (continue on error)"
    uses: lfreleng-actions/node-build-action@main
    with:
      scripts: 'lint,build,optional-script'
      exit_on_fail: 'false'  # Show warnings, don't fail on missing scripts
```

### Lockfile Enforcement

The `install_mode` input controls how the action treats the project's
lockfile when installing dependencies:

<!-- markdownlint-disable MD013 -->

| Value            | Behaviour                                                                  |
| ---------------- | -------------------------------------------------------------------------- |
| `auto` (default) | Frozen install when a lockfile exists, otherwise a resolving install       |
| `frozen`         | Frozen install; fails when the lockfile is missing or out of date          |
| `install`        | Resolving install, which may update the lockfile (the previous behaviour)  |

<!-- markdownlint-enable MD013 -->

A frozen install runs `npm ci` for npm (lockfile `package-lock.json` or
`npm-shrinkwrap.json`), `yarn install --frozen-lockfile` for Yarn classic
and `yarn install --immutable` for Yarn Berry (`yarn.lock`),
`pnpm install --frozen-lockfile` for pnpm (`pnpm-lock.yaml`), and
`bun install --frozen-lockfile` for Bun (`bun.lock` or `bun.lockb`). It
installs the exact dependency tree recorded in the lockfile, so a build
cannot drift from the tree that audits and scans saw. A resolving install
passes `--no-immutable` to Yarn Berry and `--no-frozen-lockfile` to pnpm,
which otherwise install frozen whenever the runner defines `CI`.

Passing `--no-package-lock` in `npm_flags`, `--no-lockfile` in `yarn_flags`
(Yarn classic) or `--no-lockfile` in `pnpm_flags` tells the tool to ignore
its lockfile: `auto` then performs a resolving install, and `frozen` fails.
Yarn Berry and Bun have no such flag.

```yaml
  - name: "Build/install Node.js project (lockfile required)"
    uses: lfreleng-actions/node-build-action@main
    with:
      install_mode: 'frozen'  # Fail without an up-to-date lockfile
```

**Behaviour change:** before `install_mode` existed, the action always
performed a resolving install. With the `auto` default, a project that
commits a lockfile now gets a frozen install, which fails when the lockfile is
out of date with `package.json`. Update the lockfile, or set
`install_mode: 'install'` to keep the old behaviour.

<!-- markdownlint-enable MD046 -->

## Security Considerations

For security, the action validates `npm_flags`, `yarn_flags`, `pnpm_flags`
and `bun_flags` inputs against an allowlist of safe flags for each tool. This
prevents command injection attacks and other security vulnerabilities.

The action also disables the dependency cache that `actions/setup-node`
enables by default when `package.json` declares npm as its `packageManager`,
and the Bun download cache of `oven-sh/setup-bun`. A cache written by a
less-trusted run could otherwise feed a later privileged build (cache
poisoning).

### Allowed NPM Flags

- `--no-save` - Don't save to package.json
- `--no-package-lock` - Don't generate package-lock.json
- `--ignore-scripts` - Don't run lifecycle scripts
- `--no-audit` - Skip audit
- `--no-fund` - Skip funding messages
- `--legacy-peer-deps` - Use legacy peer dependency resolution
- `--force` - Force installation
- `--verbose` - Verbose output
- `--quiet` - Quiet output
- `--loglevel=*` - Set log level
- `--production` - Install production dependencies
- `--omit=dev` - Omit dev dependencies
- `--omit=peer` - Omit peer dependencies
- `--omit=optional` - Omit optional dependencies
- `--include=dev` - Include dev dependencies
- `--include=peer` - Include peer dependencies
- `--include=optional` - Include optional dependencies
- `--prefer-offline` - Prefer offline cache
- `--prefer-online` - Prefer online registry

### Allowed Yarn Flags

For Yarn classic (`build_tool: yarn`):

- `--production` - Install production dependencies
- `--frozen-lockfile` - Don't update lockfile
- `--pure-lockfile` - Don't update lockfile (alias)
- `--no-lockfile` - Don't read or generate lockfile
- `--ignore-scripts` - Don't run lifecycle scripts
- `--ignore-optional` - Don't install optional dependencies
- `--force` - Force installation
- `--verbose` - Verbose output
- `--silent` - Silent output
- `--no-progress` - Disable progress bar
- `--network-timeout=*` - Set network timeout
- `--prefer-offline` - Prefer offline cache
- `--offline` - Use offline mode

### Allowed Yarn Berry Flags

For Yarn Berry (`build_tool: yarn-berry`), `yarn_flags` accepts Berry's
flags; `install_mode` chooses `--immutable`:

- `--immutable-cache` - Fail if the cache folder would change
- `--check-cache` - Refetch packages and verify their checksums
- `--check-resolutions` - Verify that package resolutions are coherent
- `--inline-builds` - Print dependency build output
- `--json` - Output an NDJSON stream
- `--mode=skip-build` - Don't run dependency build scripts

### Allowed pnpm Flags

`install_mode` chooses `--frozen-lockfile`.

- `--prod` - Don't install devDependencies
- `--dev` - Install devDependencies alone
- `--no-optional` - Don't install optionalDependencies
- `--no-lockfile` - Don't read or generate pnpm-lock.yaml
- `--ignore-scripts` - Don't run lifecycle scripts
- `--prefer-offline` - Prefer cached metadata
- `--offline` - Use offline mode
- `--strict-peer-dependencies` - Fail on missing or invalid peers
- `--silent` - Silent output
- `--reporter=REPORTER` - `default`, `ndjson` or `silent`
- `--loglevel=LEVEL` - `debug`, `info`, `warn` or `error`

### Allowed Bun Flags

`install_mode` chooses `--frozen-lockfile`.

- `--production` - Don't install devDependencies
- `--ignore-scripts` - Don't run the project's lifecycle scripts
- `--omit=TYPE` - Omit `dev`, `optional` or `peer` dependencies
- `--linker=LINKER` - `isolated` or `hoisted`
- `--verbose` - Verbose output
- `--silent` - Silent output
- `--no-progress` - Disable progress bar

Yarn Berry, pnpm and Bun take a flag's value in the `--flag=value` form,
not as a separate word.

The action blocks flags containing special characters (`;`, `&`, `|`, `` ` ``,
`(`, `)`, `<`, `>`) and command substitution patterns (such as `$(` and `${`)
to prevent injection attacks.

## Inputs

<!-- markdownlint-disable MD013 -->

| Variable Name | Required | Default | Description                                    |
| ------------- | -------- | ------- | ---------------------------------------------- |
| node_version  | False    | 22      | Node.js version for build/install              |
| build_tool    | False    | npm     | Build tool [npm/yarn/yarn-berry/pnpm/bun]      |
| path_prefix   | False    | .       | Path to Node.js project code                   |
| npm_flags     | False    |         | Flags for npm (allowlist enforced)             |
| yarn_flags    | False    |         | Flags for yarn (allowlist enforced)            |
| pnpm_flags    | False    |         | Flags for pnpm (allowlist enforced)            |
| bun_flags     | False    |         | Flags for bun (allowlist enforced)             |
| bun_version   | False    | 1.4.3   | Exact Bun release for `bun` (X.Y.Z)            |
| install_mode  | False    | auto    | Lockfile handling [auto/frozen/install]        |
| scripts       | False    | build   | Scripts to run (comma/space/newline-separated) |
| exit_on_fail  | False    | true    | Exit on missing/rejected script (true/false)   |

<!-- markdownlint-enable MD013 -->

**Migration Note:** Previous versions of this action had default values for
`npm_flags` (`--omit=dev`) and `yarn_flags` (`--prod`). This version removes
these defaults. If you need the previous behavior, explicitly set these inputs
in your workflow.
