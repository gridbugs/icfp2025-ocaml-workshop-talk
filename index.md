<style>
  h1 {
      font-size: 48pt;
  }
  .space-above {
      padding-top: 100px;
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

{ pause up=title }

{ pause }

{ .large-text .space-above }
> Let's make an OCaml project with Dune:
> { pause }
> - with external dependencies
> { pause }
> - using only Dune to install the dependencies
> { pause }
> - and using Dune to install development tools

{ pause up="~duration:0" }
## Make a New Dune Project

```
$ dune init project hello
```

{ up }
### Generated files which we'll modify

`bin/main.ml`:
```ocaml
let () = print_endline "Hello, World!"
```

`bin/dune`:
```
(executable
 (public_name hello)
 (name main)
 (libraries hello))
```

`dune-project`:
```
(lang dune 3.20)

(name hello)
...
(package
 (name hello)
 ...
 (depends ocaml))
```

{ pause up="~duration:0" }
## Build with Package Management

The project currently depends on a single package: `ocaml`.

Make a "Lock Directory" (lockdir):
```
$ dune pkg lock
Solution for dune.lock:
- ocaml.5.3.0
- ocaml-base-compiler.5.3.0
- ocaml-compiler.5.3.0
- ocaml-config.3
```

Enable feature to print build progress:
```
$ export DUNE_CONFIG__PKG_BUILD_PROGRESS=enabled
```

Clean up the previously built artifacts:
```
$ dune clean
```

Build the project with package management!

```
$ dune build
 Downloading ocaml-compiler.5.3.0
    Building ocaml-compiler.5.3.0
 Downloading ocaml-base-compiler.5.3.0
    Building ocaml-base-compiler.5.3.0

Hello, World!
```
{ down }

{ pause up="~duration:0" }
## Dune as a Package Manager

- Solves dependencies within one or more Opam repositories
- Transcribes metadata from Opam files into sexp files in a "lockdir"

{ .remark }
> Lockdirs contain source urls and build commands for each package in the
> project's transitive dependency closure.
>
> The package solution in a lockdir is specialized to the machine where it was
> generated. They are not portable between machines. Consider this when
> deciding whether to check them into version control.
>
> Think of a lockdir as a local cache of an opam package solution rather than a
> lockfile in the sense of say `package-lock.json` or `Cargo.lock`.
>
> It's a work-in-progress to generate lockdirs inside the _build directory by
> default rather than the project root directory.

More details on lockdirs later in this talk!

{ pause up="~duration:0" }
## Add a Dependency

Let's make a little website with Dream!

Add a dependency on the `dream` package in `dune-project`:
```
...
(package
 (name hello)
 ...
 (depends ocaml dream))
```

Declare that the `hello` executable uses the `dream` _library_ in `bin/dune`:
```
(executable
 (public_name hello)
 (name main)
 (libraries hello dream))
```

Build the project with the dependency:
```text
$ dune build
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

Build, downloading and building all deps:
```text
$ dune build
    Building base-threads.base
    Building conf-pkg-config.4
...
 Downloading dream.1.0.0~alpha8
    Building dream.1.0.0~alpha8
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

Install LSP:
```bash
$ dune tools install ocamllsp
```

Where did it go:
```bash
$ dune tools which ocamllsp
_build/_private/default/.dev-tool/ocaml-lsp-server/target/bin/ocamllsp
```

Add it to `PATH`:
```bash
$ eval $(dune tools env)
```

...and install ocamlformat while we're at it:
```
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

And re-solve dependencies:

```text
$ dune pkg lock
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

{ pause up="~duration:0" .fullscreen-center }
# Extras

{ pause up="~duration:0" }
## Binary Install Script

If you don't have Opam you can install Dune by running:

``
$ curl -4fsSL https://github.com/ocaml-dune/dune-bin-install/releases/download/v3/install.sh | sh
``

- Runs an interactive installer which installs a pre-built version of Dune for your system.
- Follows installation conventions. Dune will be installed to `~/.local/bin` by default.
- Doesn't modify your shell config file without asking first.
- Installs shell completions for bash and zsh!

{ pause up="~duration:0" }
## Portable Lockfiles

- Dune's lockfiles are specialized to the machine where they are generated.
- This feature is a prototype for a lockfile format which is generalized to work on multiple platforms.
- Safe to check into version control!

```bash
$ export DUNE_CONFIG__PORTABLE_LOCK_DIR=enabled
$ dune pkg lock
```

- The feature flag is only necessary for _generating_ portable lockfiles.
- Without the flag, Dune will still interpret portable lockfiles if it sees them.

{ pause up="~duration:0" }
# Future Work

{ .large-text .space-above }
> - Automatically solving dependencies (no more `dune pkg lock`)
> { pause }
> - Allow building projects with circular deps via test deps (e.g. Dune itself via `ppx_expect`!)
> { pause }
> - Windows support
> { pause }
> - More features from Opam (e.g. package search)
