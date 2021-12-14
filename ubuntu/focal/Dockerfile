FROM ubuntu:20.04 AS build

RUN apt-get update && env DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    bison \
    build-essential \
    ca-certificates \
    clang \
    cmake \
    git \
    libclang-dev \
    libgdbm-dev \
    libffi-dev \
    libncurses5-dev \
    libreadline-dev \
    libssl-dev \
    libyaml-dev \
    llvm-dev \
    musl-tools \
    pkg-config \
    wget \
    xz-utils \
    zlib1g-dev \
    && rm -rf /var/lib/apt/lists/*

# Rust setup
ARG RUST_VERSION=1.57.0

ENV RUSTUP_HOME=/usr/local/rustup \
    CARGO_HOME=/usr/local/cargo \
    PATH=/usr/local/cargo/bin:$PATH

SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN set -eux; \
    dpkgArch="$(dpkg --print-architecture)"; \
    case "${dpkgArch##*-}" in \
        amd64) rustArch='x86_64-unknown-linux-gnu'; rustupSha256='ed7773edaf1d289656bdec2aacad12413b38ad0193fff54b2231f5140a4b07c5' ;; \
        armhf) rustArch='armv7-unknown-linux-gnueabihf'; rustupSha256='7a7b9d246ad63358705d8d4a7d5c2ef1adfec24525d1d5c44a7739e1b867e84d' ;; \
        arm64) rustArch='aarch64-unknown-linux-gnu'; rustupSha256='f80a0a792b3ab905ab4919474daf4d3f60e574fc6987e69bfba2fd877241a8de' ;; \
        i386) rustArch='i686-unknown-linux-gnu'; rustupSha256='4473c18286aa1831683a772706d9a5c98b87a61cc014d38063e00a63a480afef' ;; \
        *) echo >&2 "unsupported architecture: ${dpkgArch}"; exit 1 ;; \
    esac; \
    url="https://static.rust-lang.org/rustup/archive/1.23.1/${rustArch}/rustup-init"; \
    wget --progress=dot:giga "$url"; \
    echo "${rustupSha256} *rustup-init" | sha256sum -c -; \
    chmod +x rustup-init; \
    ./rustup-init -y --no-modify-path --profile minimal --default-toolchain $RUST_VERSION --default-host ${rustArch}; \
    rm rustup-init; \
    chmod -R a+w $RUSTUP_HOME $CARGO_HOME; \
    rustup --version; \
    cargo --version; \
    rustc --version;

# Ruby setup
ARG RUBY_VERSION=3.0.2
ARG RUBY_INSTALL_VERSION=0.8.3
ARG RUBY_INSTALL_SHA=e2f69949757d032d48ee5c028be020bdc8863c41d5648b53328903d2e16ab3b2

RUN set -eux; \
    echo 'gem: --no-document' >> /usr/local/etc/gemrc; \
    url="https://github.com/postmodern/ruby-install/archive/v$RUBY_INSTALL_VERSION.tar.gz"; \
    wget --progress=dot:giga -O /tmp/ruby-install.tar.gz "$url"; \
    echo "${RUBY_INSTALL_SHA} /tmp/ruby-install.tar.gz" | sha256sum -c -; \
    tar -xzvf /tmp/ruby-install.tar.gz -C /opt && rm /tmp/ruby-install.tar.gz; \
    /opt/ruby-install-$RUBY_INSTALL_VERSION/bin/ruby-install --no-install-deps --install-dir /usr/local/ ruby $RUBY_VERSION -- --disable-install-rdoc; \
    rm /usr/local/src/ruby-$RUBY_VERSION.tar.xz; \
    gem update --system; \
    ruby --version; \
    gem --version; \
    bundle --version;

# Install bindgen
RUN set -eux; \
    cargo install --version 0.59.1 bindgen; \
    bindgen --version

# The `ARTICHOKE_NIGHTLY_VER` build arg allows building a specific revision of
# Artichoke. The GitHub Actions workflow sets this to the latest trunk commit
# SHA in the upstream Artichoke repository.
#
# If this build arg is not set via the docker command line, the `trunk` branch
# is built.
ARG ARTICHOKE_NIGHTLY_VER=latest
# Install artichoke
RUN set -eux; \
    if [ "$ARTICHOKE_NIGHTLY_VER" = "latest" ]; then \
      RUSTFLAGS="-C link-arg=-s" cargo install --git https://github.com/artichoke/artichoke --branch trunk --locked artichoke; \
    else \
      RUSTFLAGS="-C link-arg=-s" cargo install --git https://github.com/artichoke/artichoke --rev "$ARTICHOKE_NIGHTLY_VER" --locked artichoke; \
    fi; \
    airb --version; \
    artichoke --version; \
    mkdir /target; \
    touch /target/THIRDPARTY;

COPY empty.txt THIRDPART[Y] /target/

# Setup runtime
FROM ubuntu:20.04 AS runtime

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

RUN groupadd --system --gid 128142 artichoke && \
    useradd --no-log-init --system --gid artichoke --uid 128142 artichoke
USER artichoke

ENV ARTICHOKE_BIN /opt/artichoke/bin
ENV PATH $ARTICHOKE_BIN:$PATH

CMD [ "airb" ]
