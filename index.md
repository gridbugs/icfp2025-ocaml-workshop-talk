<style>
  h1 {
      font-size: 48pt;
  }
  .space-above {
      padding-top: 50px;
  }
  .large-text {
      font-size: 36pt;
  }
  .fullscreen-center {
    padding-top: calc(var(--page-height)/2);
    padding-bottom: calc(var(--page-height)/2);
  }
</style>

{ center="~duration:0" #title }
# OCaml Package Management with (only!) Dune

{ pause up="~duration:0" }
## Opam and Dune

{ pause }

### Opam

- CLI Tool for installing OCaml packages
- Metadata format for packages
- Repository of ~4500 packages

{ pause }

### Dune

- Build system for OCaml projects
- Used by about 3/4 of packages in Opam's repository
- Dependent on Opam to install dependencies... { pause } until now!

{ pause }

### Dune Package Management

- Dune can download and build Opam packages
- Still using Opam metadata format and repository

{ pause }

### Live Demo of Dune Package Management

Let's make an OCaml project using Dune:
- With external dependencies
- Using only Dune to install the dependencies
- And using Dune to install development tools

{ pause up="~duration:0" }
## Project Setup

Unload the Opam environment:
```text
$ eval $(opam env --revert)
```

{ pause }

Make a project:
```text
$ dune init project hello
```

{ pause }

Make a "Lock Directory" (lockdir):
```text
$ dune pkg lock
Solution for dune.lock:
- ocaml.5.3.0
- ocaml-base-compiler.5.3.0
- ocaml-compiler.5.3.0
- ocaml-config.3
```

{ pause }

Build the project with package management!

```text
$ dune exec hello
Hello, World!
```

{ pause up="~duration:0" }
## Add a Dependency

Let's make a little website with the Dream web framework!

{ pause }

Add a dependency on the `dream` package in `dune-project`:
```text
...
(package
 (name hello)
 ...
 (depends ocaml dream))
```

{ pause }

Declare that the `hello` executable uses the `dream` _library_ in `bin/dune`:
```text
(executable
 (public_name hello)
 (name main)
 (libraries hello dream))
```

{ pause }

Build the project with the dependency:
```text
$ dune exec hello
File "dune.lock/lock.dune", line 1, characters 0-0:
Error: The lock dir is not sync with your dune-project
Hint: run dune pkg lock
```

Oops!

Dune currently requires the project to be explicitly locked after each change to its dependencies. Automatically relocking the project is a work-in-progress!

In the meantime...

```text
$ dune pkg lock
Solution for dune.lock:
- angstrom.0.16.1
- asn1-combinators.0.3.2
...
- yojson.3.0.0
- zarith.1.14
```

And build again:
```text
$ dune exec hello
Hello, World!
```

{ pause down }

{ pause up="~duration:0" }
## Modify our program to use the new dependency

Edit `bin/main.ml`:
```ocaml
let () = ...
```

{ pause }
I need LSP!

{ pause up="~duration:0" }
## ~~Modify our program to use the new dependency~~
## Installing LSP

- The LSP server must be compiled with the same compiler as the code it analyzes
- Dune installs your compiler so it must also build your LSP

{ pause }

Install LSP:
```bash
$ dune tools install ocamllsp
```

{ pause }

Where did it go:
```bash
$ dune tools which ocamllsp
_build/_private/default/.dev-tool/ocaml-lsp-server/target/bin/ocamllsp
```

{ pause }

Add it to `PATH`:
```bash
$ eval $(dune tools env)
```

{ pause }

...and install ocamlformat while we're at it:
```bash
$ dune tools install ocamlformat
```

{ pause up="~duration:0" }
## Modify our program to use the new dependency

Edit `bin/main.ml`:
```ocaml
let () = ...
```

{ pause up="~duration:0" }
## Modify our program to use the new dependency

Edit `bin/main.ml`:
```ocaml
let () =
  let open Dream in
  let handler _ = html "<h1>Hello, World!</h1>" in
  run (router [ get "/" handler ])
```

Run the site:
```text
$ dune exec hello
15.10.25 04:00:10.097       Running at http://localhost:8080
15.10.25 04:00:10.097       Type Ctrl+C to stop
```

{ pause up="~duration:0" }
## Hacking on your dependencies

```text
$ git clone https://github.com/aantron/dream
```

To use a custom source location for a dependency, "pin" it in `dune-project`:

```text
(pin
 (url "file:///path/to/dream")
 (package
  (name dream)))
```

And re-solve dependencies (again, soon this won't be necessary!):

```text
$ dune pkg lock
```

Rebuild and run including the change to the pinned `dream` package:
```text
$ dune exec hello
15.10.25 04:00:10.097       Running at http://localhost:8081
15.10.25 04:00:10.097       Type Ctrl+C to stop
```

{ pause up="~duration:0" .fullscreen-center }
# Peeking under the hood

{ pause up="~duration:0" }
## What's in a Lock Directory?

### Metadata file `lock.dune`
```text
(lang package 0.1)

(dependency_hash 99133f158db3a6d424c473358549c957)

(ocaml ocaml-base-compiler)

(repositories
 (complete true)
 (used
  ((source
    https://github.com/ocaml-dune/opam-overlays.git#2a9543286ff0e0656058fee5c0da7abc16b8717d))
  ((source
    https://github.com/ocaml/opam-repository.git#deb3de7fc5bdf4eb6ebbacd9c3207c8d6820bc64))))

(expanded_solver_variable_bindings
 (variable_values
  (with-doc false)
  (with-dev-setup false)
  (post true)
  (os-distribution homebrew)
  (os macos)
  (opam-version 2.2.0~alpha-vendored)
  (arch arm64))
 (unset_variables with-test sys-ocaml-version sys-ocaml-libc dev build))
```

{ up }
### Lockfiles, e.g. `dream.pkg`
```text
(version 1.0.0~alpha8)

(build
 (run dune build -p %{pkg-self:name} -j %{jobs}))

(depends
 base-unix
 ...
 uri
 yojson)

(source
 (fetch
  (url
   https://github.com/aantron/dream/releases/download/1.0.0-alpha8/dream-1.0.0-alpha8.tar.gz)
  (checksum
   sha256=23ed812890c03fe5c9974a4961a9e8e62126bed7bc7d7d1440b84652c95cf296)))
```

{ pause }

- Dune uses lock _directories_ rather than lock files for easy of code review of dependency changes.

- It's a work-in-progress to generate lockdirs inside the _build directory by
default rather than the project root directory.

- Think of a lockdir as a local cache of an opam package solution rather than a
lockfile in the sense of say `package-lock.json` or `Cargo.lock`

- The package solution in a lockdir is specialized to the machine where it was
generated. They are not portable between machines. Consider this when
deciding whether to check them into version control.

{ pause down }

{ pause up="~duration:0" }
## Portable Lockdirs Prototype

- This feature is a prototype for a lockfile format which is generalized to work on multiple platforms.

- Safe to check into version control.

- A Software Bill Of Materials for all the platforms where it will be deployed.

```bash
$ export DUNE_CONFIG__PORTABLE_LOCK_DIR=enabled
$ dune pkg lock
```

- The feature flag is only necessary for _generating_ portable lockfiles.

- Without the flag, Dune will still interpret portable lockfiles if it sees them.

{ pause up="~duration:0" }
## Where are the packages?

Project and tool dependencies in:
```text
_build/_private/default/.pkg/<name>.<version>-<hash>
```

- `source` directory contains copy of the project source

- `target` directory contains built artifacts akin to an Opam switch

{ pause up="~duration:0" }
## Custom Opam Repos

Opam repo configuration goes in `dune-workspace`:

```text
(lang dune 3.20)

(repository
 (name repo1)
 (url
  file:///local/path/to/repo))

(repository
 (name repo2)
 (url
  git+https://example.com/repo2.git))

(lock_dir
 (repositories repo1 repo2))
```

{ pause up="~duration:0" }
## Freezing the package repo

Specify revisions to freeze opam repo states:

```text
(lang dune 3.20)

(repository
 (name upstream_frozen)
 (url
  git+https://github.com/ocaml/opam-repository.git#12d0447fd13754c2d6cce56abb2ff54637046559))

(repository
 (name overlay_frozen)
 (url
  git+https://github.com/ocaml-dune/opam-overlays.git#2a9543286ff0e0656058fee5c0da7abc16b8717d))

(lock_dir
 (repositories upstream_frozen overlay_frozen))
```

The dependency solver is a pure function of:
 - the Opam repo(s)
 - the platform (e.g. macOS aarch64)
 - the dependencies listed in `dune-project`

Freezing the repositories lets you choose when you update your dependencies
without needing to check a lock directory into version control.

{ pause up="~duration:0" }
# Future Work

{ .large-text .space-above }
> { pause }
> - Automatically solving dependencies (no more `dune pkg lock`)
> { pause }
> - Allow building projects with circular deps via test deps (e.g. Dune itself via `ppx_expect`!)
> { pause }
> - Windows support
> { pause }
> - Package maintenance features (e.g. building reverse dependencies to test impact of local changes)
> { pause }
> - Dune packages for system packages (apt, brew, etc)
> { pause }
> - More features from Opam (e.g. package search)
