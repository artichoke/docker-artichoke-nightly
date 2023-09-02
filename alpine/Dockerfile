# Build stage
FROM --platform=$BUILDPLATFORM ubuntu:22.04 AS build

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
ARG RUST_VERSION=1.72.0

ENV RUSTUP_HOME=/usr/local/rustup \
    CARGO_HOME=/usr/local/cargo \
    PATH=/usr/local/cargo/bin:$PATH

RUN set -eux; \
    dpkgArch="$(dpkg --print-architecture)"; \
    case "${dpkgArch##*-}" in \
        amd64) rustArch='x86_64-unknown-linux-gnu'; rustupSha256='0b2f6c8f85a3d02fde2efc0ced4657869d73fccfce59defb4e8d29233116e6db' ;; \
        armhf) rustArch='armv7-unknown-linux-gnueabihf'; rustupSha256='f21c44b01678c645d8fbba1e55e4180a01ac5af2d38bcbd14aa665e0d96ed69a' ;; \
        arm64) rustArch='aarch64-unknown-linux-gnu'; rustupSha256='673e336c81c65e6b16dcdede33f4cc9ed0f08bde1dbe7a935f113605292dc800' ;; \
        i386) rustArch='i686-unknown-linux-gnu'; rustupSha256='e7b0f47557c1afcd86939b118cbcf7fb95a5d1d917bdd355157b63ca00fc4333' ;; \
        *) echo >&2 "unsupported architecture: ${dpkgArch}"; exit 1 ;; \
    esac; \
    url="https://static.rust-lang.org/rustup/archive/1.26.0/${rustArch}/rustup-init"; \
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
        --locked \
        artichoke; \
    else \
      cargo install \
        --target "$artichokeArch" \
        --git https://github.com/artichoke/artichoke \
        --rev "$ARTICHOKE_NIGHTLY_REVISION" \
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

ENV ARTICHOKE_BIN /opt/artichoke/bin
ENV PATH $ARTICHOKE_BIN:$PATH

# Set the entry point command for the container to the REPL
CMD [ "airb" ]
