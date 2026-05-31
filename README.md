# Mise Elixir Cache

A GitHub Action that caches Elixir/Mix dependencies and compilation artifacts for projects using [Mise](https://mise.jdx.dev/) as a version manager.

## Features

- **Dependency & build caching** — caches `deps/` and environment-scoped `_build` compilation artifacts with keys segmented by OS, `MIX_ENV`, Erlang version, Elixir version, `mise.toml`, and `mix.lock`
- **Hex & Mix home caching** — caches `~/.hex` and `~/.mix` to skip Hex registry fetches and archive downloads
- **Auto-clean on retry** — when a workflow run is retried (`run_attempt > 1`), dependencies and build artifacts are cleaned before fetching
- **Dependency fetching** — runs `mix deps.get` with optional extra arguments

## Usage

```yaml
steps:
  - uses: actions/checkout@v4

  - name: Install Elixir via Mise
    uses: jdx/mise-action@v2

  - uses: bazziletech/mise-elixir-cache@main
    with:
      mix-env: test

  - run: mix test
```

### With custom `deps.get` arguments

```yaml
  - uses: bazziletech/mise-elixir-cache@main
    with:
      mix-env: prod
      deps-get-args: "--only prod"
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `mix-env` | Yes | — | `MIX_ENV` value used for cache key segmentation |
| `deps-get-args` | No | `""` | Additional arguments passed to `mix deps.get` |

## How it works

1. **Detect tool versions** — reads the active Erlang and Elixir versions from Mise for cache key segmentation.
2. **Restore deps & build cache** — restores `deps/`, `_build/<mix-env>/lib`, and `_build/<mix-env>/phoenix-colocated` using a composite key of `runner.os`, `mix-env`, Erlang version, Elixir version, `mise.toml` hash, and `mix.lock` hash. Falls back only within the same Erlang and Elixir toolchain when `mix.lock` has changed.
3. **Restore Hex/Mix home cache** — restores `~/.hex` and `~/.mix` so Hex and Rebar are available without network requests. This runs before the clean step since `mix deps.clean` needs Hex to parse `mix.exs`.
4. **Clean on retry** — if `github.run_attempt > 1`, forces Hex/Rebar installation and runs `mix deps.clean --all` and `mix clean` to clear potentially stale artifacts.
5. **Fetch dependencies** — runs `mix deps.get` (with any extra args). This is fast (~1s) on cache hits and installs Hex/Rebar if not already cached.

## License

MIT
