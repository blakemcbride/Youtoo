# Youtoo - EuLisp System

Youtoo is a public domain implementation of the
[EuLisp](https://en.wikipedia.org/wiki/EuLisp) programming language.
It compiles EuLisp Level-1 source into C-embedded bytecodes that are
statically or dynamically linked with the Eutopia virtual machine.

Youtoo was originally created by Andreas Kind and Julian Padget at the
University of Bath (1997) and subsequently maintained by Henry G.
Weller.  This fork is maintained by Blake McBride.

**Repository:** <https://github.com/blakemcbride/Youtoo>

## Changes in This Fork

This fork modernizes the build system and brings the implementation
into conformance with the EuLisp 0.991 specification.

### Build System

- Added a `configure` script that auto-detects the OS, architecture,
  C compiler, and library paths (Boehm GC, readline)
- Simplified Makefile clean targets to just `clean` and `realclean`
- Fixed build artifacts (`Modules/`, `Tools/u2/`,
  `Runtime/u2/level-{0,1}_.c`) that previously survived `make clean`
- Added REPL tab-completion for EuLisp keywords and Youtoo bindings

### EuLisp 0.991 Spec Conformance

Renamed functions and generics to match the specification:

| Old Name | New Name | Module |
|---|---|---|
| `find` | `find-key` | collect |
| `collectionp` | `collection?` | collect |
| `emptyp` | `empty?` | collect |
| `fpi?` | `int?` | fpi / boot1 |
| `most-positive-fpi` | `most-positive-int` | fpi |
| `most-negative-fpi` | `most-negative-int` | fpi |
| `maximum-vector-size` | `maximum-vector-index` | vector |
| `alphap` | `alpha?` | character |
| `alnump` | `alnum?` | character |
| `namep` | `name?` | symbol |
| `slot-reader` | `slot-slot-reader` | mop-class |
| `slot-writer` | `slot-slot-writer` | mop-class |
| `slot-default` | `slot-default-function` | mop-class |
| `slotp` | `slot?` | mop-inspect |
| `methodp` | `method?` | mop-inspect |
| `sprint-char` | `sprin-char` | stream |
| `print-char` | `prin-char` | stream |

The `find-key` generic was also given the correct EuLisp 0.991
semantics (predicate-based search with `skip` and `failure` parameters)
rather than the old value-based `find` behavior.

## Quick Start

```bash
./configure
make static
make test
```

See [BUILD.md](BUILD.md) for full build instructions, prerequisites,
bootstrapping, and compiler usage.

## Project Structure

```
Vm/           Virtual machine (Eutopia) - C bytecode interpreter
Telos/        The EuLisp Object System (meta-object protocol)
Runtime/      Level-0 and level-1 standard libraries
Comptime2/    Compiler (EuLysses) - EuLisp to C bytecode
Youtoo/       Main executable entry point
Test/         Test suite
Tools/        Build utilities (b2h, i2c, etc.)
include/      C headers for VM and FFI
Lib/          Pre-compiled library interface files
EuLisp/       Language specification and documentation
```

## License

GNU General Public License v2.  See [LICENSE](LICENSE) for details.
