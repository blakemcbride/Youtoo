# Building Youtoo

Youtoo is an implementation of the EuLisp programming language. It compiles
EuLisp Level-1 into C-embedded bytecodes linked with the Eutopia virtual
machine.

## Prerequisites

- **GNU Make**
- **GCC** (C99 or later)
- **Boehm garbage collector** (libgc runtime and development headers)
  - Fedora/RHEL: `sudo dnf install gc-devel`
  - Ubuntu/Debian: `sudo apt-get install libgc-dev`
  - macOS (Homebrew): `brew install bdw-gc`
  - Or build from source (default install prefix: `/usr/local`)
- **GNU Readline** (development headers)
  - Fedora/RHEL: `sudo dnf install readline-devel`
  - Ubuntu/Debian: `sudo apt-get install libreadline-dev`
  - macOS (Homebrew): `brew install readline`
- **pthreads** (standard on Linux and macOS)

## Quick Start

```bash
./configure
make static
make test
```

## Configure

The `configure` script detects your OS and architecture and generates:
- `Lib.$(ARCH)/Makefile` — build variables included by all Makefiles
- `.eulrc.$(ARCH)` — runtime configuration for the compiler
- `Bin.$(ARCH)/` and `Lib.$(ARCH)/` — output directories

To override the architecture (e.g., 32-bit on a 64-bit machine):

```bash
./configure i686
```

Environment variables you can set before running `configure`:
- `CC` — C compiler (default: auto-detected, prefers gcc)
- `U2_GC_DIR` — Boehm GC installation prefix (auto-detected from
  `/usr/include/gc/`, `/usr/local/include/gc/`, `/opt/homebrew/include/gc/`,
  or `brew --prefix bdw-gc` on macOS)

On macOS, `configure` auto-detects Homebrew (Apple Silicon and Intel) and
adds the appropriate `-I`/`-L` flags for `bdw-gc` and the keg-only
`readline`. No environment variables are needed if both are installed via
Homebrew.

## Build Targets

| Target | Description |
|---|---|
| `make static` | Build with static libraries (default) |
| `make shared` | Build with shared/dynamic libraries |
| `make test` | Run the test suite (test1–test4) |
| `make clean` | Remove all build artifacts (keeps `Bin.*`, `Lib.*`, `.eulrc.*`) |
| `make realclean` | Full clean including installed binaries and libraries |
| `make all` | Build both 32-bit and 64-bit (on 64-bit machine) |
| `make boot` | Bootstrap the compiler (requires pre-existing `youtoo`) |

## Verify

After building (substitute your architecture — e.g., `arm64` on Apple
Silicon, `x86_64` on Intel/Linux):

```bash
Bin.$(uname -m)/youtoo --version
Bin.$(uname -m)/youtoo --help
make test
```

Expected test output:
```
Testing base ... OK.
Testing module boot ... OK.
Testing module telos ... OK.
Testing module level-1 ... OK.
```

## Running the REPL

```bash
Bin.$(uname -m)/youtoo -i
```

```
EuLisp System Youtoo - Version pre-0.991

user> (+ 1 2)
-> 3
user> (cons 'a '(b c))
-> (a b c)
```

## Build Order

The subsystems compile in dependency order:

1. **Vm/** — C bytecode interpreter → `libeulvm.a`
2. **Telos/** — Object system → `libboot.a`, `libtelos.a`
3. **Runtime/** — Standard libraries → `liblevel-0.a`, `liblevel-1.a`, `libmath.a`
4. **Comptime2/** — Compiler → `libeval.a`, `libmain.a`
5. **Youtoo/** — Links all libraries + `-lgc -lpthread` → `youtoo` executable

## Shared Libraries

When using `make shared`, set `LD_LIBRARY_PATH` before running:

```bash
export LD_LIBRARY_PATH=$(pwd)/Lib.x86_64:$LD_LIBRARY_PATH
```

Platform-specific linker scripts: `Tools/makeso.{Linux,FreeBSD,IRIX,SunOS5}`.

## Bootstrapping

The `make boot` target rebuilds the compiler from its own EuLisp sources.
This requires a **pre-existing working `youtoo`** at
`$(EUL_BOOT_DIR)/Bin.$(ARCH)/youtoo.sh`.

```bash
make clean         # always clean first
make boot          # two-stage bootstrap
```

Bootstrap process:
1. Compiles VM (C sources only)
2. Compiles Telos and Runtime using the bootstrap compiler
3. Builds a preliminary `youtoo` executable
4. Regenerates `Vm/level-1i.c` from interface files via `Tools/i2c`
5. Recompiles everything with the preliminary compiler
6. Links the final executable

**Note:** `make clean` before every `make boot` is recommended — the
dependency analysis does not catch all module interactions.

## Compiler Usage

```
youtoo [OPTION]... [FILE]...
```

Key options (from `--help`):

| Option | Description |
|---|---|
| `-h --help` | Print usage information |
| `-V --version` | Print version |
| `-i --interpret` | Start the interactive REPL |
| `-l --library lib` | Link with library (e.g., `level-1`) |
| `-c --c-module` | Create C module file only (don't link) |
| `--stand-alone` | Create a stand-alone executable |
| `--archive` | Create a library interface file |
| `-O --object-dir dir` | Set output directory |
| `--load-path dir` | Add module search path |
| `--recompile` | Force recompilation of imports |
| `--no-recompile` | Skip dependency checking |
| `--no-gc` | Link without garbage collector |
| `-g --debug` | Generate C debug info |
| `-s --script file` | Run a script file |

## Environment Variables

| Variable | Purpose |
|---|---|
| `EUL_DIR` | Root of the Youtoo installation (auto-detected at build time) |
| `EUL_ARCH` | Architecture string (e.g., `x86_64`) |
| `EUL_LOAD_PATH` | Colon-separated module search path |
| `EUL_LIBRARY_LOAD_PATH` | Colon-separated library search path |
| `LD_LIBRARY_PATH` | Must include `Lib.$(ARCH)/` for shared library builds |

## Architecture Notes

- **64-bit** (`x86_64`, `arm64`): `WORD_LENGTH=64`, `Instruction` = `uint16_t`, pointers = `long`
- **32-bit** (`i686`): `WORD_LENGTH=32`, `Instruction` = `uint8_t`, pointers = `int`
- Architecture-specific code generation: `Comptime2/32bit/` and `Comptime2/64bit/`
- Generated C files: `u2/` subdirectories
- Object files: `platforms/$(ARCH)/` subdirectories
