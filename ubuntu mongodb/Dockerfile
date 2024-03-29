# Use Ubuntu 20.04 LTS as base image
FROM ubuntu:focal

ARG TARGETARCH

# Metadata as provided
LABEL com.vmware.cp.artifact.flavor="sha256:1e1b4657a77f0d47e9220f0c37b9bf7802581b93214fff7d1bd2364c8bf22e8e" \
      org.opencontainers.image.base.name="ubuntu:focal" \
      org.opencontainers.image.created="2024-02-02T13:45:59Z" \
      org.opencontainers.image.description="Application packaged by VMware, Inc" \
      org.opencontainers.image.licenses="Apache-2.0" \
      org.opencontainers.image.ref.name="7.0.5-ubuntu-focal" \
      org.opencontainers.image.title="mongodb" \
      org.opencontainers.image.vendor="VMware, Inc." \
      org.opencontainers.image.version="7.0.5"

ENV HOME="/" \
    OS_ARCH="${TARGETARCH:-amd64}" \
    OS_FLAVOUR="ubuntu-focal" \
    OS_NAME="linux"

COPY prebuildfs /
SHELL ["/bin/bash", "-o", "errexit", "-o", "nounset", "-o", "pipefail", "-c"]

# Install required system packages and dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates curl libbrotli1 libcom-err2 libcurl4 libffi7 libgcc-s1 libgcrypt20 libgmp10 \
    libgnutls30 libgpg-error0 libgssapi-krb5-2 libhogweed6 libidn2-0 libk5crypto3 libkeyutils1 libkrb5-3 \
    libkrb5support0 libldap-2.4-2 liblzma5 libnettle8 libnghttp2-14 libp11-kit0 libpsl5 librtmp1 \
    libsasl2-2 libssh2-1 libssl1.1 libtasn1-6 libunistring2 numactl procps zlib1g

# Bitnami specific commands
# Note: Check the compatibility of the components with Ubuntu.
RUN mkdir -p /tmp/bitnami/pkg/cache/ ; cd /tmp/bitnami/pkg/cache/ ; \
    COMPONENTS=( \
      "yq-4.40.5-3-linux-${OS_ARCH}-ubuntu-focal" \
      "wait-for-port-1.0.7-7-linux-${OS_ARCH}-ubuntu-focal" \
      "render-template-1.0.6-7-linux-${OS_ARCH}-ubuntu-focal" \
      "mongodb-shell-2.1.3-0-linux-${OS_ARCH}-ubuntu-focal" \
      "mongodb-7.0.5-3-linux-${OS_ARCH}-ubuntu-focal" \
    ) ; \
    for COMPONENT in "${COMPONENTS[@]}"; do \
      if [ ! -f "${COMPONENT}.tar.gz" ]; then \
        curl -SsLf "https://downloads.bitnami.com/files/stacksmith/${COMPONENT}.tar.gz" -O ; \
        curl -SsLf "https://downloads.bitnami.com/files/stacksmith/${COMPONENT}.tar.gz.sha256" -O ; \
      fi ; \
      sha256sum -c "${COMPONENT}.tar.gz.sha256" ; \
      tar -zxf "${COMPONENT}.tar.gz" -C /opt/bitnami --strip-components=2 --no-same-owner --wildcards '*/files' ; \
      rm -rf "${COMPONENT}".tar.gz{,.sha256} ; \
    done

# Cleaning up
RUN apt-get purge -y --auto-remove curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /var/cache/apt/archives

# File permissions and other configurations
RUN chmod g+rwX /opt/bitnami
RUN find / -perm /6000 -type f -exec chmod a-s {} \; || true

COPY rootfs /
RUN /opt/bitnami/scripts/mongodb/postunpack.sh
ENV APP_VERSION="7.0.5" \
    BITNAMI_APP_NAME="mongodb" \
    PATH="/opt/bitnami/common/bin:/opt/bitnami/mongodb/bin:$PATH"

EXPOSE 27017

USER 1001
ENTRYPOINT [ "/opt/bitnami/scripts/mongodb/entrypoint.sh" ]
CMD [ "/opt/bitnami/scripts/mongodb/run.sh" ]
