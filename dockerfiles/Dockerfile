#
# Copyright (c) 2018-2019 Red Hat, Inc.
# This program and the accompanying materials are made
# available under the terms of the Eclipse Public License 2.0
# which is available at https://www.eclipse.org/legal/epl-2.0/
#
# SPDX-License-Identifier: EPL-2.0
#

###########################################################
# Build /plugins + index.json
###########################################################
FROM alpine:3.10 AS plugin-builder
RUN apk add --no-cache py-pip jq bash && pip install yq jsonschema

COPY ./build/plugins ./build/plugins/meta.yaml.schema /build/
COPY /plugins /build/plugins/
WORKDIR /build/
RUN ./check_plugins_location.sh plugins
RUN ./set_plugin_dates.sh plugins
RUN ./check_plugins_viewer_mandatory_fields.sh plugins
RUN ./ensure_latest_exists.sh plugins
RUN ./index.sh plugins > /build/plugins/index.json

###########################################################
# Build /devfiles + index.json
###########################################################
FROM alpine:3.10 AS devfile-builder
RUN apk add --no-cache py-pip jq bash && pip install yq

# Registry, organization, and tag to use for base images in dockerfiles. Devfiles
# will be rewritten during build to use these values for base images.
ARG PATCHED_IMAGES_REG="quay.io"
ARG PATCHED_IMAGES_ORG="eclipse"
ARG PATCHED_IMAGES_TAG="nightly"

COPY ./build/devfiles ./arbitrary-users-patch/base_images /build/
COPY /devfiles /build/devfiles
WORKDIR /build/
RUN TAG=${PATCHED_IMAGES_TAG} \
    ORGANIZATION=${PATCHED_IMAGES_ORG} \
    REGISTRY=${PATCHED_IMAGES_REG} \
    ./update_devfile_patched_image_tags.sh
RUN ./check_mandatory_fields.sh devfiles
RUN ./index.sh > /build/devfiles/index.json

###########################################################
# Build registry image
###########################################################
FROM registry.centos.org/centos/httpd-24-centos7
RUN mkdir /var/www/html/plugins
COPY .htaccess README.md /var/www/html/
COPY --from=devfile-builder /build/devfiles/ /var/www/html/devfiles/
COPY --from=plugin-builder /build/plugins/ /var/www/html/plugins/
USER 0
RUN chmod -R g+rwX /var/www/html/

USER 10001