# Buildx parameters
ARG BUILDPLATFORM
ARG BUILDOS
ARG BUILDARCH
ARG BUILDVARIANT
ARG TARGETPLATFORM
ARG TARGETOS
ARG TARGETARCH
ARG TARGETVARIANT

# Shiper system parameters


# Shiper build parameters
ARG BUILD_PARAM_NODE_VERSION=22
ARG BUILD_PARAM_PACKAGE_MANAGER=npm
ARG BUILD_PARAM_BUILD_COMMAND=npm run build
ARG BUILD_PARAM_RUN_COMMAND=node server.js

# Base image stage
FROM node:${BUILD_PARAM_NODE_VERSION}-alpine AS base
RUN apk add --no-cache libc6-compat
RUN npm install -g pnpm
RUN npm install -g yarn

# Build dependencies stage
FROM base AS deps
WORKDIR /app
COPY --chown=app:app . .
# Install dependencies
ARG BUILD_PARAM_PACKAGE_MANAGER
RUN if [ "${BUILD_PARAM_PACKAGE_MANAGER}" = "pnpm" ]; then \
  pnpm install --frozen-lockfile; \
  elif [ "${BUILD_PARAM_PACKAGE_MANAGER}" = "yarn" ]; then \
  yarn install --frozen-lockfile; \
  else \
  npm install --production; \
  fi

# Build stage
FROM base AS builder
RUN apk add --no-cache git
WORKDIR /app
COPY --from=deps /app /app
ARG BUILD_PARAM_BUILD_COMMAND
RUN ${BUILD_PARAM_BUILD_COMMAND}

# Runner stage
FROM base AS runner
WORKDIR /app
RUN addgroup --system --gid 1001 nodejs && \
  adduser --system --uid 1001 nextjs --ingroup nodejs

# Copy public directory if it exists
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

USER nextjs
EXPOSE 3000
ENV PORT=3000
ENV NODE_ENV=production
ARG BUILD_PARAM_RUN_COMMAND
# Set the run command as an environment variable to allow use in ENTRYPOINT
ENV BUILD_PARAM_RUN_COMMAND=${BUILD_PARAM_RUN_COMMAND}
ENTRYPOINT ["/bin/sh", "-c", "$BUILD_PARAM_RUN_COMMAND"]
