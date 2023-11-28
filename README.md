# Artichoke Nightly Docker

[![GitHub Actions](https://github.com/artichoke/docker-artichoke-nightly/workflows/CI/badge.svg)](https://github.com/artichoke/docker-artichoke-nightly/actions)
[![Nightly Build](https://github.com/artichoke/docker-artichoke-nightly/workflows/Docker%20Nightly%20Builder/badge.svg)](https://github.com/artichoke/docker-artichoke-nightly/actions)
[![Discord](https://img.shields.io/discord/607683947496734760)](https://discord.gg/QCe2tp2)
[![Twitter](https://img.shields.io/twitter/follow/artichokeruby?label=Follow&style=social)](https://twitter.com/artichokeruby)

Docker images for nightly builds of [Artichoke Ruby][artichoke-repo].

Pull and run the [latest image from Docker Hub][docker-hub]:

[artichoke-repo]: https://github.com/artichoke/artichoke
[docker-hub]: https://hub.docker.com/r/artichokeruby/artichoke

```console
$ docker run -it docker.io/artichokeruby/artichoke airb
```

## Quickstart

For the workflow to work 2 secrets need to be setup in repository settings:

- `secrets.DOCKERHUB_USER`
- `secrets.DOCKERHUB_TOKEN`

## Platforms

Currently supported docker platforms are:

- `ubuntu` - canonical mainline Ubuntu 22.04 Jammy Jellyfish image, tagged with
  `latest`, `ubuntu-nightly`, `ubuntu-jammy-nightly`, and `ubuntu22.04-nightly`.
  Ubuntu images are multi-arch images with `linux/amd64` and `linux/arm64`
  support.
- `debian-slim` - Debian 11 (Bullseye) slim image, tagged `slim-nightly` and
  `slim-bullseye-nightly`. Debian images are multi-arch images with
  `linux/amd64` and `linux/arm64` support.
- `alpine` - Alpine 3 image, tagged with `alpine-nightly` and `alpine3-nightly`.
  Alpine images only have `linux/amd64` architecture support.
