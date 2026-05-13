FROM node:18-alpine AS builder

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM nginx:alpine AS runtime

RUN apk add --no-cache gettext

COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf.template

ENV BACKEND_DESPACHOS_HOST=backend-despachos \
    BACKEND_VENTAS_HOST=backend-ventas

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=5s --start-period=15s --retries=3 \
    CMD wget -q --spider http://127.0.0.1:8080/ || exit 1

USER nginx

ENTRYPOINT ["/bin/sh", "-c", "envsubst '${BACKEND_DESPACHOS_HOST} ${BACKEND_VENTAS_HOST}' < /etc/nginx/nginx.conf.template > /tmp/nginx.conf && exec nginx -c /tmp/nginx.conf -g 'daemon off;'"]
