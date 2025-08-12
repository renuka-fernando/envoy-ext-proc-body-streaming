FROM --platform=linux/amd64 ubuntu:20.04

# Add Envoy user
RUN groupadd --system envoy \
    && useradd --system --create-home --gid envoy envoy

# Install runtime dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    libc++1 \
    libgoogle-perftools4 \
    && rm -rf /var/lib/apt/lists/*

# Copy Envoy binary and entrypoint script
COPY envoy-static /usr/local/bin/envoy
COPY docker-entrypoint.sh /docker-entrypoint.sh

# Set permissions
RUN chmod +x /docker-entrypoint.sh \
    && chmod +x /usr/local/bin/envoy \
    && chown envoy:envoy /docker-entrypoint.sh \
    && chown envoy:envoy /usr/local/bin/envoy

# Set environment variables
ENV ENVOY_UID=0
ENV ENVOY_GID=0

EXPOSE 9901
EXPOSE 10000

ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["envoy", "-c", "/etc/envoy/envoy.yaml"]
