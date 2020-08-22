FROM ubuntu:18.04 AS build

RUN apt-get update \
    && apt-get install -y libssl-dev \
    llvm-dev clang libclang-dev \
    wget git cmake build-essential pkg-config

# Rust setup
ARG RUST_VERSION=1.44.0

ENV RUSTUP_HOME=/usr/local/rustup \
    CARGO_HOME=/usr/local/cargo \
    PATH=/usr/local/cargo/bin:$PATH

RUN set -eux; \
    dpkgArch="$(dpkg --print-architecture)"; \
    case "${dpkgArch##*-}" in \
        amd64) rustArch='x86_64-unknown-linux-gnu'; rustupSha256='ad1f8b5199b3b9e231472ed7aa08d2e5d1d539198a15c5b1e53c746aad81d27b' ;; \
        armhf) rustArch='armv7-unknown-linux-gnueabihf'; rustupSha256='6c6c3789dabf12171c7f500e06d21d8004b5318a5083df8b0b02c0e5ef1d017b' ;; \
        arm64) rustArch='aarch64-unknown-linux-gnu'; rustupSha256='26942c80234bac34b3c1352abbd9187d3e23b43dae3cf56a9f9c1ea8ee53076d' ;; \
        i386) rustArch='i686-unknown-linux-gnu'; rustupSha256='27ae12bc294a34e566579deba3e066245d09b8871dc021ef45fc715dced05297' ;; \
        *) echo >&2 "unsupported architecture: ${dpkgArch}"; exit 1 ;; \
    esac; \
    url="https://static.rust-lang.org/rustup/archive/1.21.1/${rustArch}/rustup-init"; \
    wget "$url"; \
    echo "${rustupSha256} *rustup-init" | sha256sum -c -; \
    chmod +x rustup-init; \
    ./rustup-init -y --no-modify-path --profile minimal --default-toolchain $RUST_VERSION; \
    rm rustup-init; \
    chmod -R a+w $RUSTUP_HOME $CARGO_HOME; \
    rustup --version; \
    cargo --version; \
    rustc --version;

# Add libmusl support for Alpine
RUN set -eux; \
    rustup target add x86_64-unknown-linux-musl

# Ruby setup
ARG RUBY_VERSION=2.6.3
ARG RUBY_INSTALL_VERSION=0.7.0
ARG RUBY_INSTALL_SHA=500a8ac84b8f65455958a02bcefd1ed4bfcaeaa2bb97b8f31e61ded5cd0fd70b

RUN set -eux; \
    echo 'gem: --no-document' >> /usr/local/etc/gemrc; \
    url="https://github.com/postmodern/ruby-install/archive/v$RUBY_INSTALL_VERSION.tar.gz"; \
    wget -O /tmp/ruby-install.tar.gz "$url"; \
    echo "${RUBY_INSTALL_SHA} /tmp/ruby-install.tar.gz" | sha256sum -c -; \
    tar -xzvf /tmp/ruby-install.tar.gz -C /opt && rm /tmp/ruby-install.tar.gz; \
    /opt/ruby-install-$RUBY_INSTALL_VERSION/bin/ruby-install -i /usr/local/ ruby $RUBY_VERSION -- --disable-install-rdoc; \
    rm /usr/local/src/ruby-$RUBY_VERSION.tar.bz2; \
    gem update --system; \
    ruby --version; \
    gem --version; \
    bundle --version;

# XXX HACK - The `ARTICHOKE_NIGHTLY_VER` build arg is unused, but can be set at
# build time to break layer caching and force a rebuild of Artichoke from latest
# trunk. The GitHub Actions workflow sets this to the latest trunk commit SHA in
# the upstream Artichoke repository.
ARG ARTICHOKE_NIGHTLY_VER=latest
# Intall musl-gcc for alpine taget
RUN apt-get install -y musl-tools
# Install artichoke
RUN set -eux; \
    RUSTFLAGS="-C link-arg=-s" cargo install --git https://github.com/artichoke/artichoke --locked artichoke \
    --target x86_64-unknown-linux-musl; \
    airb --version; \
    artichoke --version;

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

ENV ARTICHOKE_BIN /opt/artichoke/bin
ENV PATH $ARTICHOKE_BIN:$PATH

CMD [ "airb" ]