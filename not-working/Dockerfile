################################################################################
# Copyright 2022 IBM Corp. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
################################################################################
#######################################
# Build the preliminary image
#######################################
FROM wcp-goldeneye-team-docker-virtual.artifactory.swg-devops.com/external-secrets/external-secrets:v0.5.9 as buildImg

#######################################
# Get latest RH updates
#######################################
FROM wcp-goldeneye-team-docker-virtual.artifactory.swg-devops.com/ubi8/ubi-minimal:8.6 AS update-image

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

#######################################
# Build the production image
#######################################

FROM wcp-goldeneye-team-docker-virtual.artifactory.swg-devops.com/ubi8/ubi-minimal:8.6 AS build-image

COPY --from=buildImg /bin/external-secrets /bin/external-secrets
COPY --from=update-image /tmp /tmp

# hadolint ignore=DL3041
RUN microdnf upgrade -y \
    && microdnf install -y dnf \
    && dnf install -y /tmp/*.rpm \
    # resolve SSL-related configuration issues found by VA scan
    && dnf remove -y httpd sqlite mariadb* \
    && rm -rf /etc/httpd /usr/lib64/httpd /usr/include/mysql /usr/include/mysql/mysql \
    && microdnf clean all \
    && dnf clean all

USER 65534

ENTRYPOINT ["/bin/external-secrets"]
