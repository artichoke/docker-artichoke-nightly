# Build stage
FROM --platform=$BUILDPLATFORM ubuntu:24.04 AS build

# Set shell options
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Install necessary build dependencies
RUN apt-get update && env DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    ca-certificates \
    clang \
    git \
    libclang-dev \
    musl-tools \
    wget \
    && rm -rf /var/lib/apt/lists/*

# Rust setup
ARG RUST_VERSION=1.91.0

ENV RUSTUP_HOME=/usr/local/rustup \
    CARGO_HOME=/usr/local/cargo \
    PATH=/usr/local/cargo/bin:$PATH

RUN set -eux; \
    dpkgArch="$(dpkg --print-architecture)"; \
    case "${dpkgArch##*-}" in \
        amd64) rustArch='x86_64-unknown-linux-gnu'; rustupSha256='6aeece6993e902708983b209d04c0d1dbb14ebb405ddb87def578d41f920f56d' ;; \
        armhf) rustArch='armv7-unknown-linux-gnueabihf'; rustupSha256='3c4114923305f1cd3b96ce3454e9e549ad4aa7c07c03aec73d1a785e98388bed' ;; \
        arm64) rustArch='aarch64-unknown-linux-gnu'; rustupSha256='1cffbf51e63e634c746f741de50649bbbcbd9dbe1de363c9ecef64e278dba2b2' ;; \
        i386) rustArch='i686-unknown-linux-gnu'; rustupSha256='0a6bed6e9f21192a51f83977716466895706059afb880500ff1d0e751ada5237' ;; \
        ppc64el) rustArch='powerpc64le-unknown-linux-gnu'; rustupSha256='079430f58ad4da1d1f4f5f2f0bd321422373213246a93b3ddb53dad627f5aa38' ;; \
        s390x) rustArch='s390x-unknown-linux-gnu'; rustupSha256='e7f89da453c8ce5771c28279d1a01d5e83541d420695c74ec81a7ec5d287c51c' ;; \
        *) echo >&2 "unsupported architecture: ${dpkgArch}"; exit 1 ;; \
    esac; \
    url="https://static.rust-lang.org/rustup/archive/1.27.1/${rustArch}/rustup-init"; \
    wget --progress=dot:giga "$url"; \
    echo "${rustupSha256} *rustup-init" | sha256sum -c -; \
    chmod +x rustup-init; \
    ./rustup-init -y --no-modify-path --profile minimal --default-toolchain $RUST_VERSION --default-host ${rustArch}; \
    rm rustup-init; \
    chmod -R a+w $RUSTUP_HOME $CARGO_HOME; \
    rustup --version; \
    cargo --version; \
    rustc --version;

# Add multi-platform musl target support
RUN set -eux; \
    rustup target add x86_64-unknown-linux-musl; \
    rustup target add aarch64-unknown-linux-musl;

# To get a cross compiler for aarch64-unknown-linux-musl, use clang and override
# the `cc` crate's compiler detection, which defaults to `aarch64-linux-musl-gcc`.
#
# https://github.com/briansmith/ring/issues/1414#issuecomment-1055177218
ENV CC_aarch64_unknown_linux_musl=clang
ENV AR_aarch64_unknown_linux_musl=llvm-ar
ENV CARGO_TARGET_AARCH64_UNKNOWN_LINUX_MUSL_RUSTFLAGS="-Clink-self-contained=yes -Clinker=rust-lld"

# This option tells Cargo to use the `git` binary instead of `libgit2` for git
# operations, which is significantly faster.
#
# https://doc.rust-lang.org/cargo/reference/config.html#netgit-fetch-with-cli
ENV CARGO_NET_GIT_FETCH_WITH_CLI=true
# This option tells Cargo to use the HTTP-based `sparse` protocol for accessing
# crates.io, which only downloads the necessary parts of the index and is faster.
#
# https://doc.rust-lang.org/cargo/reference/config.html#registriescrates-ioprotocol
ENV CARGO_REGISTRIES_CRATES_IO_PROTOCOL=sparse

ARG TARGETPLATFORM
ARG TARGETARCH
ARG TARGETVARIANT

# The `ARTICHOKE_NIGHTLY_REVISION` build arg allows building a specific revision
# of Artichoke. The GitHub Actions workflow sets this to the latest trunk commit
# SHA in the upstream Artichoke repository.
#
# If this build arg is not set via the docker command line, the `trunk` branch
# is built.
ARG ARTICHOKE_NIGHTLY_REVISION=latest
# Install artichoke
RUN set -eux; \
    case "${TARGETARCH}" in \
        amd64) artichokeArch='x86_64-unknown-linux-musl' ;; \
        arm64) artichokeArch='aarch64-unknown-linux-musl' ;; \
        *) echo >&2 "unsupported architecture: ${TARGETARCH}"; exit 1 ;; \
    esac; \
    if [ "$ARTICHOKE_NIGHTLY_REVISION" = "latest" ]; then \
      cargo install \
        --target "$artichokeArch" \
        --git https://github.com/artichoke/artichoke \
        --branch trunk \
        --bins \
        --locked \
        artichoke; \
    else \
      cargo install \
        --target "$artichokeArch" \
        --git https://github.com/artichoke/artichoke \
        --rev "$ARTICHOKE_NIGHTLY_REVISION" \
        --bins \
        --locked \
        artichoke; \
    fi; \
    mkdir /target; \
    touch /target/THIRDPARTY;

# Copy `THIRDPARTY` license manifest if it exists
#
# If provided, copy the file from the host into the build stage image. If not
# provided, the COPY instruction will be skipped
COPY THIRDPART[Y] /target/THIRDPARTY

# Runtime stage
FROM alpine:3 AS runtime

ARG ARTICHOKE_NIGHTLY_VERSION=latest
ARG ARTICHOKE_NIGHTLY_REVISION=latest
ARG BUILD_TIMESTAMP=unknown

LABEL org.opencontainers.image.created=$BUILD_TIMESTAMP
LABEL org.opencontainers.image.authors="Artem Yakimenko <http://temikus.net>, Ryan Lopopolo <rjl@hyperbo.la>"
LABEL org.opencontainers.image.url="https://github.com/artichoke/docker-artichoke-nightly"
LABEL org.opencontainers.image.documentation="https://artichoke.github.io/artichoke/artichoke/"
LABEL org.opencontainers.image.source="https://github.com/artichoke/artichoke"
LABEL org.opencontainers.image.version=$ARTICHOKE_NIGHTLY_VERSION
LABEL org.opencontainers.image.revision=$ARTICHOKE_NIGHTLY_REVISION
LABEL org.opencontainers.image.vendor="Artichoke Ruby"
LABEL org.opencontainers.image.licenses="MIT"
LABEL org.opencontainers.image.ref.name="alpine3-nightly"
LABEL org.opencontainers.image.title="Artichoke Ruby nightly"
LABEL org.opencontainers.image.description="Distribution of artichoke CLI and airb REPL from Artichoke Ruby"
LABEL org.opencontainers.image.base.name="docker.io/library/alpine:3"

COPY --from=build /usr/local/cargo/bin/artichoke /opt/artichoke/bin/artichoke
COPY --from=build /usr/local/cargo/bin/airb /opt/artichoke/bin/airb
COPY --from=build /target/THIRDPARTY /opt/artichoke/THIRDPARTY

# Use a non-root user
RUN addgroup --system --gid 1888 artichoke && \
    adduser --system -G artichoke --uid 1888 artichoke
USER artichoke

ENV ARTICHOKE_BIN=/opt/artichoke/bin
ENV PATH=$ARTICHOKE_BIN:$PATH

# Set the entry point command for the container to the REPL
CMD [ "airb" ]
