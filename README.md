Zig for Debian and Ubuntu
=========================

This repository contains the `debian/` directory for building [Zig][]
from source as Debian and Ubuntu packages. It follows the layout of
Ubuntu's `zig0.16` packaging: one source package per Zig release series,
built with CMake against the distribution's LLVM.

[Zig]: https://ziglang.org

Packages
--------

| Package        | Contents                                                       |
| :------------- | :------------------------------------------------------------- |
| `zig0.17`      | The compiler, `/usr/lib/zig0.17/bin/zig`, linked as `/usr/bin/zig0.17` |
| `zig0.17-dev`  | The standard library (`/usr/lib/zig0.17/lib`) and documentation |

The package names include the release series, so different series can be
installed side by side. Each one registers `/usr/bin/zig` (and the `zig(1)`
man page) with `update-alternatives`. The newest installed series is the
default; to choose another:

```bash
$ sudo update-alternatives --config zig
```

If Ubuntu's `zig-default` package is installed, it owns `/usr/bin/zig`
and the alternative is not used.

Supported systems
-----------------

Zig 0.17 needs LLVM 22, so the build needs a distribution that ships
`llvm-22-dev`, for example Ubuntu 26.04 LTS (Resolute) or Debian unstable.
Supported architectures: `amd64`, `arm64`, `riscv64`, `s390x`.

Building
--------

### 1. Install the tools

```bash
$ sudo apt install build-essential devscripts equivs curl jq minisign
```

### 2. Check out the packaging

```bash
$ mkdir zig && cd zig
$ git clone https://github.com/danudey/zig-debian.git debian
$ sudo mk-build-deps -ir debian/control
```

### 3. Build

```bash
$ debuild -us -uc          # source and binary packages
$ debuild -S -us -uc       # source package only
$ debuild -b -us -uc       # binary packages only
```

Any of `debian/rules build`, `dpkg-buildpackage` or `debuild` works
from a checkout that contains only `debian/`. The first step finds no
upstream source, so it runs `debian/fetch-orig-source --unpack`, which:

1. downloads the source tarball for the version in `debian/changelog`
   from <https://ziglang.org/download/>;
2. checks its SHA-256 against upstream's `index.json`, and its minisign
   signature against the Zig release key in `debian/upstream/signing-key`;
3. removes the files listed in `Files-Excluded` in `debian/copyright`
   (with `mk-origtargz`) and writes `../zig0.17_0.17.0~us1.orig.tar.xz`;
4. unpacks that tarball next to `debian/`.

An existing orig tarball in the parent directory is reused. To only
download it, run `debian/rules get-orig-source`.

The build runs upstream's behaviour tests; set `DEB_BUILD_OPTIONS=nocheck`
to skip them. Expect the build to take 20 minutes or more.

Installation
------------

```bash
$ sudo apt install ./zig0.17_*.deb ./zig0.17-dev_*.deb
$ zig0.17 version
```

Maintenance
-----------

### Updating to a new upstream release

1. Run `debian/update-upstream` from the package directory. It finds the
   newest release on ziglang.org and:
   - renames the packages and `debian/zig0.X*` files when the series
     changes (for example `zig0.17` to `zig0.18`);
   - adds a changelog entry with a link to upstream's release notes;
   - downloads and verifies the new orig tarball;
   - updates the LLVM version in `debian/control`, `debian/rules` and
     `debian/*.links` to the one upstream's `CMakeLists.txt` asks for.
2. Review the changes: `git -C debian diff`.
3. Delete the old upstream source next to `debian/` (everything except
   `debian/`), then build as above.
4. Run `lintian -I ../zig0.*.changes` and update the overrides and
   `debian/copyright` if upstream's files changed.

The changelog entry only links to upstream's release notes.
`debian/changelog` records changes to the Debian package; upstream's
changes belong in their release notes.

Origin
------

This repository started as a copy of Arto Bendiken's
[dryzig/zig-debian](https://github.com/dryzig/zig-debian), which
repackaged upstream's prebuilt Zig 0.6.0 binaries (2020), and keeps its
history. The current source-based packaging follows Ubuntu's `zig0.16`
package by nick black, Nick Hastings and Pushkar Kulkarni, available from
[Launchpad](https://launchpad.net/ubuntu/+source/zig0.16) and
[Salsa](https://salsa.debian.org/nickblack/zig0.15).

License
-------

The packaging is distributed under the Expat (MIT) license; see
[LICENSE](LICENSE). It includes work from Ubuntu's `zig0.16` packaging,
which is under the same license. The original 2020 packaging was released
into the public domain under the Unlicense. Zig itself, and the libraries
it bundles, are under their own licenses, listed in
[`copyright`](copyright).
