# 🛠️ VPP Build Action

This GitHub composite action builds the VPP (Vector Packet Processing) project. It supports both debug and release builds with configurable parallelism.

## Description

The action compiles VPP from source, optionally building a static `vppctl` binary, and then performs either a debug build or a release package verification depending on the specified build type.

## Usage

```yaml
- name: Build VPP
  uses: fdio/vpp/.github/actions/vpp-build@master
  with:
    MAKE_PARALLEL_JOBS: '16'
    BUILD_TYPE: 'release'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `MAKE_PARALLEL_JOBS` | Number of parallel jobs for `make` to control build parallelism | No | `""` (empty, uses make's default) |
| `BUILD_TYPE` | Type of build to perform: `debug` or `release` | No | `release` |
| `TUI_LINE` | Delimiter line for terminal UI output formatting | No | `*******************************************************************` |

## Outputs

This action does not produce explicit outputs, but it will:
- Build VPP binaries and packages
- Exit with code `0` on success
- Exit with code `1` on failure
- Display formatted build results in the workflow logs

## Build Behavior

### Release Build (default)
- Runs `make UNATTENDED=yes MAKE_PARALLEL_JOBS=<value> pkg-verify`
- Builds and verifies VPP packages
- Suitable for production releases and package validation

### Debug Build
- Runs `make UNATTENDED=yes MAKE_PARALLEL_JOBS=<value> build`
- Builds VPP with debug symbols
- Suitable for development and debugging

### Static vppctl
If `extras/scripts/build_static_vppctl.sh` exists, it will be executed before the main build to create a statically-linked vppctl binary.

## Prerequisites

Before using this action, ensure the following actions have been run:
1. **Checkout**: Code must be checked out
2. **Setup Environment**: Environment variables must be configured (particularly `OS_ID`, `OS_VERSION_ID`, `OS_ARCH`)
3. **Install Dependencies**: VPP dependencies must be installed via `vpp-install-deps` action
4. **Install External Dependencies**: External dependencies must be installed via `vpp-install-ext-deps` action

## Example Workflow

```yaml
jobs:
  build-vpp:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        build-type: ['debug', 'release']

    steps:
      - name: Checkout
        uses: actions/checkout@v5

      - name: Setup Environment
        uses: fdio/.github/.github/actions/setup-executor-env@main

      - name: Install VPP Dependencies
        uses: fdio/vpp/.github/actions/vpp-install-deps@master

      - name: Install VPP External Dependencies
        uses: fdio/vpp/.github/actions/vpp-install-ext-deps@master

      - name: Build VPP
        uses: fdio/vpp/.github/actions/vpp-build@master
        with:
          MAKE_PARALLEL_JOBS: '16'
          BUILD_TYPE: ${{ matrix.build-type }}
```

## Error Handling

The action implements comprehensive error handling:
- Exits immediately on any command failure (`set -euo pipefail`)
- Captures specific error messages for each build stage
- Displays clear error context in the failure message
- Returns appropriate exit codes

## Environment Variables Used

The following environment variables are expected to be set by upstream actions:
- `OS_ID`: Operating system identifier (e.g., `ubuntu`, `debian`)
- `OS_VERSION_ID`: OS version identifier (e.g., `22.04`, `24.04`)
- `OS_ARCH`: CPU architecture (e.g., `x86_64`, `aarch64`)

## Notes

- The action uses `UNATTENDED=yes` to prevent interactive prompts during the build
- Build output includes visual separators for improved log readability
- Parallel job count can be left empty to use make's default parallelism
- The action supports both Makefile targets: `build` (debug) and `pkg-verify` (release)

## Troubleshooting

### Build Fails with Missing Dependencies
Ensure `vpp-install-deps` and `vpp-install-ext-deps` actions have run successfully before this action.

### Build Fails with Missing OS Variables
Verify that `setup-executor-env` action has been executed to set `OS_ID`, `OS_VERSION_ID`, and `OS_ARCH`.

### Out of Memory Errors
Reduce the `MAKE_PARALLEL_JOBS` value to decrease memory consumption during parallel compilation.

## License

This action is part of the VPP project. See the main repository LICENSE file for details.
