<!--
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: 2025 The Linux Foundation
-->

# 🛠️ Build/install Node.js Project

Builds/installs a Node.js project.

Supports these build/install tools:

- NPM
- Yarn

## node-build-action

## Usage Example

<!-- markdownlint-disable MD046 -->

```yaml
  # Checkout code repository performed in earlier step
  - name: "Build/install Node.js project"
    uses: lfreleng-actions/node-build-action@main
```

<!-- markdownlint-enable MD046 -->

## Inputs

<!-- markdownlint-disable MD013 -->

| Variable Name | Required | Default    | Description                               |
| ------------- | -------- | ---------- | ----------------------------------------- |
| NODE_VERSION  | False    |            | Node.js version to use for build/install  |
| BUILD_TOOL    | False    |            | Tool used to perform the build [npm/yarn] |
| PATH_PREFIX   | False    |            | Path/directory to Node.js project code    |
| NPM_FLAGS     | False    | --omit=dev | Flags to pass to npm                      |
| YARN_FLAGS    | False    | --prod     | Flags to pass to yarn                     |
