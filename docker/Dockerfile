FROM alpine:latest as base

ARG NODE_ENV
ENV NODE_ENV=${NODE_ENV:-production}

FROM node:16-alpine as frontend-builder

WORKDIR /builder

# Install dependencies
ADD ./client/package.json .
ADD ./client/package-lock.json .
RUN npm ci

ADD ./client .

# Build the production build
RUN npm run build

FROM base as runner

# Install dependencies
RUN apk add --no-cache \
          supervisor \
          nginx

# Add nginx configuration
RUN rm -rf /etc/nginx/http.d
ADD docker/nginx/http.d /etc/nginx/http.d/
ADD docker/nginx/nginx.conf /etc/nginx/nginx.conf

# Add Supervisor configuration
RUN mkdir -p /etc/supervisor/conf.d
ADD docker/supervisor/supervisord.conf /etc/supervisor
ADD docker/supervisor/app.ini /etc/supervisor/conf.d

WORKDIR /app

# Add the compiled frontend
COPY --from=frontend-builder /builder/dist ./dist

# Add the entrypoint and set it
ADD docker/entrypoint.sh /entrypoint.sh

ENTRYPOINT [ "/bin/sh", "/entrypoint.sh" ]
