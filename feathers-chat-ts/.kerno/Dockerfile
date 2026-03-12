# Multi-stage Dockerfile for feathers-chat-ts
# This Dockerfile should be placed in the root directory of the feathers-chat-ts application

# Build stage
FROM node:18-alpine AS builder

# Install build dependencies for native modules (sqlite3)
RUN apk add --no-cache python3 make g++

# Set working directory
WORKDIR /app

# Copy package files first for better layer caching
COPY package.json ./
COPY package-lock.json* ./

# Install all dependencies (including dev dependencies needed for build)
RUN npm install

# Copy TypeScript configuration
COPY tsconfig.json ./

# Copy source code
COPY src ./src

# Copy migrations if they exist
COPY migrations ./migrations 2>/dev/null || :

# Compile TypeScript to JavaScript
RUN npm run compile

# Run database migrations
RUN npm run migrate || true

# Production stage
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package.json ./
COPY package-lock.json* ./

# Install production dependencies only
# sqlite3 requires build tools, so install them temporarily
RUN apk add --no-cache python3 make g++ && \
    npm install --omit=dev && \
    npm cache clean --force && \
    apk del python3 make g++

# Copy compiled JavaScript from builder stage
COPY --from=builder /app/lib ./lib

# Copy public assets if they exist
COPY --from=builder /app/public ./public 2>/dev/null || :

# Copy configuration files if they exist
COPY --from=builder /app/config ./config 2>/dev/null || :

# Copy database file created during migration
COPY --from=builder /app/*.sqlite* ./ 2>/dev/null || :

# Expose application port
EXPOSE 3030

# Set production environment
ENV NODE_ENV=production

# Run the application
CMD ["npm", "start"]
