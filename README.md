# m1cpu-crash-repro

This repository is a minimal Go project I created to explore a crash related to the `github.com/shoenig/go-m1cpu` library on my new Apple Silicon MacBook Pro (M5 Pro).

My goal is to understand and document the issue that makes the Anytype Mac arm64 app crash on startup on my machine, and to use this as a first step toward contributing to open source.

---

## Background

While trying to run Anytype on my MacBook Pro (M5 Pro, 16", 24 GB RAM), the app crashed immediately on launch. The crash log showed:

- A `SIGSEGV: segmentation violation`
- The crash happened **during cgo execution**
- The stack trace pointed to:

  - `github.com/shoenig/go-m1cpu._Cfunc_initialize()`
  - Go runtime files inside `/opt/homebrew/Cellar/go@1.24/1.24.10/libexec/src/runtime/...`

This suggests that the problem is likely related to `go-m1cpu` running on very new Apple Silicon hardware (M5), and not a simple user configuration issue.

---

## Environment

The tests in this repo were run on:

- **Machine:** MacBook Pro 16" (Apple Silicon M5 Pro, 24 GB RAM)
- **OS:** macOS (Tahoe 26.3.2)
- **Architecture:** arm64
- **Go version:** `go1.24.10` (installed via Homebrew `go@1.24`)
- **go-m1cpu version:** `github.com/shoenig/go-m1cpu@v0.1.6`

The `go-m1cpu` version matches the version referenced in the original crash log from Anytype.

---

## Purpose of this repo

This repository is **not** a full reproduction of Anytype itself.

Instead, it focuses on a very small Go program that:

- Imports `github.com/shoenig/go-m1cpu`  
- Compiles and runs on my Apple Silicon machine
- Can be used to:
  - Check whether `go-m1cpu` initialization crashes on this hardware / Go version
  - Experiment with different `go-m1cpu` versions
  - Provide a simple test case for maintainers of `go-m1cpu` or any application that depends on it

The idea is to have a **minimal, easy-to-run** example that other people can clone and test on their own Apple Silicon devices.

---

## How to run

1. Clone this repository:

   ```bash
   git clone https://github.com/8Bing/m1cpu-crash-repro.git
   cd m1cpu-crash-repro
   ```

2. Make sure Go is installed and on your PATH:

   ```bash
   go version
   ```

3. Run the program:

   ```bash
   go run .
   ```

4. Observe the output:

   - On my M5 Pro, the program currently outputs:

     ```text
     Program started, go-m1cpu imported.
     ```

   - If there is a segmentation fault related to `go-m1cpu` initialization, you would see something similar to:

     ```text
     SIGSEGV: segmentation violation
     signal arrived during cgo execution
     github.com/shoenig/go-m1cpu._Cfunc_initialize()
     ...
     ```

If you reproduce a crash on a different Apple Silicon model (M1, M2, M3, M4, etc.), it would be very helpful to compare environments and share results with the `go-m1cpu` maintainers.

---

## Code overview

The core of this project is a single, minimal `main.go`:

```go
package main

import (
    "fmt"

    _ "github.com/shoenig/go-m1cpu"
)

func main() {
    fmt.Println("Program started, go-m1cpu imported.")
}
```

Key points:

- The library is imported with a blank identifier (`_`) so that its `init()` function runs, even though we do not call any exported functions directly.
- This is enough to trigger any initialization logic inside `go-m1cpu`, including the cgo-based CPU detection that appears in the original crash log.

This tiny program is intentionally simple, so that:
- It is easy to understand, even for beginners (like me).
- It isolates the behavior of `go-m1cpu` from the rest of Anytype or other applications.

---

## Relation to Anytype crash

In my original situation, the Anytype desktop app for macOS could not start on my M5 Pro. The crash log contained entries like:

```text
SIGSEGV: segmentation violation
PC=0x18c3514f8 m=0 sigcode=2 addr=0x0
signal arrived during cgo execution

github.com/shoenig/go-m1cpu._Cfunc_initialize()
    _cgo_gotypes.go:150 +0x30
github.com/shoenig/go-m1cpu.init.0()
    /Users/user1/go/pkg/mod/github.com/shoenig/go-m1cpu@v0.1.6/cpu.go:148 +0x1c
```

This repo is a starting point for:

- Checking whether similar issues can be reproduced in isolation
- Testing newer versions of `go-m1cpu`
- Providing maintainers with a minimal test case that they can run on their own machines

---

## Version comparison on my M5 Pro

On my machine:

- **go-m1cpu v0.1.6**
  - Used by Anytype at the time of the original crash.
  - Anytype desktop app crashed on startup with:
    - `SIGSEGV: segmentation violation`
    - `signal arrived during cgo execution`
    - stack trace including `_Cfunc_initialize()` and `cpu.go` at v0.1.6.

- **go-m1cpu v0.2.0**
  - Includes the commit `fix segfault with m5 cpu (#27)`.
  - When I run this minimal repro project with `v0.2.0`, I still see a crash:
    - `SIGTRAP: trace trap`
    - `signal arrived during cgo execution`
    - stack trace still points to `_Cfunc_initialize()` in `cpu.go` at v0.2.0.

This suggests that the M5-related issues may not be fully resolved in my particular environment yet, and that further investigation or additional fixes/workarounds might be needed.

---

## Next steps / ideas

Some possible follow-up work (for myself or others):

- Test with different Go versions (e.g. Go 1.22, 1.23) to see if the crash is related to a specific Go toolchain.
- Open an issue on the `go-m1cpu` GitHub repository, referencing:
  - This repository as a minimal reproduction
  - My machine / OS / Go / library versions
  - Any crash logs observed here or in Anytype.

---
