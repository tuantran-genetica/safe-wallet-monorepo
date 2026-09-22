# Life AI devnet build of apps/web: a Next.js static export served by nginx.
#
# The upstream image builds Next.js when the container starts and needs roughly
# 3 GiB. This builds once here and ships only the exported files, which lets the
# Cloud Run service run at 0.08 vCPU / 128 MiB.
#
# The install stage mirrors the repository root Dockerfile. Keep them in step when
# upstream changes the Node version or the system packages.
FROM node:24-alpine AS builder
RUN apk add --no-cache libc6-compat git python3 py3-pip make g++ libusb-dev eudev-dev linux-headers

WORKDIR /app
COPY . .

WORKDIR /app/apps/web
RUN corepack enable && yarn config set httpTimeout 300000
RUN yarn install --immutable
RUN yarn after-install

# NEXT_PUBLIC_* values are read at compile time, so a change needs a rebuild.
ARG NEXT_PUBLIC_GATEWAY_URL_PRODUCTION
ARG NEXT_PUBLIC_DEFAULT_MAINNET_CHAIN_ID=94909
ARG NEXT_PUBLIC_SAFE_VERSION=1.4.1
ENV NEXT_PUBLIC_GATEWAY_URL_PRODUCTION=${NEXT_PUBLIC_GATEWAY_URL_PRODUCTION} \
    NEXT_PUBLIC_DEFAULT_MAINNET_CHAIN_ID=${NEXT_PUBLIC_DEFAULT_MAINNET_CHAIN_ID} \
    NEXT_PUBLIC_SAFE_VERSION=${NEXT_PUBLIC_SAFE_VERSION} \
    NEXT_PUBLIC_IS_PRODUCTION=true \
    NEXT_TELEMETRY_DISABLED=1 \
    NODE_OPTIONS=--max-old-space-size=6144
RUN yarn build --no-lint

FROM nginx:1.27-alpine
COPY deploy/lifeai/nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /app/apps/web/out /usr/share/nginx/html
EXPOSE 8080
