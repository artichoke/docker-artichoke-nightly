# Artichoke Ruby

[![GitHub Actions](https://github.com/artichoke/artichoke/workflows/CI/badge.svg)](https://github.com/artichoke/artichoke/actions)
[![Discord](https://img.shields.io/discord/607683947496734760)](https://discord.gg/QCe2tp2)
[![Twitter](https://img.shields.io/twitter/follow/artichokeruby?label=Follow&style=social)](https://twitter.com/artichokeruby)
<br>
[![Crate](https://img.shields.io/crates/v/artichoke.svg)](https://crates.io/crates/artichoke)
[![API](https://docs.rs/artichoke/badge.svg)](https://docs.rs/artichoke)
[![API master](https://img.shields.io/badge/docs-master-blue.svg)](https://artichoke.github.io/artichoke/artichoke/)

<p align="center">
  <a href="https://www.artichokeruby.org">
    <img height="200" width="200" src="https://www.artichokeruby.org/artichoke-logo.svg">
  </a>
</p>

Nightly builds of Artichoke Ruby.

Artichoke is a Ruby implementation written in Rust and Ruby. Artichoke intends
to be [MRI-compatible][ruby-spec] and targets Ruby 2.6.3. Artichoke provides a
Ruby runtime implemented in Rust and Ruby.

[ruby-spec]: https://github.com/ruby/spec

## Install Artichoke

Pull and run the latest image:

```console
$ docker run -it docker.io/artichokeruby/artichoke airb
```

### Platforms

Currently supported docker platforms are:

- `ubuntu` - canonical mainline Ubuntu 20.04 Focal Fossa image, tagged with
  `latest`, `ubuntu-nightly`, `ubuntu-focal-nightly`, and `ubuntu20.04-nightly`.
- `debian-slim` - Debian 11 (Bullseye) slim image, tagged `slim-nightly` and
  `slim-bullseye-nightly`.
- `alpine` - Alpine 3 image, tagged with `alpine-nightly` and `alpine3-nightly`.

## Usage

Artichoke ships with two binaries: `airb` and `artichoke`.

### `airb`

`airb` is the Artichoke implementation of `irb` and is an interactive Ruby shell
and [REPL].

`airb` is a readline-enabled shell, although it does not persist history.

[repl]: https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop

### `artichoke`

`artichoke` is the `ruby` binary frontend to Artichoke.

`artichoke` supports executing programs via files, stdin, or inline with one or
more `-e` flags.

Artichoke can `require`, `require_relative`, and `load` files from the local
file system, but otherwise does not yet support local file system access. A
temporary workaround is to inject data into the interpreter with the
`--with-fixture` flag, which reads file contents into a `$fixture` global.

```console
$ artichoke --help
artichoke 0.1.0-pre.0
Artichoke is a Ruby made with Rust.

USAGE:
    artichoke [OPTIONS] [ARGS]

ARGS:
    <programfile>
    <arguments>...

OPTIONS:
        --copyright                 print the copyright
    -e <commands>                   one line of script. Several -e's allowed. Omit [programfile]
    -h, --help                      Print help information
    -V, --version                   Print version information
        --with-fixture <fixture>    file whose contents will be read into the `$fixture` global
```

## Design and Goals

Artichoke is [designed to enable experimentation][artichoke-vision]. The top
goals of the project are:

- [Support WebAssembly as a build target][wasm-target].
- Support embedding and executing Ruby in untrusted environments.
- [Distribute Ruby applications as single-binary artifacts][a-single-binary].
- [Implement Ruby with state-of-the-art dependencies][a-deps].
- Experiment with VMs to support [dynamic codegen][a-codegen], [ahead of time
  compilation][a-compiler], [parallelism and eliminating the
  GIL][a-parallelism], and novel [memory management and garbage collection
  techniques][a-memory-management].

[artichoke-vision]: https://github.com/artichoke/artichoke/blob/trunk/VISION.md
[wasm-target]: https://github.com/artichoke/artichoke/labels/O-wasm-unknown
[a-single-binary]: https://github.com/artichoke/artichoke/labels/A-single-binary
[a-deps]: https://github.com/artichoke/artichoke/labels/A-deps
[a-codegen]: https://github.com/artichoke/artichoke/labels/A-codegen
[a-compiler]: https://github.com/artichoke/artichoke/labels/A-compiler
[a-parallelism]: https://github.com/artichoke/artichoke/labels/A-parallelism
[a-memory-management]:
  https://github.com/artichoke/artichoke/labels/A-memory-management

## Contributing

Artichoke aspires to be a Ruby 2.6.3-compatible implementation of the Ruby
programming language. [There is lots to do].

If Artichoke does not run Ruby source code in the same way that MRI does, it is
a bug and we would appreciate if you [filed an issue so we can fix it].

If you would like to contribute code 👩‍💻👨‍💻, find an issue that looks interesting
and leave a comment that you're beginning to investigate. If there is no issue,
please file one before beginning to work on a PR. [Good first issues are labeled
`E-easy`].

[there is lots to do]: https://github.com/artichoke/artichoke/issues
[filed an issue so we can fix it]:
  https://github.com/artichoke/artichoke/issues/new
[good first issues are labeled `e-easy`]:
  https://github.com/artichoke/artichoke/labels/E-easy

### Discussion

If you'd like to engage in a discussion outside of GitHub, you can [join
Artichoke's public Discord server].

[join artichoke's public discord server]: https://discord.gg/QCe2tp2

## License

artichoke is licensed with the [MIT License][artichoke-license] (c) Ryan
Lopopolo.

[artichoke-license]: https://github.com/artichoke/artichoke/blob/trunk/LICENSE

Some portions of Artichoke are derived from third party sources. The READMEs in
each crate discuss which third party licenses are applicable to the sources and
derived works in Artichoke.
