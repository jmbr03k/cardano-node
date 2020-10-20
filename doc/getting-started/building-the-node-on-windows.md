# Development build of Cardano Node on Windows

This document describes how to build a __DEVELOPMENT__ version of `cardano-node`.
This is not a __PRODUCTION__ build!

Building on `Windows` is a non trivial task.  There are various tools one needs
to install and configure.  We recommend to use
[chocolatey](https://chocolatey.org) (which provides the `choco` command) to
install them.

All instructrions that invoke `choco` can be done in a PowerShell with root
previlages.  For development work we recommend installing and using `git-bash`
(with `choco install git-bash`).

## Install GHC

The recommended way is to

```PowerShell
choco install --version 8.6.5 ghc
```
in PowerShell>

We are about to transition to `ghc-8.10.2` but this version is broken on
windows ([see](https://gitlab.haskell.org/ghc/ghc/-/issues/18550)), though
a fix is rather easy.

## Install pkg-config

```PowerShell
choco install pgkconfiglite
```

## Install vcpkg

`vcpkg` is needed to install `libsodium` library, which will be the
next step.

To install it one needs `git` (which you can also install using `choco`).  Then
follow [these
instructions](https://github.com/microsoft/vcpkg#quick-start-windows)

Now you can install `libsodium` with:
```bash
./vcpkg install --triplet x64-windows libsodium
```

## Write libsodium.pc file

`cabal` is using `pkg-config` to find system dependencies, like `libsodium`.
On Windows we need to write `libsodium.pc` description file in a correct
directory.

In one of the paths reported by:
```bash
pkg-config --variable pc_path pkg-config
```
create `libsodium.pc` file, which contains (but replace `VCPKG_PATH` with the
absolute path where you put `vcpkg`, please verify that the paths below contain
`libsodium.dll` file and headers).

```
libdir=VCPKG_PATH/installed/x64-windows/bin
includedir=VCPKG_PATH/intsalled/x64-windows/include

Name: libsodium
Description: libsodium library
Cflags: -I${includedir}/sodium
Libs: -L${libdir} -llibsodium
```

Note: one cannot use `prefix=` in `libsodium.pc` file, it might be changed for
some other directory, `pkg-config` provides a switch to use the provided
`prefix`, but there's no way to instruct `cabal` to do so.


## cabal configuration

Now go to the directory where you cloned `cardano-node` repository and add this
to your `cabal.project.local` file (if you don't already have it, create one):

```
max-backjump: 5000
reorder-goals: True

package cardano-crypt-praos
  flags: -external-libsodium-vrf

extra-lib-dirs: "VCPKG_PATH\\installed\\x64-windows\\bin"
extra-include-dirs: "VCPKG_PATH\\installed\\x64-windows\\include"
```

The final part is to add `VCPKG_PATH/installed/x64-windows/bin` to `PATH`
variable, since we are using `git-bash` this can be done with (remember
to substitute `VCPKG_PATH` with real path):

```
export PATH="VCPKG_PATH/installed/x64-windows/bin:${PATH}"
```

Now build the node with:

```bash
cabal build --builddir /c/dist exe:cardano-node
```

And you can verify that it can run:
```bash
cabal run --builddir /c/dist exe:cardano-node -- run --help
```
