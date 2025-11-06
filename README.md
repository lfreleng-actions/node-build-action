<!--
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: 2025 The Linux Foundation
-->

# 🛠️ Build/install Node.js Project

Builds/installs a Node.js project, including gathering build metadata,
installing dependencies, and executing build scripts defined in package.json.

Supports these build/install tools:

- NPM
- Yarn

## node-build-action

This action automatically:

- Gathers comprehensive build metadata about your Node.js project
- Validates the presence of package.json
- Installs project dependencies using npm or yarn
- Executes specified build scripts from package.json

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

### Use Yarn Instead of NPM

```yaml
  - name: "Build/install Node.js project with Yarn"
    uses: lfreleng-actions/node-build-action@main
    with:
      build_tool: 'yarn'
      scripts: 'build'
```

### Continue on Missing Scripts (Warning Mode)

```yaml
  - name: "Build/install Node.js project (continue on error)"
    uses: lfreleng-actions/node-build-action@main
    with:
      scripts: 'lint,build,optional-script'
      exit_on_fail: 'false'  # Show warnings, don't fail on missing scripts
```

<!-- markdownlint-enable MD046 -->

## Security Considerations

For security, the action validates `npm_flags` and `yarn_flags` inputs
against an allowlist of safe flags. This prevents command injection attacks
and other security vulnerabilities.

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
- `--include=dev` - Include dev dependencies
- `--include=peer` - Include peer dependencies
- `--prefer-offline` - Prefer offline cache
- `--prefer-online` - Prefer online registry

### Allowed Yarn Flags

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

The action blocks flags containing special characters (`;`, `&`, `|`, `` ` ``,
`$`, `(`, `)`, `<`, `>`) or command substitution patterns to prevent injection
attacks.

## Inputs

<!-- markdownlint-disable MD013 -->

| Variable Name | Required | Default    | Description                      |
| ------------- | -------- | ---------- | -------------------------------- |
| node_version  | False    | 22         | Node.js version for build/install |
| build_tool    | False    | npm        | Build tool [npm/yarn]            |
| path_prefix   | False    |            | Path to Node.js project code     |
| npm_flags     | False    |            | Flags for npm (allowlist enforced) |
| yarn_flags    | False    |            | Flags for yarn (allowlist enforced) |
| scripts       | False    | build      | Scripts to run (comma/space/newline-separated) |
| exit_on_fail  | False    | true       | Exit on missing script (true/false) |

**Migration Note:** Previous versions of this action had default values for
`npm_flags` (`--omit=dev`) and `yarn_flags` (`--prod`). This version removes
these defaults. If you need the previous behavior, explicitly set these inputs
in your workflow.
