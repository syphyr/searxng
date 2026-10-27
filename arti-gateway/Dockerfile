FROM ubuntu:latest

# 1. Install utilities, OpenSSL 3.5 libraries, pkg-config helper, iproute2 diagnostics, and Rust compiler stack
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    git \
    ca-certificates \
    libssl-dev \
    libsqlite3-dev \
    pkg-config \
    iproute2 \
    rustc \
    cargo \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /workspace

# 2. Clone the absolute correct upstream Tor Project repository
RUN git clone --depth 1 https://gitlab.torproject.org/tpo/core/arti.git \
    && cd arti \
    && cargo build --release --locked -p arti --features=full \
    && cp target/release/arti /usr/local/bin/arti \
    && cp -av crates/arti/src/arti-example-config.toml /etc/arti.toml

# 3. Create the dedicated, unprivileged system user with a home directory
RUN groupadd -g 10022 _arti && \
    useradd -u 10022 -g _arti -c "Arti Tor Daemon" -m -d /var/lib/arti -s /usr/sbin/nologin _arti && \
    chown -R _arti:_arti /etc/arti.toml

# 4a. Open up the proxy listener to accept both IPv4 and IPv6 traffic across ollama-net
RUN sed -ri 's/^#socks_listen.+/socks_listen = ["0.0.0.0:9050", "[::]:9050"]/' /etc/arti.toml

# 4b. TUNING ENGINE: Surgically override default settings with performance parameters
RUN sed -ri 's/^#stream_isolation = .+/stream_isolation = "none"/' /etc/arti.toml && \
    sed -ri 's/^#max_unused_duration = "3m"/max_unused_duration = "5m"/' /etc/arti.toml && \
    sed -ri 's/^#max_circuit_dirtiness = "10m"/max_circuit_dirtiness = "20m"/' /etc/arti.toml && \
    sed -ri 's/^#parallelism = 3/parallelism = 4/' /etc/arti.toml

# 5. Clean up the source tree to save image space
RUN rm -rf /workspace/arti

# Isolate execution under our dedicated '_arti' identity
USER _arti
EXPOSE 9050

# Force Arti to use the configuration file
ENTRYPOINT ["arti", "-c", "/etc/arti.toml"]
CMD ["proxy"]
