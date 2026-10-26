# ============================================================================
# STAGE 1: Download, Verify, and Compile Modern Tor from Source
# ============================================================================
FROM alpine:latest AS builder

# Install the build toolchain, autotools macros, and cryptographic dependencies
RUN apk add --no-cache \
    build-base \
    autoconf \
    automake \
    libtool \
    libevent-dev \
    openssl-dev \
    zlib-dev \
    xz-dev \
    git

# Clone the bleeding-edge stable 0.4.9 maintenance branch straight from the canonical core repo
WORKDIR /src
RUN git clone --depth 1 --branch maint-0.4.9 https://gitlab.torproject.org/tpo/core/tor.git

# Configure, compile, and install the native binaries
WORKDIR /src/tor
RUN ./autogen.sh && \
    ./configure --disable-asciidoc --sysconfdir=/etc --localstatedir=/var && \
    make -j$(nproc) && \
    make install

# ============================================================================
# STAGE 2: Distill into an Ultra-Lightweight Production Runtime
# ============================================================================
FROM alpine:latest

# Pull ONLY the runtime dependencies (no compilers, no source artifacts)
RUN apk add --no-cache libevent openssl zlib xz && \
    mkdir -p /var/lib/tor && \
    chown -R nobody:nobody /var/lib/tor

# Copy the compiled modern binaries directly out of our builder sandbox stage
COPY --from=builder /usr/local/bin/tor /usr/local/bin/tor

# Write the runtime configurations to open SOCKS5 on all interfaces inside the container
RUN mkdir -p /etc/tor && \
    echo "SocksPort 0.0.0.0:9050" > /etc/tor/torrc && \
    echo "HTTPTunnelPort 0.0.0.0:8118" >> /etc/tor/torrc && \
    echo "DataDirectory /var/lib/tor" >> /etc/tor/torrc && \
    echo "RunAsDaemon 0" >> /etc/tor/torrc

# Isolate execution under the unprivileged 'nobody' user for secure execution
USER nobody
EXPOSE 9050 8118

CMD ["tor"]
