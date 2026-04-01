# Builder stage
FROM node:22 AS builder

WORKDIR /app

# Copy monorepo root files
COPY package.json yarn.lock ./
COPY packages/happy-server/package.json ./packages/happy-server/

# Install dependencies
RUN yarn install --immutable

# Copy happy-server source
COPY packages/happy-server ./packages/happy-server

# Build TypeScript for type checking
RUN cd packages/happy-server && yarn build

# Generate Prisma client
RUN cd packages/happy-server && yarn generate

# Runner stage
FROM node:22-slim

WORKDIR /app

# Install runtime dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    ffmpeg \
    python3 \
    && rm -rf /var/lib/apt/lists/*

# Copy node_modules from builder
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/packages/happy-server/node_modules ./packages/happy-server/node_modules

# Copy happy-server source and generated files
COPY --from=builder /app/packages/happy-server ./packages/happy-server

# Set working directory
WORKDIR /app/packages/happy-server

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# Start the server
CMD ["node", "sources/main.js"]