{ center #title }
# OCaml Package Management with (only!) Dune

{ pause up=title }

{ pause }

Let's make an OCaml project with Dune:
{ pause }
- with external dependencies
{ pause }
- using only Dune to install the dependencies
{ pause }
- and using Dune to install development tools

{ pause up }
## If you want to follow along in the audience...

You'll need Dune!

Package management has been (quietly) enabled in Dune for about a year.

If you have Dune (>= 3.19) then you're already good to go!

Otherwise...

### Install Dune with Opam

Recommended if you already have Opam:

```
$ opam install dune
```

### Binary installer

If you don't have Opam on your machine, follow the instructions at:
```
https://github.com/ocaml-dune/dune-bin-install
```

{ pause up }
## Make a New Dune Project

```
$ dune init project hello
$ cd hello
$ dune exec hello  # should print "Hello, World!"
```

### Generated files which we'll modify

`bin/main.ml`:
```ocaml
let () = print_endline "Hello, World!"
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

{ pause up }
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

{ pause down }

{ pause up }
## Add a Dependency
## Install Tools
