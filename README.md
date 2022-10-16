# Artichoke Nightly Docker

[![GitHub Actions](https://github.com/artichoke/docker-artichoke-nightly/workflows/CI/badge.svg)](https://github.com/artichoke/docker-artichoke-nightly/actions)
[![Nightly Build](https://github.com/artichoke/docker-artichoke-nightly/workflows/Docker%20Image%20CI/badge.svg)](https://github.com/artichoke/docker-artichoke-nightly/actions)
[![Discord](https://img.shields.io/discord/607683947496734760)](https://discord.gg/QCe2tp2)
[![Twitter](https://img.shields.io/twitter/follow/artichokeruby?label=Follow&style=social)](https://twitter.com/artichokeruby)

Docker images for nightly builds of [Artichoke Ruby][artichoke-repo].

Pull and run the [latest image from Docker Hub][docker-hub]:

```console
$ docker run -it docker.io/artichokeruby/artichoke airb
```

## Quickstart

For the workflow to work 2 secrets need to be setup in repository settings:

- `secrets.DOCKERHUB_USER`
- `secrets.DOCKERHUB_TOKEN`

## Workflows

### docker-nightly

#### Platforms

Currently supported docker platforms are:

- `ubuntu` - canonical mainline Ubuntu 20.04 Focal Fossa image, tagged with
  `latest`, `ubuntu-nightly`, `ubuntu-focal-nightly`, and `ubuntu20.04-nightly`.
  Ubuntu images are multi-arch images with `linux/amd64` and `linux/arm64`
  support.
- `debian-slim` - Debian 11 (Bullseye) slim image, tagged `slim-nightly` and
  `slim-bullseye-nightly`. Debian images are multi-arch images with
  `linux/amd64` and `linux/arm64` support.
- `alpine` - Alpine 3 image, tagged with `alpine-nightly` and `alpine3-nightly`.
  Alpine images only have `linux/amd64` architecture support.

#### Secrets

- `secrets.DOCKERHUB_USER` - DockerHub username that the workflow will use to
  push the image
- `secrets.DOCKERHUB_TOKEN` - DockerHub user token that the workflow will
  authorise with

#### Build Args

- `RUST_VERSION` - Rust toolchain version.
- `ARTICHOKE_NIGHTLY_VER` - Argument used for cache invalidation (see
  `Dockerfile`), defaults to SHA of latest trunk commit in upstream Artichoke
  repository.

[artichoke-repo]: https://github.com/artichoke/artichoke
[docker-hub]: https://hub.docker.com/r/artichokeruby/artichoke
