# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is the Scala 2 compiler, standard library, and language specification repository. The codebase uses a bootstrapping build: the current version is built by the previously released version ("STARR" - stable reference release).

## Build Commands

All commands run in sbt shell (start with `sbt`):

```bash
compile                    # Compile all subprojects
dist/mkBin                 # Generate runner scripts in build/quick/bin
dist/mkPack                # Create distribution in build/pack
publishLocal               # Publish to local ivy/maven repository
```

### Running the Compiler/REPL

```bash
scala                      # Run REPL from sbt
scalac                     # Run compiler from sbt (accepts options/arguments)
```

## Testing

### JUnit Tests
```bash
junit/test                 # Run all JUnit tests
junit/testOnly *Foo        # Run specific test class
junit/testQuick            # Run only tests affected by recent changes
```

### ScalaCheck Tests
```bash
scalacheck/test            # Run all property-based tests
scalacheck/testOnly FooTest # Run specific test
```

### Partest (Integration Tests)
```bash
partest --verbose test/files/neg/foo.scala    # Run single test
partest --neg                                  # Run all neg tests
partest --run                                  # Run all run tests
partest --pos                                  # Run all pos tests
partest --grep range                           # Run tests matching pattern
partest --update-check                         # Update .check files
partest --failed                               # Re-run only failed tests
partest --debug --verbose test/files/pos/foo.scala  # Keep generated files
```

Partest categories in `test/files/`:
- `pos`: Must compile successfully
- `neg`: Must fail to compile, output compared to `.check` file
- `run`: Compiles and runs `Test.main`, output compared to `.check` file
- `jvm`: Same as run

### Benchmarks
```bash
bench/Jmh/run scala.collection.mutable.ListBufferBenchmark
```

## Repository Structure

```
src/
├── compiler/     # Scala compiler (scalac)
├── library/      # Scala standard library
├── reflect/      # Scala reflection library
├── interactive/  # Presentation compiler (IDE support)
├── repl/         # REPL core
├── repl-frontend/# REPL frontend
├── scaladoc/     # Documentation generator
├── scalap/       # Class file decompiler
├── partest/      # Testing framework
└── tastytest/    # TASTy compatibility testing

test/
├── files/        # Partest integration tests
├── junit/        # JUnit unit tests
└── scalacheck/   # Property-based tests
```

## Compiler Architecture

The compiler (`src/compiler/scala/tools/nsc/`) uses a phase-based pipeline:

- `Global.scala`: Main compiler entry point, orchestrates all phases
- `ast/`: Tree definitions and operations
- `typechecker/`: Type checking and inference phases
- `transform/`: Code transformation phases (patmat, tailcalls, erasure, etc.)
- `backend/jvm/`: JVM bytecode generation
- `symtab/`: Symbol table implementation

Key compilation phases in order: parser → namer → typer → patmat → erasure → backend

## Key Conventions

**Binary compatibility**: The standard library cannot add new public classes or methods. Additions go to [scala-library-next](https://github.com/scala/scala-library-next).

**Partest flags**: Add compiler flags to tests via comment: `//> using options -Werror -Xlint`

**Bootstrapping**: Run `restarrFull` for a full bootstrap when changing code generation. CI always does bootstrapped builds.

**Fatal warnings**: CI enables `-Werror`. Enable locally with `set Global / fatalWarnings := true`.

**Sandbox directory**: Use `sandbox/` for local test files (gitignored).

## Issues and PRs

- Bug tracker: [scala/bug](https://github.com/scala/bug)
- Target branch: `2.13.x` for most changes
- Link bugs in PR descriptions: "Fixes scala/bug#NNNN"
- CI commands: `/rebuild` to retry failed builds
