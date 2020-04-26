# Artichoke Nightly Docker

[![GitHub Actions](https://github.com/artichoke/docker-artichoke-nightly/workflows/CI/badge.svg)](https://github.com/artichoke/docker-artichoke-nightly/actions)
[![Discord](https://img.shields.io/discord/607683947496734760)](https://discord.gg/QCe2tp2)
[![Twitter](https://img.shields.io/twitter/follow/artichokeruby?label=Follow&style=social)](https://twitter.com/artichokeruby)

Docker images for nightly builds of
[Artichoke Ruby](https://github.com/artichoke/artichoke).

Pull and run the latest image:

```
$ docker run -it docker.io/artichokerb/artichoke airb
```

## Quickstart

For the workflow to work 2 secrets need to be setup in repository settings:

- `secrets.DOCKERHUB_USER`
- `secrets.DOCKERHUB_TOKEN`

# Workflows

## Docker-nightly

### Platforms

Currently supported docker platforms are:

- `ubuntu` - canonical mainline image, tagged with `-ubuntu` and `latest`
- `debian-slim` - Debian 10 (Buster) slim image
- `alpine` - Alpine 3 image

### Secrets

- `secrets.DOCKERHUB_USER` - DockerHub username that the workflow will use to
  push the image
- `secrets.DOCKERHUB_TOKEN` - DockerHub user token that the workflow will
  authorise with

### Build Args

- `RUST_VERSION` - Rust toolchain version
- `RUBY_VERSION` - Version of the ruby interpreter to build
- `RUBY_INSTALL_VERSION` - Version of the `ruby-install` tool to use
- `RUBY_INSTALL_SHA` - SHA256 of the `ruby-install` package (e.g.
  `openssl dgst -sha256 PACKAGE`)
- `ARTICHOKE_NIGHTLY_VER` - Argument used for cache invalidation (see
  `Dockerfile`), defaults to `date -u +'%Y%m%d%H%M%S'`.
