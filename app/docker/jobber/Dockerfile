ARG BUILDKIT_INLINE_CACHE

FROM jobber:latest
ARG TZ
ENV TZ=${TZ}
USER root
RUN apk update && \
    apk add curl wget ca-certificates tzdata && \
    rm -rf /var/cache/apk/*
USER jobberuser