FROM debian:bullseye-slim AS build

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    ca-certificates \
    clang \
    git \
    libclang-dev \
    llvm-dev \
    pkg-config \
    wget \
    && rm -rf /var/lib/apt/lists/*

# Rust setup
ARG RUST_VERSION=1.58.1

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

COPY <<EOF $CARGO_HOME/config.toml
[net]
git-fetch-with-cli = true
EOF

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
FROM debian:bullseye-slim AS runtime

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
