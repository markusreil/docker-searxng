ARG SEARXNG_VERSION=latest
FROM searxng/searxng:${SEARXNG_VERSION}

ENV BASE_IMAGE=searxng/searxng
ARG BUILD_DATE=unknown
ENV BUILD_DATE=${BUILD_DATE}

COPY docker/settings.yml /usr/local/searxng/settings.template.yml
