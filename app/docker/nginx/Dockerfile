ARG BUILDKIT_INLINE_CACHE
ARG NGINX_BASEIMAGE_VERSION

FROM nginx:${NGINX_BASEIMAGE_VERSION}-alpine as base
COPY ./docker/nginx/default.conf.template /etc/nginx/templates/

FROM base as develop

FROM base as production
COPY --from=front --chown=nginx:nginx /app/ .
