FROM node:22.13-alpine AS node_upstream

# Base node image
FROM node_upstream AS node_base

ENV NODE_PORT=${NODE_PORT}

# Create app directory
WORKDIR /usr/src/app


# Dev node image
FROM node_base AS node_dev

COPY --link ../docker/docker-entrypoint.sh /usr/local/bin/docker-entrypoint
RUN chmod +x /usr/local/bin/docker-entrypoint

EXPOSE ${NODE_PORT}
ENTRYPOINT ["docker-entrypoint"]

ENV NODE_ENV=development
CMD ["yarn", "dev-host"]


# Prod node image
FROM node_base AS node_builder

WORKDIR /app
COPY ../ .

RUN yarn install
RUN yarn build


# Serve frontend using a lightweight web server
FROM nginx:alpine AS node_prod

# Set default port if not provided
ENV NGINX_PORT=${NGINX_PORT:-3000}

# Copy nginx configuration template
COPY ../docker/nginx.conf.template /etc/nginx/templates/default.conf.template

# Copy built frontend files
COPY --from=node_builder /app/dist /usr/share/nginx/html

# Expose the port dynamically
EXPOSE ${NGINX_PORT}

# Replace environment variables in the Nginx config before running
CMD envsubst '$NGINX_PORT' < /etc/nginx/templates/default.conf.template > /etc/nginx/conf.d/default.conf && \
    nginx -g "daemon off;"
