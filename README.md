# sh

[![GoDoc](https://godoc.org/mvdan.cc/sh?status.svg)](https://godoc.org/mvdan.cc/sh)

A shell parser, formatter, and interpreter. Supports [POSIX Shell], [Bash], and
[mksh]. Requires Go 1.14 or later.

### Quick start

To parse shell scripts, inspect them, and print them out, see the [syntax
examples](https://godoc.org/mvdan.cc/sh/syntax#pkg-examples).

For high-level operations like performing shell expansions on strings, see the
[shell examples](https://godoc.org/mvdan.cc/sh/shell#pkg-examples).

### shfmt

	GO111MODULE=on go get mvdan.cc/sh/v3/cmd/shfmt

`shfmt` formats shell programs. See [canonical.sh](syntax/canonical.sh) for a
quick look at its default style. For example:

	shfmt -l -w script.sh

For more information, see [its manpage](cmd/shfmt/shfmt.1.scd), which can be
viewed directly as Markdown or rendered with [scdoc].

Packages are available on [Alpine], [Arch], [Docker], [FreeBSD], [Homebrew],
[MacPorts], [NixOS], [Scoop], [Snapcraft], and [Void].

### gosh

	GO111MODULE=on go get mvdan.cc/sh/v3/cmd/gosh

Proof of concept shell that uses `interp`. Note that it's not meant to replace a
POSIX shell at the moment, and its options are intentionally minimalistic.

### Fuzzing

This project makes use of [go-fuzz] to find crashes and hangs in both the parser
and the printer. Note that this requires Go 1.14.x at the moment, since go-fuzz
doesn't support 1.15 or later just yet. The `fuzz-corpus` branch contains a
corpus to get you started. For example:

	git checkout fuzz-corpus
	PATH=$HOME/sdk/go1.14.9/bin:$PATH ./fuzz

### Caveats

* When indexing Bash associative arrays, always use quotes. The static parser
  will otherwise have to assume that the index is an arithmetic expression.

```sh
$ echo '${array[spaced string]}' | shfmt
1:16: not a valid arithmetic operator: string
$ echo '${array[dash-string]}' | shfmt
${array[dash - string]}
```

* `$((` and `((` ambiguity is not supported. Backtracking would complicate the
  parser and make streaming support via `io.Reader` impossible. The POSIX spec
  recommends to [space the operands][posix-ambiguity] if `$( (` is meant.

```sh
$ echo '$((foo); (bar))' | shfmt
1:1: reached ) without matching $(( with ))
```

* Some builtins like `export` and `let` are parsed as keywords. This is to allow
  statically parsing them and building their syntax tree, as opposed to just
  keeping the arguments as a slice of arguments.

### JavaScript

A subset of the Go packages are available as an npm package called [mvdan-sh].
See the [_js](_js) directory for more information.

### Docker

To build a Docker image, checkout a specific version of the repository and run:

	docker build -t my:tag -f cmd/shfmt/Dockerfile .

This creates an image that only includes shfmt. Alternatively, if you want an
image that includes alpine, add `--target alpine`.
To use the Docker image, run:

	docker run --rm -v $PWD:/mnt -w /mnt my:tag <shfmt arguments>

### pre-commit

It is possible to use shfmt with [pre-commit][pre-commit] and a `local`
repo configuration like:

```yaml
  - repo: local
    hooks:
      - id: shfmt
        name: shfmt
        minimum_pre_commit_version: 2.4.0
        language: golang
        additional_dependencies: [mvdan.cc/sh/v3/cmd/shfmt@v3.2.0]
        entry: shfmt
        args: [-w]
        types: [shell]
```

### Related projects

The following editor integrations wrap `shfmt`:

- [format-shell] - Atom plugin
- [intellij-shellcript] - Intellij Jetbrains `shell script` plugin
- [micro] - Editor with a built-in plugin
- [shell-format] - VS Code plugin
- [shfmt.el] - Emacs package
- [Sublime-Pretty-Shell] - Sublime Text 3 plugin
- [vim-shfmt] - Vim plugin

Other noteworthy integrations include:

- Alternative docker image by [PeterDaveHello][dockerized-peterdavehello]
- [modd] - A developer tool that responds to filesystem changes
- [prettier-plugin-sh] - [Prettier] plugin using [mvdan-sh]
- [sh-checker] - A GitHub Action that performs static analysis for shell scripts

[alpine]: https://pkgs.alpinelinux.org/packages?name=shfmt
[arch]: https://www.archlinux.org/packages/community/x86_64/shfmt/
[bash]: https://www.gnu.org/software/bash/
[docker]: https://hub.docker.com/r/mvdan/shfmt/
[dockerized-peterdavehello]: https://github.com/PeterDaveHello/dockerized-shfmt/
[editorconfig]: https://editorconfig.org/
[examples]: https://godoc.org/mvdan.cc/sh/syntax#pkg-examples
[format-shell]: https://atom.io/packages/format-shell
[freebsd]: https://www.freshports.org/devel/shfmt
[go-fuzz]: https://github.com/dvyukov/go-fuzz
[homebrew]: https://formulae.brew.sh/formula/shfmt
[intellij-shellcript]: https://www.jetbrains.com/help/idea/shell-scripts.html
[macports]: https://ports.macports.org/port/shfmt/summary
[micro]: https://micro-editor.github.io/
[mksh]: http://www.mirbsd.org/mksh.htm
[modd]: https://github.com/cortesi/modd
[mvdan-sh]: https://www.npmjs.com/package/mvdan-sh
[nixos]: https://github.com/NixOS/nixpkgs/blob/HEAD/pkgs/tools/text/shfmt/default.nix
[posix shell]: https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html
[posix-ambiguity]: https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html#tag_18_06_03
[prettier]: https://prettier.io
[prettier-plugin-sh]: https://github.com/rx-ts/prettier/tree/master/packages/sh
[scdoc]: https://sr.ht/~sircmpwn/scdoc/
[scoop]: https://github.com/ScoopInstaller/Main/blob/HEAD/bucket/shfmt.json
[sh-checker]: https://github.com/luizm/action-sh-checker
[shell-format]: https://marketplace.visualstudio.com/items?itemName=foxundermoon.shell-format
[shfmt.el]: https://github.com/purcell/emacs-shfmt/
[snapcraft]: https://snapcraft.io/shfmt
[sublime-pretty-shell]: https://github.com/aerobounce/Sublime-Pretty-Shell
[vim-shfmt]: https://github.com/z0mbix/vim-shfmt
[void]: https://github.com/void-linux/void-packages/blob/HEAD/srcpkgs/shfmt/template
[pre-commit]: https://pre-commit.com
