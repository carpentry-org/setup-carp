# setup-carp

A composite GitHub Action that installs the [Carp](https://github.com/carp-lang/Carp) compiler and prepares a CI environment for Carp libraries.

## What it does

- Uses the `stack` pre-installed on GitHub-hosted runners. Stack downloads the GHC version Carp's `stack.yaml` pins into `~/.stack/programs/` on first run.
- Optionally installs SDL2 system packages (for tests that use the SDL bindings).
- Clones `carp-lang/Carp` at the ref you pick and builds it with `stack install`.
- Puts the `carp` binary on `PATH` and exports `CARP_DIR` so the compiler can find its `core/` sources.
- Caches `~/.stack` and `.stack-work` keyed on the ref and `stack.yaml.lock`, so subsequent runs reuse the build.

Runs on `ubuntu-latest` and `macos-latest`. On custom or self-hosted runners that don't ship with `stack`, prepend [`haskell-actions/setup@v2`](https://github.com/haskell-actions/setup) yourself.

## Usage

Minimal library CI:

```yaml
name: CI
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: carpentry-org/setup-carp@v1
      - run: carp -x test/all.carp
```

Pinning a specific Carp version and enabling SDL:

```yaml
      - uses: carpentry-org/setup-carp@v1
        with:
          carp-ref: v0.6.0
          sdl: true
```

Matrix across OSes:

```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - uses: carpentry-org/setup-carp@v1
      - run: carp -x test/all.carp
```

## Inputs

| Input             | Default           | Description                                                                 |
|-------------------|-------------------|-----------------------------------------------------------------------------|
| `carp-ref`        | `master`          | Git ref (branch, tag, or SHA) of the Carp compiler to build.                |
| `carp-repository` | `carp-lang/Carp`  | Source repository for Carp.                                                 |
| `sdl`             | `false`           | If `true`, install SDL2 system packages.                                    |
| `install-path`    | `.carp-src`       | Workspace-relative path where Carp is cloned and built.                     |
| `rewrite-git-urls`| `true`            | Rewrite `git@github.com:` to `https://github.com/` so `(load "git@...")` works without an SSH key. Set to `false` if you have your own SSH auth configured. |

## Outputs

| Output         | Description                                                              |
|----------------|--------------------------------------------------------------------------|
| `carp-dir`     | Absolute path of the Carp source directory (same as `CARP_DIR`).         |
| `carp-sha`     | Resolved commit SHA of the built compiler.                               |
| `carp-version` | Human-readable version string via `git describe --tags --always` (e.g. `v0.6.0`, `v0.6.0-3-gabcdef`). |

## Notes

- The first cold build takes ~5–10 minutes (Haskell). Cached runs are typically under a minute.
- For performance comparisons in CI, run Carp with `--optimize` so the generated C is built at `-O2` rather than `-O0`.
- If your library depends on system libraries beyond SDL2, install them in your own step before invoking `carp`.

## License

Apache 2.0.

<hr/>

Have fun!
