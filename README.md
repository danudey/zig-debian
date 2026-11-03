Zig for Debian and Ubuntu
=========================

This repository contains the [`debuild`][] configuration for building
[Zig][] binary packages for Debian and Ubuntu from upstream's signed
[binary release tarballs](https://ziglang.org/download/).

[Zig]:     https://ziglang.org
[`debuild`]: https://manpages.debian.org/testing/devscripts/debuild.1.en.html

Supported systems
-----------------

The package repacks upstream's statically linked compiler, so one `.deb`
works on any reasonably current Debian-based system. Zig 0.17.0 packages
have been tested on:

| Distribution     | Alias    | AMD64 |
| :--------------- | :------- | :---- |
| Ubuntu 26.04 LTS | Resolute | ✔     |
| Ubuntu 24.04 LTS | Noble    | ✔     |
| Ubuntu 22.04 LTS | Jammy    | ✔     |
| Debian 13        | Trixie   | ✔     |
| Debian 12        | Bookworm | ✔     |

ARM64 (`arm64`) packages can be built the same way. They have been built
and tested on Debian 13 (Trixie) under emulation.

Installation
------------

Build the package (see [Building](#building)), then install it with `apt`
so that dependencies are handled:

```bash
$ sudo apt install ./zig_0.17.0-1_amd64.deb
```

The compiler is installed to `/usr/lib/zig`, with `/usr/bin/zig` linked to
it.

Building
--------

### 1. Install the build tools

```bash
$ sudo apt install build-essential debhelper devscripts curl jq minisign
```

### 2. Fetch and verify the upstream release

```bash
$ mkdir zig-build && cd zig-build
$ git clone https://github.com/dryzig/zig-debian.git debian
$ debian/fetch-orig-source
```

`fetch-orig-source` downloads the release for the version in
`debian/changelog` and the build machine's architecture. It refuses to
continue unless both checks pass:

- the SHA-256 listed in upstream's `index.json`
- the minisign signature from the Zig release key
  (`RWSGOq2NVecA2UPNdBUZykf1CCb147pkmdtYxgb3Ti+JO/wCYvhbAb/U`)

It then writes `zig_<version>.orig.tar.xz`. To build for a different
version or architecture, pass them explicitly, for example
`debian/fetch-orig-source 0.17.0 arm64`.

### 3. Build

```bash
$ tar xf zig_0.17.0.orig.tar.xz
$ mv debian zig-0.17.0/
$ cd zig-0.17.0
$ debuild -us -uc
```

The build checks that the binary matches the target architecture, and
compiles and runs a small program with the packaged compiler.

Maintenance
-----------

### Updating to a new upstream release

1. Check for a new release: `uscan --report` (from the source tree).
2. Add a changelog entry: `dch -v <version>-1 "New upstream release."`
3. Follow [Building](#building) from step 2.
4. Run `lintian -I ../zig_<version>-1_amd64.changes` and update the
   overrides if upstream's file layout changed.
