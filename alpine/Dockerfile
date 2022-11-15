FROM --platform=$BUILDPLATFORM ubuntu:20.04 AS build

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN apt-get update && env DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    ca-certificates \
    clang \
    git \
    libclang-dev \
    llvm \
    musl-tools \
    wget \
    && rm -rf /var/lib/apt/lists/*

# Rust setup
ARG RUST_VERSION=1.65.0

ENV RUSTUP_HOME=/usr/local/rustup \
    CARGO_HOME=/usr/local/cargo \
    PATH=/usr/local/cargo/bin:$PATH

RUN set -eux; \
    dpkgArch="$(dpkg --print-architecture)"; \
    case "${dpkgArch##*-}" in \
        amd64) rustArch='x86_64-unknown-linux-gnu'; rustupSha256='5cc9ffd1026e82e7fb2eec2121ad71f4b0f044e88bca39207b3f6b769aaa799c' ;; \
        armhf) rustArch='armv7-unknown-linux-gnueabihf'; rustupSha256='48c5ecfd1409da93164af20cf4ac2c6f00688b15eb6ba65047f654060c844d85' ;; \
        arm64) rustArch='aarch64-unknown-linux-gnu'; rustupSha256='e189948e396d47254103a49c987e7fb0e5dd8e34b200aa4481ecc4b8e41fb929' ;; \
        i386) rustArch='i686-unknown-linux-gnu'; rustupSha256='0e0be29c560ad958ba52fcf06b3ea04435cb3cd674fbe11ce7d954093b9504fd' ;; \
        *) echo >&2 "unsupported architecture: ${dpkgArch}"; exit 1 ;; \
    esac; \
    url="https://static.rust-lang.org/rustup/archive/1.25.1/${rustArch}/rustup-init"; \
    wget --progress=dot:giga "$url"; \
    echo "${rustupSha256} *rustup-init" | sha256sum -c -; \
    chmod +x rustup-init; \
    ./rustup-init -y --no-modify-path --profile minimal --default-toolchain $RUST_VERSION --default-host ${rustArch}; \
    rm rustup-init; \
    chmod -R a+w $RUSTUP_HOME $CARGO_HOME; \
    rustup --version; \
    cargo --version; \
    rustc --version;

# Add multi-platform musl target support.
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

# Install bindgen
ARG BINDGEN_PACKAGE=bindgen-cli
ARG BINDGEN_VERSION=0.61.0
RUN set -eux; \
    rustup component add rustfmt; \
    cargo install --version $BINDGEN_VERSION --locked $BINDGEN_PACKAGE; \
    bindgen --version

ARG TARGETPLATFORM
ARG TARGETARCH
ARG TARGETVARIANT

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
        amd64) artichokeArch='x86_64-unknown-linux-musl' ;; \
        arm64) artichokeArch='aarch64-unknown-linux-musl' ;; \
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

COPY empty.txt THIRDPART[Y] /target/

# Setup runtime
FROM alpine:3 AS runtime

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

RUN addgroup --system --gid 128142 artichoke && \
    adduser --system -G artichoke --uid 128142 artichoke
USER artichoke

ENV ARTICHOKE_BIN /opt/artichoke/bin
ENV PATH $ARTICHOKE_BIN:$PATH

CMD [ "airb" ]
