# Builder stage
FROM node:22-alpine AS builder

WORKDIR /app

# Copy monorepo root files
COPY package.json yarn.lock ./
COPY scripts ./scripts/
COPY patches ./patches/

# Copy workspace packages
COPY packages/happy-server/package.json ./packages/happy-server/
COPY packages/happy-wire/package.json ./packages/happy-wire/
COPY packages/happy-server/prisma ./packages/happy-server/prisma

# Install dependencies with wire build skipped
RUN SKIP_HAPPY_WIRE_BUILD=1 yarn install --immutable --frozen-lockfile

# Build happy-wire
RUN cd packages/happy-wire && yarn build

# Copy happy-server source files
COPY packages/happy-server/sources ./packages/happy-server/sources

# Generate Prisma client
RUN cd packages/happy-server && yarn generate

# Runner stage
FROM node:22-alpine

WORKDIR /app

# Install runtime dependencies
RUN apk add --no-cache \
    ffmpeg \
    python3 \
    bash

# Copy node_modules from builder
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/packages/happy-server/node_modules ./packages/happy-server/node_modules

# Copy happy-server source, prisma, and generated files
COPY --from=builder /app/packages/happy-server/sources ./packages/happy-server/sources
COPY --from=builder /app/packages/happy-server/prisma ./packages/happy-server/prisma
COPY --from=builder /app/packages/happy-server/node_modules/.prisma ./packages/happy-server/node_modules/.prisma

# Set working directory
WORKDIR /app/packages/happy-server

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# Start the server
CMD ["node", "sources/main.js"]
