#####################################
#          compile-image            #
#####################################
ARG IMAGE

FROM wcp-goldeneye-team-docker-virtual.artifactory.swg-devops.com/ubi8/ubi-minimal:8.6 AS compile-image

ARG REDHAT_USERNAME
ARG REDHAT_PASSWORD

# hadolint ignore=DL3033,DL3041
RUN set -x \
    && microdnf install -y yum subscription-manager \
    && microdnf upgrade -y \
    && microdnf clean all \
    && dnf clean all \
    && set +x \
    && subscription-manager register --username ${REDHAT_USERNAME} --password ${REDHAT_PASSWORD} --auto-attach \
    && set -x \
    && yum --assumeyes install squid --downloadonly --destdir /tmp/ \
    && yum --assumeyes upgrade --downloadonly --destdir /tmp/ \
    && yum clean all

#####################################
#           build-image             #
#####################################
ARG IMAGE

FROM wcp-goldeneye-team-docker-virtual.artifactory.swg-devops.com/ubi8/ubi-minimal:8.6 AS build-image

ENV SQUID_CACHE_DIR=/var/spool/squid \
    SQUID_LOG_DIR=/var/log/squid \
    SQUID_USER=squid

ENV HOME=/home/"${SQUID_USER}"

COPY --from=compile-image /tmp /tmp
COPY --chmod=755 entrypoint.sh /sbin/entrypoint.sh

# Add runtime user and group, shadow-utils contains these commands
# hadolint ignore=DL3041
RUN set -x \
    && microdnf install -y shadow-utils systemd dnf \
    && microdnf upgrade -y \
    && groupadd --gid 1000 --system "${SQUID_USER}" \
    && useradd --uid 1000 --system --gid "${SQUID_USER}" --create-home "${SQUID_USER}" \
    && dnf install -y /tmp/*.rpm \
    # resolve SSL-related configuration issues found by VA scan
    && dnf remove -y httpd sqlite mariadb* \
    && rm -rf /etc/httpd /usr/lib64/httpd /usr/include/mysql /usr/include/mysql/mysql \
    && microdnf clean all \
    && dnf clean all \
    && chown -R ${SQUID_USER}:${SQUID_USER} /var/run/ \
    && chown -R ${SQUID_USER}:${SQUID_USER} /etc/squid/

USER ${SQUID_USER}

EXPOSE 3128/tcp
ENTRYPOINT ["/sbin/entrypoint.sh"]
