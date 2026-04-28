# Build red

With all [dependencies](index.md#dependencies) in place, the build is a single script.

## Clone

```bash
git clone --recursive https://github.com/moments-behavior/red.git
cd red
```

The `--recursive` flag fetches the submodules under `lib/`.

## Build

```bash
./build.sh
```

This runs CMake into a `release/` directory and produces the GUI executable:

- **`release/red`** — the labeling app

## Run

```bash
./run.sh
```

## Optional: install a desktop launcher

```bash
./install.sh
```

This installs `red` to `$HOME/.local/opt/red`, drops a wrapper at `$HOME/.local/bin/red`, and creates a `.desktop` entry. Use `./uninstall.sh` to remove.
