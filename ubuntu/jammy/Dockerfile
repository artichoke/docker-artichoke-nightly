# Build stage
FROM --platform=$BUILDPLATFORM ubuntu:22.04 AS build

# Set shell options
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Install necessary build dependencies
RUN apt-get update && env DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    binutils-aarch64-linux-gnu \
    binutils-x86-64-linux-gnu \
    ca-certificates \
    clang \
    gcc-aarch64-linux-gnu \
    gcc-x86-64-linux-gnu \
    git \
    libc6-dev-arm64-cross \
    libclang-dev \
    wget \
    && rm -rf /var/lib/apt/lists/*

# Rust setup
#
# https://github.com/rust-lang/docker-rust/blob/10f1b7cab43bde8b4d6299aa4d91a895a0e5388d/1.64.0/bullseye/slim/Dockerfile#L3-L37
ARG RUST_VERSION=1.70.0

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

# Add multi-platform GNU target support
RUN set -eux; \
    rustup target add x86_64-unknown-linux-gnu; \
    rustup target add aarch64-unknown-linux-gnu;

# Cargo does not set the correct linker by default when acting as a cross
# compiler. Tell cargo to use the aarch64 GCC toolchain's linker.
#
# https://github.com/rust-lang/rust/issues/28924
ENV CARGO_TARGET_AARCH64_UNKNOWN_LINUX_GNU_LINKER=aarch64-linux-gnu-gcc

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

ENV BINDGEN_EXTRA_CLANG_ARGS_aarch64-unknown-linux-gnu=--sysroot=/usr/aarch64-linux-gnu

# The `ARTICHOKE_NIGHTLY_VER` build arg allows building a specific revision of
# Artichoke. The GitHub Actions workflow sets this to the latest trunk commit
# SHA in the upstream Artichoke repository.
#
# If this build arg is not set via the docker command line, the `trunk` branch
# is built.
ARG ARTICHOKE_NIGHTLY_VER=latest
# Install artichoke
RUN set -eux; \
    case "${TARGETARCH}" in \
        amd64) artichokeArch='x86_64-unknown-linux-gnu' ;; \
        arm64) artichokeArch='aarch64-unknown-linux-gnu'  ;; \
        *) echo >&2 "unsupported architecture: ${TARGETARCH}"; exit 1 ;; \
    esac; \
    if [ "$ARTICHOKE_NIGHTLY_VER" = "latest" ]; then \
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
        --rev "$ARTICHOKE_NIGHTLY_VER" \
        --locked \
        artichoke; \
    fi; \
    mkdir /target; \
    touch /target/THIRDPARTY;

# Copy `THIRDPARTY` license manifest if it exists
#
# Define a build argument to specify the optional file path
ARG THIRDPARTY_FILE
# Check if the THIRDPARTY_FILE build argument is provided.
#
# If provided, copy the file from the host into the build stage image. If not
# provided, the COPY instruction will be skipped
COPY ${THIRDPARTY_FILE} /target/THIRDPARTY

# Runtime stage
FROM ubuntu:22.04 AS runtime

LABEL org.opencontainers.image.authors="Artem Yakimenko <http://temikus.net>, Ryan Lopopolo <rjl@hyperbo.la>"
LABEL org.opencontainers.image.url="https://github.com/artichoke/docker-artichoke-nightly"
LABEL org.opencontainers.image.source="https://github.com/artichoke/artichoke"
LABEL org.opencontainers.image.vendor="Artichoke Ruby <https://www.artichokeruby.org>"
LABEL org.opencontainers.image.licenses="MIT"
LABEL org.opencontainers.image.ref.name="nightly"
LABEL org.opencontainers.image.title="Artichoke Ruby"
LABEL org.opencontainers.image.description="Distribution of artichoke CLI and airb REPL from Artichoke Ruby"

COPY --from=build /usr/local/cargo/bin/artichoke /opt/artichoke/bin/artichoke
COPY --from=build /usr/local/cargo/bin/airb /opt/artichoke/bin/airb
COPY --from=build /target/THIRDPARTY /opt/artichoke/THIRDPARTY

# Use a non-root user
RUN groupadd --system --gid 1888 artichoke && \
    useradd --no-log-init --system --gid artichoke --uid 1888 artichoke
USER artichoke

ENV ARTICHOKE_BIN /opt/artichoke/bin
ENV PATH $ARTICHOKE_BIN:$PATH

# Set the entry point command for the container to the REPL
CMD [ "airb" ]
